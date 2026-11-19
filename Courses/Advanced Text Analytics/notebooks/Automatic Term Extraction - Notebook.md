---
type: course-note
date: 2026-11-19
tags:
  - course
  - advanced-text-analytics
  - notebook
  - term-extraction
  - c-value
  - scikit-learn
  - pandas
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L15_notebook]]"
ai-first: true
---

## For Future Agent

Notebook companion for Lecture 15, `ATA_L15_Automatic_Term_Extraction_companion.ipynb`: a deliberately small, fully inspectable ATE pipeline on a 12-sentence synthetic GDPR-style corpus plus a 12-sentence general reference corpus, scored against 20 gold term types. Pipeline: surface n-gram candidates with a boundary stop-word filter -> five signals (frequency, mean TF-IDF, smoothed log-ratio specificity, bigram PMI, C-value) plus a head-noun linguistic proxy -> MinMax-normalized weighted hybrid -> type-level Precision/Recall/F1@k, per-signal ablation at k = 30, error analysis; optional spaCy POS patterns, sentence-embedding and LLM extensions are off by default. Extracted 2026-11-19 from `raw/ATA_L15_notebook.md`, which describes the code but reports **no numeric outputs** - any number below is a toy illustration. Critical caveats added here: PMI is computed but **not used** in the hybrid; the `LEGAL_HEADS` list overlaps almost entirely with the gold heads (leakage); junk n-grams dilute the C-value discount. Theory note: [[Automatic Term Extraction]].

## 1. Purpose and Learning Objectives

```
corpus -> candidate generation -> several scores -> hybrid ranking -> evaluation
```

The notebook is a teaching pipeline, explicitly **not** a state-of-the-art legal ATE system, and a starting point for DETECH 2027 legal term extraction. The core runs offline with `numpy`, `pandas` and `scikit-learn`; heavier extensions sit behind flags.

After it you should be able to: separate candidate generation from ranking; compute frequency, TF-IDF, specificity, PMI and C-value; explain why nested terms need special treatment; build a hybrid ranker; evaluate a ranked list of term **types**; place POS patterns, embeddings, supervised models and LLMs into the same pipeline.

## 2. Setup

```python
from collections import Counter, defaultdict
import math, re
import numpy as np
import pandas as pd
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.preprocessing import MinMaxScaler
```

| Import | Role |
|---|---|
| `Counter` | counts tokens, n-grams, candidates |
| `re` | tokenizer that keeps hyphenated words together |
| `TfidfVectorizer` | n-gram TF-IDF over the legal corpus |
| `MinMaxScaler` | rescales every signal to $[0, 1]$ before the weighted sum |

## 3. Data

### 3.1 Two Corpora

| Corpus | Size | Role |
|---|---|---|
| `LEGAL_DOCS` | 12 synthetic sentences | **domain corpus**: candidates are extracted and scored here |
| `REFERENCE_DOCS` | 12 general-language sentences | **reference corpus** for domain specificity |

The reference corpus is built with **confusable words** used in a non-legal sense: *court* (sports), *controller* (of a company), *transfer* (of funds or technology), *authority* (city authority), *safeguards* (laboratory equipment), *personal* (personal goals), *rules* (staff rules). They exist so the specificity signal has something to discriminate.

### 3.2 Gold Terminology

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

20 **term types**. Note:

- **Nesting is intentional**: *personal data* sits inside *sensitive personal data* and *transfer of personal data*; *data protection* is the stem of three gold terms but is itself **not** gold.
- Length mix: 1 unigram (*proportionality*), 12 bigrams, 6 trigrams (one with an internal preposition: *right to erasure*), 1 four-gram (*transfer of personal data*).
- In a real project the gold inventory must come from an **explicit annotation protocol with expert validation**, not a hand-written set.

## 4. Candidate Generation

> Candidate generation asks **what could be a term**; ranking asks **which candidates are likely to be terms**.

```python
TOKEN_RE = re.compile(r"[A-Za-z]+(?:-[A-Za-z]+)?")

BOUNDARY_STOP = {
    "the", "a", "an", "and", "or", "but", "if", "when", "whether",
    "in", "on", "at", "by", "with", "from", "for", "to", "of", "as",
    "is", "are", "was", "were", "be", "been", "being", "had", "has", "have",
    "may", "must", "can", "they", "it", "its", "their", "this", "that"
}
```

- **Tokenizer**: alphabetic words, optionally joined by **one** hyphen, lowercased. Digits are dropped (so *Article 6* loses its number - harmless for terms, fatal for legal references).
- **`generate_ngrams(max_n=4)`**: all 1- to 4-grams per document, with two filters:
  1. **Boundary stop-word filter** - discard a candidate whose **first or last** word is in `BOUNDARY_STOP`. Internal function words are allowed, so *right to erasure* and *transfer of personal data* survive while *of personal data* and *the data* do not.
  2. **Unigram length floor** - single words shorter than 4 characters are dropped (*law*, *act*): a blunt instrument.
- Counts over all legal documents go into `candidate_freq` (a `Counter`); its keys are the candidate inventory.

**Design reflection.** The generator deliberately **over-generates**: *important decision* passes, and so would fragments like *assessed whether the data* on the lecture's running sentence (first word *assessed*, last word *data*, internal stop words allowed). This is the recall-first stage; precision is the ranker's job.

## 5. The Ranking Signals

All signals become columns of one `pandas` DataFrame (`freq_df`) indexed by candidate.

### 5.1 Frequency

*How often does the candidate occur in the legal corpus?* Raw count. It cannot tell legal terminology from frequent generic words (*data*, *authority*).

### 5.2 TF-IDF

```python
vectorizer = TfidfVectorizer(
    lowercase=True,
    token_pattern=r"(?u)\b[A-Za-z]+(?:-[A-Za-z]+)?\b",
    ngram_range=(1, 4),
    stop_words=None,
)
```

- Same token definition as the generator, 1- to 4-grams.
- Score = **mean TF-IDF over the 12 documents** (`X.mean(axis=0)`), column `tfidf`.
- `stop_words=None` keeps internal function words available inside n-grams.
- Note: the vectorizer has its **own** n-gram vocabulary (no boundary filter), so it must be looked up per candidate; candidates are a subset of its features.

Caution: TF-IDF measures document salience, not termhood.

### 5.3 Domain Specificity

*Is the candidate frequent here only because it is frequent everywhere?*

$$
\mathrm{specificity}(t) = \log_2 \frac{\hat p(t \mid \text{domain})}{\hat p(t \mid \text{reference})},
\qquad
\hat p(t \mid C) = \frac{\mathrm{count}_C(t) + \epsilon}{\mathrm{total}_C + \epsilon\,(n_{\text{cand}} + 1)},\quad \epsilon = 0.5
$$

- `corpus_candidate_counts(docs)` runs the **same generator** on both corpora, so counts are comparable.
- The additive smoothing over the candidate vocabulary (the $+1$ reserves mass for unseen strings) prevents a zero denominator when the reference corpus never contains $t$.
- The raw source also writes the formula with an extra $+\epsilon$ inside the ratio; with $\epsilon = 0.5$ added to probabilities that would flatten every score towards 0, so the smoothing is most plausibly applied once, to the counts (medium).
- Sign: $>0$ legal-concentrated, $\approx 0$ generic, $<0$ more general-language.

**TODO 1 - reference-corpus effect.** Compare *court* (present in both corpora: legal court vs sports court) with *legal certainty* (legal only). Expected: *court* near 0, *legal certainty* clearly positive. Then reflect on corpus quality: with a **tiny** reference corpus almost every candidate has reference count 0, the denominator is the same smoothed constant for all of them, and specificity degenerates into a monotone transform of domain frequency; with a **mismatched** reference (e.g. one that already contains legal news) real terms are penalized. Specificity is only as good as its reference corpus.

### 5.4 PMI - Bigram Cohesion

$$
\mathrm{PMI}(x, y) = \log \frac{P(x, y)}{P(x)\,P(y)}
$$

- Unigram and bigram counts over the tokenized legal documents.
- `bigram_pmi(term)` returns `0.0` for non-bigrams and for bigrams seen **fewer than 2 times** - the frequency floor against PMI's rare-event bias.
- Only defined for two-word candidates; longer ones would need e.g. likelihood-ratio or generalized cohesion measures.
- **PMI does not enter the hybrid score** (see the weight table in Section 7): it is computed and shown for inspection only. Unithood is thus represented in the final ranking only indirectly (via C-value's length term and the linguistic proxy).

### 5.5 C-value - Nested Terms

The conceptually central signal. A short candidate is frequent partly **because** it sits inside longer ones: *data* is counted inside *personal data*, *data controller*, *sensitive personal data*...

$$
C\text{-value}(a) = w(a) \cdot \max\!\left(0,\; f(a) - \frac{1}{|T_a|}\sum_{b \in T_a} f(b)\right),
\qquad
w(a) = \begin{cases} 1.0 & |a| = 1 \\ \log_2 |a| & |a| \ge 2 \end{cases}
$$

(when $T_a = \emptyset$ the subtraction is skipped and the score is $w(a)\, f(a)$).

Implementation details:

- `contains_as_token_sequence(shorter, longer)` - strict containment: `shorter` must have **fewer tokens** and occur as a **contiguous** token sub-sequence of `longer` (so *data* is in *personal data*, but *personal data* is not in *personal and sensitive data*).
- Unigram weight $1.0$ instead of $\log_2 1 = 0$, otherwise every unigram (including gold *proportionality*) would score 0.
- Floor at 0 so C-value is never negative.

**Inspection cell** prints frequency vs C-value for *data*, *personal data*, *sensitive personal data*, *transfer of personal data*, *data protection obligations*: *data* has a high count but a much reduced C-value, while longer, independently used expressions rise.

**Caveat specific to this implementation.** $T_a$ ranges over **all** candidates, and the surface generator produces many junk n-grams that contain *data* once each (fragments of the *assessed whether the data* kind). Each adds an $f(b) = 1$ to the average, pulling it towards 1 and **weakening the discount**. The original C-value applies to linguistically filtered candidates, where $T_a$ is cleaner - one more reason to use real POS patterns (Section 10.1).

## 6. Linguistic Plausibility Proxy

No NLP model is downloaded in the main run, so POS tagging is replaced by a rule:

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
        score += 1.0        # plausible legal head noun
    if 2 <= len(words) <= 4:
        score += 0.4        # typical multiword length
    return score
```

- English NPs are **right-headed**, so the last word is the head; legal terms tend to end in a domain head noun.
- 2-4 words is the typical multiword-term band.
- Possible values: 0, 0.4, 1.0, 1.4.

**Leakage warning.** 19 of the 20 gold terms end in a word of `LEGAL_HEADS` (only *data subject* does not: *subject* is missing). The list was evidently written with the gold set in view, so the 0.25-weighted linguistic signal partly **encodes the answer key**, and the hybrid's evaluation is optimistic. In a real experiment the head list must be derived independently (from a grammar, a termbase, or a held-out development set).

## 7. Hybrid Ranking

Each signal is MinMax-scaled to $[0, 1]$, then combined with **explicit** weights:

| Signal | Weight |
|---|---|
| frequency | 0.15 |
| TF-IDF | 0.10 |
| domain specificity | 0.25 |
| C-value | 0.25 |
| linguistic plausibility | 0.25 |

```python
rank_df["hybrid"] = sum(rank_df[c] * w for c, w in weights.items())
rank_df["gold"] = rank_df["candidate"].isin(GOLD_TERMS)
```

- Weights sum to 1.0, so `hybrid` is in $[0, 1]$.
- **Weights are a design choice, not a discovery**: the goal is to make them visible and testable, not to claim optimality.
- The three heavy weights push towards **domain-specific, long, structurally plausible** candidates rather than merely frequent strings.
- Sorted by `hybrid` descending, ties broken alphabetically (reproducibility).
- MinMax is sensitive to outliers: one huge raw value (e.g. the frequency of *data*) compresses all other candidates near 0 on that column.

## 8. Type-Level Evaluation

The unit is the **distinct term type**, because the use case is a **ranked candidate list for human validation**, not mention tagging.

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

$$
P@k = \frac{|\text{top-}k \cap G|}{k}, \qquad R@k = \frac{|\text{top-}k \cap G|}{|G|}, \qquad F_1@k = \frac{2\,P@k\,R@k}{P@k + R@k}
$$

Reported for $k \in \{10, 20, 30, 50, 75\}$. Built-in ceilings with $|G| = 20$: at $k = 10$, $R@10 \le 0.5$; at $k = 50$, $P@50 \le 0.4$. Perfect P and R together are only possible at $k = 20$. Example: 12 gold terms in the top 30 -> $P = 0.40$, $R = 0.60$, $F_1 = 0.48$.

### 8.1 Ablation: Single Signals vs Hybrid

```python
systems = {
    "frequency":   ranking_from("frequency"),
    "tfidf":       ranking_from("tfidf"),
    "specificity": ranking_from("specificity"),
    "c_value":     ranking_from("c_value"),
    "hybrid":      ranked_terms,
}
```

All five compared at $k = 30$. Habit to keep: **never report only the final system** - show what each signal contributes. (The linguistic proxy has no single-signal row; with only 4 distinct values it would mostly be tie-breaking.)

## 9. Error Analysis

Two lists: **false positives in the top 30** with their signal values (which signal pulled them up?) and **gold terms missed in the top 30**.

**TODO 2** - pick **one** failure mode, change **one** component, state the hypothesis:

| Failure mode | Component to change |
|---|---|
| too many generic phrases | domain specificity (better reference corpus) |
| good multiword terms ranked too low | C-value or the length signal |
| terms with internal function words missed | candidate-generation boundary rules |
| variants split across types (*right of erasure* / *right to erasure*, plurals) | lemmatization / normalization |
| semantically right but lexically unusual terms | embeddings |

The question *"what hypothesis does your change test?"* turns debugging into a controlled experiment.

## 10. Optional Extensions (off by default)

### 10.1 POS Patterns with spaCy - `RUN_SPACY = False`

Needs `en_core_web_sm`. A `Matcher` replaces surface n-grams with linguistic patterns: `ADJ NOUN`, `NOUN NOUN`, `ADJ* NOUN+`, `NP of NP` (*transfer of personal data*), `NP to NP` (*right to erasure*). This is real syntactic knowledge where the boundary-stop heuristic only works by luck.

### 10.2 Embedding Signal - `RUN_SENTENCE_TRANSFORMERS = False`

Model `all-MiniLM-L6-v2`.

```
candidate -> embedding -> cosine to a domain vector -> semantic score
```

- Seed terms *data controller*, *supervisory authority*, *right to erasure*, *legal certainty*, *data protection* are embedded and **averaged into one domain vector**.
- The top-100 hybrid candidates are embedded; score = dot product of normalized vectors = cosine.
- Semantic similarity complements lexical statistics but **does not prove termhood**. Also note: 4 of the 5 seeds are gold terms, the same leakage risk as Section 6.

### 10.3 LLM-Assisted ATE

Three insertion points: **generation** (terms + exact spans), **validation** (does this expression denote a legal concept *in context*?), **enrichment** (variants, definitions, concept links).

Reproducibility requires reporting: model and version; prompt and output schema; temperature/decoding; whether external context or retrieval is used; exact span-matching and normalization policy; run-to-run variability.

```text
Extract domain-specific legal terms from the passage.
Return only spans that occur verbatim in the text.
For each span return: {term, start, end, confidence, rationale}.
Do not invent terms absent from the passage.
```

Verbatim spans, a fixed schema and the no-invention rule target the two classic LLM extraction failures: **hallucinated terms** and **span drift** (offsets that do not match the text - check `text[start:end] == term`).

## 11. DETECH 2027 Experiment Plan

> Design **comparisons**, not merely systems.

The notebook ends with the Systems A-F matrix (n-grams + frequency; POS + C-value; POS + specificity; supervised span/token model; hybrid + legal vocabulary; LLM prompt/validator). Its full reading is in the theory note, Section 8. The notebook itself implements a mix that matches none of A-F exactly: n-gram candidates (as A) with a combined ranker but no external vocabulary (closest to E without knowledge).

Reflection questions: what counts as a term under our guidelines? Are nested/overlapping terms allowed? Mentions, types or a ranked list? What does each component contribute? Which errors matter most downstream?

## 12. End-to-End Flow

```
LEGAL_DOCS (12)                         REFERENCE_DOCS (12)
   |                                         |
tokenize -> 1..4-grams, boundary filter   same generator -> ref counts
   |                                         |
candidate_freq (Counter)                     |
   |-- frequency                             |
   |-- TF-IDF (mean over docs)               |
   |-- specificity (smoothed log2 ratio) <---+
   |-- PMI (bigrams, f >= 2)  [inspection only]
   |-- C-value (nesting discount)
   '-- linguistic proxy (head noun + length)
              |
   MinMax -> weighted sum -> ranked list -> P/R/F1@k, ablation @30, error analysis
```

## 13. Takeaways

1. Two stages: generate for recall, rank for precision.
2. No single signal suffices; each captures a different facet of "termness".
3. Nesting inflates the frequency of short sub-terms - C-value removes the inherited part.
4. Specificity needs a reference corpus, and that corpus bounds its quality.
5. Hybrid weights are design choices: explicit, visible, testable.
6. Evaluate type-level P/R/F1@k, run per-signal ablations, analyse errors, change one component at a time.
7. Check every hand-built resource (head lists, seed terms) for overlap with the gold set before trusting a score.

## Related

- [[Automatic Term Extraction]] - Lecture 15 theory note: termhood/unithood, C-value/NC-value, Systems A-F
- [[Legal Information Extraction and Knowledge Graphs - Notebook]] - Lecture 14: same GDPR domain, regex + `EntityRuler` legal NER
- [[TF-IDF and Static Embeddings - Notebook]] - `TfidfVectorizer` and cosine similarity basics
- [[Advanced Named Entity Recognition - Notebook]] - span candidates and threshold sweeps
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L15_notebook]] (AY 2026/2027)
