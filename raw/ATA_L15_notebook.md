# Lecture 15 — Automatic Term Extraction: Detailed Notebook Explanation

**Course:** Advanced Text Analytics (ATA)
**Topic:** Automatic Term Extraction (ATE) — statistical, linguistic, and hybrid methods
**Notebook:** `ATA_L15_Automatic_Term_Extraction_companion.ipynb`
**Domain of the examples:** law / data-protection regulation (GDPR-style language)

---

## 1. Overview and Purpose

This notebook is the executable companion to Lecture 15. It implements a small, fully transparent **Automatic Term Extraction (ATE)** pipeline that mirrors the lecture material:

```
corpus → candidate generation → several scores → hybrid ranking → evaluation
```

Key characteristics of the notebook:

- **Deliberately small and inspectable.** Every score is computed with a few lines of readable Python so that each method can be studied in isolation. It is explicitly *not* a state-of-the-art legal ATE system.
- **Legal-domain examples.** The toy corpus uses realistic legal expressions (data protection, supervisory authorities, legitimate interest, etc.), so the notebook doubles as a methodological starting point for **DETECH 2027 legal term extraction** research.
- **Offline-friendly.** The core pipeline runs with only `numpy`, `pandas`, and `scikit-learn`. Heavier extensions (spaCy POS patterns, sentence embeddings, LLMs) are present but disabled by default behind flags.

### Learning objectives

By the end of the notebook a student should be able to:

1. Distinguish **candidate generation** from **candidate ranking**.
2. Compute simple **frequency**, **TF–IDF**, **domain-specificity**, **PMI**, and **C-value** signals.
3. See why **nested terms** require special treatment.
4. Combine complementary signals in a **hybrid ranker**.
5. Evaluate a ranked list of term **types**.
6. Identify where **POS patterns**, **embeddings**, **supervised models**, and **LLMs** could enter the same pipeline.

---

## 2. Setup and Imports

```python
from collections import Counter, defaultdict
import math
import re

import numpy as np
import pandas as pd
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.preprocessing import MinMaxScaler
```

- `Counter` — frequency counting of tokens, n-grams, and candidates.
- `re` — a custom tokenizer that keeps hyphenated words (e.g. `data-protection`-style compounds) together.
- `TfidfVectorizer` — computes n-gram TF–IDF over the corpus.
- `MinMaxScaler` — normalizes all ranking signals to `[0, 1]` before they are combined in the hybrid score.

---

## 3. The Data: Legal Corpus, Reference Corpus, and Gold Terms

### 3.1 Two corpora

The notebook defines two tiny in-memory corpora:

| Corpus | Size | Role |
|---|---|---|
| `LEGAL_DOCS` | 12 synthetic sentences | The **domain corpus** from which candidates are extracted and scored (data-protection / legal language). |
| `REFERENCE_DOCS` | 12 general-language sentences | The **reference corpus** used to measure *domain specificity* — terms frequent everywhere should not be mistaken for legal terms. |

The reference corpus is carefully constructed to contain **confusable words** that also appear in legal texts but in a general sense: *court* (in a sports context), *controller* (of a company), *transfer* (of funds or technology), *authority* (city authority), *safeguards* (for laboratory equipment), *personal* (personal goals), *rules* (staff rules). This lets the specificity signal demonstrate its purpose.

### 3.2 The gold terminology

```python
GOLD_TERMS = {
    "supervisory authority", "data controller", "legitimate interest",
    "appropriate safeguards", "transfer of personal data", "personal data",
    "right to erasure", "administrative fine", "data protection obligations",
    "international data transfer", "legal certainty", "data protection authority",
    "data subject", "data protection rights", "data processor", "third country",
    "lawful basis", "fundamental rights", "sensitive personal data",
    "proportionality",
}
```

This set of **20 term types** is the reference inventory used for evaluation (Precision@k / Recall@k). The notebook stresses that in a real project the gold inventory must come from an **explicit annotation protocol and expert validation**, not from a hand-written list.

Note that the gold set contains **nested terms**: `personal data` sits inside `sensitive personal data` and inside `transfer of personal data`. This is intentional — nesting is one of the central problems of ATE addressed later in the notebook.

---

## 4. Candidate Generation (Section 2)

### 4.1 The central distinction

> **Candidate generation asks *what could be a term*; ranking asks *which candidates are likely to be terms*.**

Candidate generation is the *recall-oriented* first stage: it proposes a superset of possible terms. Ranking/filtering later removes the noise.

### 4.2 The surface baseline: token n-grams

```python
TOKEN_RE = re.compile(r"[A-Za-z]+(?:-[A-Za-z]+)?")

BOUNDARY_STOP = {
    "the", "a", "an", "and", "or", "but", "if", "when", "whether",
    "in", "on", "at", "by", "with", "from", "for", "to", "of", "as",
    "is", "are", "was", "were", "be", "been", "being", "had", "has", "have",
    "may", "must", "can", "they", "it", "its", "their", "this", "that"
}
```

**Tokenization.** `TOKEN_RE` extracts alphabetic words, optionally joined by a single hyphen, and lowercases them.

**N-gram generation (`generate_ngrams`, `max_n=4`).** For each document, all 1-grams through 4-grams are produced, with two filtering rules:

1. **Boundary stop-word filter** — a candidate is discarded if its **first or last** word is in `BOUNDARY_STOP`. Crucially, *internal* function words are allowed, so phrases like **right to erasure** and **transfer of personal data** survive. This is a common ATE heuristic: terms rarely begin or end with a function word, but prepositions frequently occur inside them.
2. **Minimum length for unigrams** — single-word candidates shorter than 4 characters are dropped (removes noise like *law*, *act* — though this is a blunt instrument).

**Frequency counting.** All candidates from all legal documents are aggregated into `candidate_freq` (a `Counter`), and the sorted list of distinct candidate **types** forms the working candidate inventory.

### 4.3 Design reflection

The baseline intentionally **over-generates**. Expressions such as *important decision* satisfy the surface pattern without being domain terms. That is acceptable and even desirable: candidate generation should favour **recall**, leaving precision to the ranking stage.

---

## 5. The Five Ranking Signals

Each signal answers a different question about a candidate. All of them are added as columns to a single `pandas` DataFrame (`freq_df`) keyed by candidate.

### 5.1 Signal 1 — Frequency (Section 3)

> *How often does this candidate occur in the legal corpus?*

The simplest possible baseline: sort candidates by raw count in `LEGAL_DOCS`.

**Limitation (stated explicitly):** frequency does **not** answer whether the expression is *specifically legal terminology*. Generic words like *data* or *authority* can be frequent without being domain terms.

### 5.2 Signal 2 — TF–IDF (Section 4)

> *How salient is this candidate at the document level?*

```python
vectorizer = TfidfVectorizer(
    lowercase=True,
    token_pattern=r"(?u)\b[A-Za-z]+(?:-[A-Za-z]+)?\b",
    ngram_range=(1, 4),
    stop_words=None,
)
```

- TF–IDF is computed over 1- to 4-grams of the legal documents.
- The **mean TF–IDF across all documents** (`X.mean(axis=0)`) is used as the candidate's score, stored in the `tfidf` column.
- `stop_words=None` keeps function words available inside n-grams (consistent with the candidate generator's permissive internal-word policy).

**Key caution:** high TF–IDF is **not identical to high termhood**. TF–IDF measures document salience/typicality, not domain-term status. It is one useful signal among several.

### 5.3 Signal 3 — Domain Specificity (Section 5)

> *Is this candidate frequent in the legal corpus simply because it is frequent everywhere?*

To answer this, the notebook compares relative frequency in the domain corpus against a general **reference corpus**, using a smoothed log-ratio:

$$
\mathrm{specificity}(t)=\log_2\frac{p(t\mid domain)+\epsilon}{p(t\mid reference)+\epsilon}
$$

Implementation details:

- `corpus_candidate_counts(docs)` runs the *same* candidate generator over both corpora, so counts are directly comparable.
- Smoothing constant `eps = 0.5` avoids division by zero for candidates absent from one corpus.
- Probabilities are estimated as `(count + eps) / (total + eps * (n_candidates + 1))` — a simple additive (Laplace-style) smoothing over the candidate vocabulary.

Interpretation:

- `specificity > 0` → relatively concentrated in the legal corpus (good term signal).
- `specificity ≈ 0` → equally common in both corpora (likely generic).
- `specificity < 0` → more characteristic of the reference corpus.

**TODO 1** in the notebook asks the reader to inspect the reference-corpus effect: compare *court* (frequent in both corpora — a sports event, a legal court) against *legal certainty* (concentrated in legal text), and to reflect on what happens if the reference corpus is too small or badly matched. This highlights that **specificity is only as good as its reference corpus** — a classic methodological dependency in domain-oriented ATE and terminology work.

### 5.4 Signal 4 — PMI for Bigram Cohesion (Section 6)

> *Do these two words occur together more often than their individual frequencies would predict?*

PMI measures **unithood** — the internal cohesion of a multiword expression:

$$
PMI(x,y) = \log \frac{P(x,y)}{P(x)\,P(y)}
$$

Implementation:

- Unigram and bigram counts are computed directly over tokenized legal documents.
- `bigram_pmi(term)` returns `0.0` for anything that is not a bigram, and **`0.0` for bigrams occurring fewer than 2 times**.

**Caution stated in the notebook:** PMI can overvalue rare bigrams (a pair seen once can have a huge PMI), hence the minimum-frequency guard of 2 occurrences.

Note that PMI here is only defined for **two-word** candidates; longer candidates receive 0. In a richer system, cohesion measures for longer expressions (e.g. nested PMI or likelihood ratios) could be added.

### 5.5 Signal 5 — C-value and Nested Terms (Section 7)

This is the most conceptually important signal of the lecture. Consider the nesting chain:

- **data**
- **personal data**
- **sensitive personal data**
- **transfer of personal data**

A short candidate may be frequent **partly because it occurs inside longer candidates**. The word *data* is frequent largely as a substring of *personal data*, *sensitive personal data*, *data controller*, etc. Raw frequency therefore inflates short, embedded terms.

**C-value** rewards length and frequency while **discounting frequency inherited from longer candidate terms that contain the candidate**:

$$
C\text{-}value(a) = \log_2 |a| \left( f(a) - \frac{1}{|T_a|} \sum_{b \in T_a} f(b) \right)
$$

where:

- $|a|$ = number of words in candidate term $a$
- $f(a)$ = frequency of $a$
- $T_a$ = set of **longer** candidate terms that contain $a$
- $f(b)$ = frequency of a longer term $b$

Implementation notes (deliberately literal and transparent):

- `contains_as_token_sequence(shorter, longer)` checks strict token-sequence containment (the shorter must be *strictly* shorter and appear as a contiguous sub-sequence).
- For unigrams the length weight is set to `1.0` (since $\log_2 1 = 0$ would zero out every unigram).
- The adjusted frequency is `f(a) - mean(f(b) for b in T_a)`, floored at 0 (`max(adjusted, 0.0)`), so C-value is never negative.

**Inspection cell.** The notebook prints frequency vs. C-value for `data`, `personal data`, `sensitive personal data`, `transfer of personal data`, and `data protection obligations` to make the nesting discount visible: `data` may have a high raw count but a much lower C-value, while the longer, independently-used expressions score higher.

C-value is the classic **termhood** measure for nested multiword terms from Frantzi, Ananiadou & Mima (2000), cited in the references.

---

## 6. Linguistic Plausibility Signal (Section 8)

The main pipeline must run **without downloading an NLP model**, so instead of real POS tagging the notebook uses a deliberately simple rule-based proxy:

```python
LEGAL_HEADS = {
    "authority", "controller", "processor", "interest", "safeguards", "data",
    "erasure", "fine", "obligations", "transfer", "certainty", "rights",
    "country", "basis", "proportionality", "regulation", "infringement",
    "protection", "processing", "responsibilities"
}

def linguistic_score(term):
    words = term.split()
    score = 0.0
    if words[-1] in LEGAL_HEADS:
        score += 1.0        # plausible legal/conceptual head noun
    if 2 <= len(words) <= 4:
        score += 0.4        # typical multiword-term length
    return score
```

Rationale:

- In English noun phrases the **final word is the head**. Legal terms typically end in a domain-relevant head noun (*authority*, *interest*, *obligations*, *proportionality*...).
- A length band of 2–4 words matches the typical span of multiword terms.

The notebook is explicit that this **does not replace POS tagging**; it illustrates how explicit domain/linguistic knowledge can enter a hybrid system as a lightweight feature. A genuine POS-pattern extractor appears later as an optional extension.

---

## 7. Hybrid Ranking (Section 9)

There is no universal formula for combining signals. The notebook normalizes each signal to `[0, 1]` with `MinMaxScaler` and combines them with **explicit, visible weights**:

| Signal | Weight |
|---|---|
| frequency | 0.15 |
| TF–IDF | 0.10 |
| domain specificity | 0.25 |
| C-value | 0.25 |
| linguistic plausibility | 0.25 |

```python
rank_df["hybrid"] = sum(rank_df[c] * w for c, w in weights.items())
rank_df["gold"] = rank_df["candidate"].isin(GOLD_TERMS)
```

Design points:

- **Weights are a design choice, not a discovery.** The notebook states the purpose is *not* to claim these weights are optimal, but to make the choice **visible and testable** — a core methodological habit.
- The three heaviest weights (specificity, C-value, linguistic) push the ranker toward *domain-specific, long, structurally plausible* terms rather than mere frequent strings.
- A `gold` boolean column is attached for later error analysis and evaluation.
- Final ordering: sort by `hybrid` descending, with alphabetical tie-breaking for reproducibility.

---

## 8. Type-Level Evaluation (Section 10)

The notebook evaluates **distinct term types**, not every mention — the natural unit when the downstream use case is a **ranked terminology candidate list for human validation**.

### 8.1 Metrics at top-k

```python
def evaluate_at_k(ranked_terms, gold, k):
    pred = ranked_terms[:k]
    pred_set = set(pred)
    tp = len(pred_set & gold)
    precision = tp / max(len(pred_set), 1)
    recall = tp / max(len(gold), 1)
    f1 = 0.0 if precision + recall == 0 else 2 * precision * recall / (precision + recall)
    ...
```

- **Precision@k** = proportion of the first *k* candidates that are gold terms.
- **Recall@k** = proportion of all gold terms recovered within the first *k*.
- **F1@k** = harmonic mean of the two.

The evaluation table is produced for `k ∈ {10, 20, 30, 50, 75}`.

### 8.2 Single-signal vs. hybrid comparison

A second evaluation block builds one ranking per signal — `frequency`, `tfidf`, `specificity`, `c_value`, and the `hybrid` — and compares them all at **k = 30**:

```python
systems = {
    "frequency": ranking_from("frequency"),
    "tfidf":     ranking_from("tfidf"),
    "specificity": ranking_from("specificity"),
    "c_value":   ranking_from("c_value"),
    "hybrid":    ranked_terms,
}
```

> **Key experimental habit (stated in the notebook): do not report only the final system — compare what each signal contributes.** This ablation-style comparison is exactly what the DETECH project should do.

---

## 9. Error Analysis (Section 11)

A ranked list becomes actionable when we inspect *why* it makes errors:

1. **False positives in the top 30** — non-gold candidates that ranked highly, shown with their signal values so one can see *which* signal pulled them up.
2. **Missed gold terms** — gold terms not yet recovered within the top 30.

### TODO 2 — improve one failure mode

The reader is asked to choose **one** error category and change **one** component of the pipeline, mapping error → hypothesis → fix:

| Failure mode | Pipeline component to change |
|---|---|
| Too many generic phrases | Strengthen domain specificity (e.g. better reference corpus) |
| Good multiword terms ranked too low | Modify C-value or the length signal |
| Missed terms with internal function words | Change candidate-generation boundary rules |
| Variants split across several types | Add lemmatization/normalization |
| Semantically right but lexically unusual terms | Add embeddings |

The accompanying reflection — *"what hypothesis does your change test?"* — frames debugging as experimental science rather than tinkering.

---

## 10. Optional Extensions (Sections 12–14)

All three extensions are **disabled by default** so the notebook runs offline; each activates behind a boolean flag.

### 10.1 Genuine POS-pattern candidate generation with spaCy (Section 12)

Flag: `RUN_SPACY = False` (requires `en_core_web_sm`).

Instead of surface n-grams, a spaCy `Matcher` extracts candidates matching linguistic patterns:

- `ADJ + NOUN`
- `NOUN + NOUN`
- `ADJ* + NOUN+` (optional adjectives followed by one or more nouns)
- noun phrase + **of** + noun phrase (e.g. *transfer of personal data*)
- noun phrase + **to** + noun phrase (e.g. *right to erasure*)

This replaces the blind `BOUNDARY_STOP` heuristic with real syntactic knowledge and is the natural home for the phrases the surface baseline handles only by luck.

### 10.2 Embedding-based semantic signal (Section 13)

Flag: `RUN_SENTENCE_TRANSFORMERS = False` (model: `all-MiniLM-L6-v2`).

Conceptual flow:

```
candidate → embedding → similarity to legal-domain signal → semantic score
```

- A set of **seed terms** (`data controller`, `supervisory authority`, `right to erasure`, `legal certainty`, `data protection`) is embedded and averaged into a single **domain vector**.
- Each of the top-100 ranked candidates is embedded and scored by cosine similarity (dot product of normalized vectors) to that domain vector.

**Caveat:** semantic similarity is complementary to lexical statistics, and by itself does **not** prove that an expression is a term.

### 10.3 LLM-assisted ATE (Section 14)

An LLM can be inserted at three different points in the pipeline:

1. **Candidate generation** — ask for legal terms and their exact spans.
2. **Candidate validation** — ask whether a proposed expression designates a legal-domain concept *in context*.
3. **Enrichment** — ask for variants, definitions, or concept links.

For a reproducible experiment the notebook demands specification of:

- model/version;
- prompt and output schema;
- temperature/decoding settings where applicable;
- whether external context or retrieval is used;
- exact span-matching and normalization policy;
- repeated-run variability if relevant.

Example structured prompt provided in the notebook:

```text
Extract domain-specific legal terms from the passage.
Return only spans that occur verbatim in the text.
For each span return: {term, start, end, confidence, rationale}.
Do not invent terms absent from the passage.
```

The emphasis on *verbatim spans*, a *fixed schema*, and *no invented terms* addresses the two classic LLM failure modes in extraction tasks: hallucinated terms and span drift.

---

## 11. DETECH 2027 Experiment Plan (Section 15)

> **The important methodological principle is to design *comparisons*, not merely systems.**

The notebook closes with an experiment matrix — six systems (A–F), each isolating a different research question:

| System | Candidate generator | Ranking/model | External knowledge | Question addressed |
|---|---|---|---|---|
| A | n-grams | frequency | none | Transparent baseline |
| B | POS patterns | C-value | none | Linguistic + nested-term evidence |
| C | POS patterns | domain specificity | reference corpus | Termhood via corpus contrast |
| D | span/token model | supervised | annotations | Learned legal ATE |
| E | hybrid | combined ranker | legal vocabulary | Knowledge-guided ATE |
| F | LLM | prompt / validator | optional retrieval | Generative ATE |

Reading the matrix: systems A–C differ in *one* factor at a time (ranking signal, then external knowledge), D tests supervised learning, E combines everything with expert knowledge, and F replaces the whole statistical stack with generative extraction. This is a classic controlled-comparison design.

### Final reflection questions (for a legal ATE project)

- What exactly counts as a **term** under our annotation guidelines?
- Are **nested and overlapping** terms allowed?
- Are we evaluating **mentions**, **types**, or a **ranked candidate list**?
- What does each component contribute: syntax, frequency, specificity, nesting, semantics, external knowledge?
- Which **errors matter most** for the downstream terminology workflow?

> **The goal is not to choose the most sophisticated model. It is to combine the available data, resources, expert knowledge and computational models in a way that matches the task.**

---

## 12. Conceptual Summary: The Vocabulary of ATE

The notebook quietly introduces several standard terminology-engineering concepts. Consolidated:

| Concept | Meaning | Where it appears |
|---|---|---|
| **Termhood** | How likely a string is to be a domain term (external evidence: frequency, specificity, corpus contrast) | Frequency, TF–IDF, specificity signals |
| **Unithood** | How strongly the words of a candidate stick together (internal evidence) | PMI signal |
| **Candidate generation vs. ranking** | Recall-oriented proposal stage vs. precision-oriented ordering stage | Section 2 vs. Sections 3–9 |
| **Nested terms** | Short terms embedded inside longer terms; short terms inherit inflated frequency | C-value signal |
| **Domain specificity** | Contrast between the domain corpus and a general reference corpus | Log-ratio signal |
| **Hybrid ranking** | Weighted combination of normalized complementary signals | Section 9 |
| **Type-level evaluation** | Scoring distinct term types (not mention occurrences) with Precision/Recall/F1@k | Section 10 |
| **Ablation / signal comparison** | Reporting each single-signal system alongside the hybrid | Section 10 comparison cell |

---

## 13. Pipeline Flowchart (End-to-End)

```
LEGAL_DOCS (12 legal sentences)          REFERENCE_DOCS (12 general sentences)
        │                                          │
        ▼                                          ▼
 tokenize → n-gram candidates (1..4)      same generator → ref counts
 (boundary stop-word filter)                        │
        │                                          │
        ▼                                          │
 candidate_freq (Counter)                           │
        │                                          │
        ├─► Signal 1: frequency ◄───────────────────┤
        ├─► Signal 2: TF–IDF (mean over docs)       │
        ├─► Signal 3: specificity (log-ratio) ◄─────┘
        ├─► Signal 4: PMI (bigram cohesion, f ≥ 2)
        ├─► Signal 5: C-value (nesting discount)
        └─► Linguistic plausibility (head noun + length)
                    │
                    ▼
        MinMax normalize → weighted sum (hybrid score)
                    │
                    ▼
        ranked candidate list  ──►  gold-flagged evaluation
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   Precision/Recall  signal     error analysis
      /F1 @ k        ablation   (FP + missed gold)
```

---

## 14. References Cited in the Notebook

- **ISO 5078:2025.** *Management of terminology resources — Terminology extraction.*
- **Ahmad, K., Davies, A., Fulford, H., & Rogers, M. (1994).** *What is a term? The semi-automatic extraction of terms from text.*
- **Kageura, K., & Umino, B. (1996).** *Methods of automatic term recognition: A review.* Terminology, 3(2), 259–289.
- **Frantzi, K., Ananiadou, S., & Mima, H. (2000).** *Automatic recognition of multi-word terms: the C-value/NC-value method.* International Journal on Digital Libraries, 3(2), 115–130. DOI: 10.1007/s007999900023.
- **DETECH 2026 Proceedings**, CEUR Workshop Proceedings, Vol. 4215.

The Kageura & Umino review is the standard survey of ATE methods; the Frantzi et al. paper is the original source of the C-value/NC-value measure implemented in Section 7; ISO 5078:2025 provides the contemporary standards framing for terminology extraction.

---

## 15. Takeaways

1. **ATE is a two-stage problem:** generate candidates for recall, then rank them for precision.
2. **No single signal suffices.** Frequency, TF–IDF, specificity, PMI, C-value, and linguistic plausibility each capture a different, complementary aspect of "termness."
3. **Nesting distorts frequency** — C-value's discount of inherited frequency is essential for multiword-term extraction.
4. **Domain specificity needs a reference corpus**, and the quality of that corpus bounds the quality of the signal.
5. **Hybrid weights are design choices** — make them explicit, visible, and testable.
6. **Evaluate like a scientist:** type-level P/R/F1@k, per-signal ablations, and qualitative error analysis — then change exactly one component at a time (TODO 2).
7. **The extension points are already in the pipeline:** POS patterns for better candidates, embeddings for semantics, supervised span models for learned extraction, LLMs for generative extraction — all comparable within the same experiment matrix (systems A–F).
8. **Match the method to the task:** the goal is not the most sophisticated model, but the right combination of data, resources, expert knowledge, and computation.
