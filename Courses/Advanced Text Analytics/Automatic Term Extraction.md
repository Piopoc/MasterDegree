---
type: course-note
date: 2026-11-19
tags:
  - course
  - advanced-text-analytics
  - term-extraction
  - terminology
  - legal-nlp
  - c-value
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L15]]"
  - "[[ATA_L15_notebook]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 15) on **Automatic Term Extraction (ATE)**: what a term is (vs named entity vs keyword), termhood vs unithood, the two-stage pipeline (recall-oriented candidate generation, precision-oriented ranking), the statistical signals (frequency, TF-IDF, domain specificity, PMI, C-value, NC-value), hybrid and resource-aware ranking, supervised/neural/LLM paradigms, and the controlled experiment matrix (Systems A-F) proposed for the DETECH 2027 legal term extraction project. Created 2026-11-19 from `raw/ATA_L15.md`, with formulas and implementation details cross-checked against the L15 notebook and worked numeric examples added (toy numbers, not notebook outputs). The runnable walkthrough lives in [[Automatic Term Extraction - Notebook]]. Prerequisite and contrast: legal NER from [[Legal Information Extraction and Knowledge Graphs]] (Lecture 14).

## 1. What ATE Is

**Automatic Term Extraction** is the computational discovery, extraction and **ranking** of lexical units that designate **domain-specific concepts** in unstructured text. It is not a dictionary lookup and not NER: the output is a ranked list of candidate *term types* that a terminologist then validates.

### 1.1 Term vs Named Entity vs Keyword

| Feature | Term | Named Entity | Keyword |
|---|---|---|---|
| Definition | lexical unit designating a **specialized concept** in a domain | a concrete real-world **instance** (person, organization, place) | word/phrase **salient for a document** or collection |
| Primary signal | specialized conceptual meaning | referential identification | document-level frequency and salience |
| Legal example | *supervisory authority*, *legitimate interest*, *right to erasure* | *Court of Justice*, *European Commission* | *Schrems* |

The categories overlap (a named entity can act as a domain concept), but ATE targets **conceptual content**, not reference (what NER targets) and not mere salience (what keyword extraction targets).

Quick test: *"European Commission"* names **one** specific institution -> entity. *"supervisory authority"* names a **class of institutions** defined by law -> term. *"Schrems"* is salient in a set of documents but designates no concept -> keyword.

### 1.2 Termhood vs Unithood

ATE scores candidates on two independent axes:

| Axis | Question | Example | Evidence type | Typical signals |
|---|---|---|---|---|
| **Termhood** | does the unit designate a domain-specific concept? | *legal certainty* is much more typical of legal text than of news | mostly **external** (how the unit is distributed across corpora) | frequency, TF-IDF, domain specificity, C-value |
| **Unithood** | do the words form a stable, non-accidental unit? | *data protection* is a conventional multiword unit, not a random co-occurrence | **internal** (how strongly the parts stick together) | PMI, other association measures |

A **single-word term** (*proportionality*) can have high termhood with no unithood question at all; a multiword expression can be highly cohesive (*Monday meeting* recurs in an office) but have no termhood. Good systems combine both.

### 1.3 From Legal NER to Legal ATE

| Dimension | Legal NER | Legal ATE | DETECH 2027 project question |
|---|---|---|---|
| Core query | *what entities/references occur?* | *what specialized concepts are expressed?* | *which signals and resources should we combine?* |
| Focus | people, courts, acts, provisions | concept-bearing lexical units | method design, signal weighting, evaluation |
| Nature | extraction of **known classes** | **discovery and ranking** | comparative benchmarking of hybrid pipelines |

Running example used throughout the lecture:

> *"The supervisory authority assessed whether the data controller had a legitimate interest and provided appropriate safeguards for the transfer of personal data."*

Terms: **supervisory authority**, **data controller**, **legitimate interest**, **appropriate safeguards**, **personal data** (and, nested around the last one, **transfer of personal data**).

## 2. The Two-Stage Pipeline

```
Legal corpus -> candidate generation -> candidate scores -> ranking / filtering -> expert or gold validation
               (linguistic, recall)    (statistical,      (precision)
                                         contextual, KB)
```

| Stage | Goal | Optimizes | Failure if done badly |
|---|---|---|---|
| **Candidate generation** | propose *what could be a term* | **recall** | a term never generated can never be ranked - the loss is permanent |
| **Candidate selection / ranking** | decide *which candidates are likely terms* | **precision** (good terms at the top) | true terms buried under generic phrases |

This is the same architectural split seen in concept normalization ([[Biomedical Named Entity Recognition and Concept Normalization]]): if the right answer is not in the candidate set, no ranker can recover it.

The split mirrors **ISO 5078:2025** (*Management of terminology resources - Terminology extraction*), which separates the extraction of candidate terms from the guidelines for selecting and validating relevant terminology.

## 3. Candidate Generation: Linguistic Frameworks

Linguistic constraints shrink the search space and encode grammatical plausibility **without needing annotated term labels**.

| Method | What it captures | Example |
|---|---|---|
| **POS patterns** | predefined tag sequences for noun phrases | `ADJ NOUN`, `NOUN NOUN`, `ADJ ADJ NOUN` |
| **Noun-phrase chunks** | head noun + pre/post-modifiers, incl. prepositional extensions | *transfer of personal data* |
| **Dependency parsing** | head-driven relations, controlled syntactic extensions | *right* -> prep *to* -> *erasure* |
| **Surface n-grams** (baseline) | every 1..n-gram, filtered by stop words at the boundaries | used in the notebook |

### 3.1 POS Patterns Are Filters, Not Definitions

A POS pattern guarantees a **grammatical** candidate, not a **term**:

| Pattern | Candidate | Linguistically | Terminology status |
|---|---|---|---|
| `ADJ NOUN` | *personal data* | well formed | valid legal term |
| `ADJ NOUN` | *important decision* | well formed | generic language (noise) |
| `NOUN NOUN` | *data controller* | well formed | valid legal term |
| `NOUN NOUN` | *Monday meeting* | well formed | transient phrase |

Over-generation is acceptable at this stage: the noise is removed by the ranker.

## 4. Statistical Signals

### 4.1 Frequency

$\mathrm{TF}(t)$ = count of candidate $t$ in the domain corpus. Cheap and often correlated with relevance, but generic expressions (*important decision*, *data*) can be very frequent. Frequency says *common here*, not *specific to here*.

### 4.2 TF-IDF

TF-IDF ([[TF-IDF and Static Embeddings]]) measures how **salient a candidate is for a document** relative to the collection. It is a keyword signal: high TF-IDF $\neq$ domain-wide termhood. A term that appears in **every** document of a legal corpus (e.g. *personal data* in a GDPR corpus) gets a **low** IDF exactly because it is pervasive - the opposite of what termhood wants.

### 4.3 Domain Specificity (Corpus Contrast)

Compare relative frequency in the **domain corpus** with relative frequency in a **general reference corpus**:

$$
\mathrm{Specificity}(t) \approx \frac{p(t \mid \text{domain})}{p(t \mid \text{reference})}
\qquad\text{or, in log form (notebook)}\qquad
\mathrm{spec}(t) = \log_2 \frac{p(t \mid \text{domain})}{p(t \mid \text{reference})}
$$

with additive smoothing so that a candidate absent from the reference corpus does not divide by zero. This family of measures is known in the terminology literature as *weirdness* (medium).

| Candidate | Legal frequency | Reference frequency | Specificity |
|---|---|---|---|
| *court* | high | also high (sports court, courtroom dramas) | moderate, near 0 in log form |
| *legitimate interest* | high | almost absent | **high** |
| *important decision* | high | equally high | low (~0) |

**Worked example** (toy counts, smoothing $\epsilon = 0.5$, both corpora assumed to have the same normaliser $Z$, so it cancels):

- *legal certainty*: 2 legal, 0 reference -> $\log_2 \frac{2.5}{0.5} = \log_2 5 \approx 2.32$ -> clearly legal.
- *court*: 3 legal, 3 reference -> $\log_2 \frac{3.5}{3.5} = 0$ -> generic.

Interpretation: $>0$ concentrated in the domain, $\approx 0$ generic, $<0$ more typical of general language. **The signal is only as good as the reference corpus**: too small and most candidates have reference count 0 (every candidate looks "specific", and the score collapses into domain frequency); badly matched (e.g. a reference corpus that is itself partly legal) and true terms get penalized.

### 4.4 Association Measures (Unithood): PMI

$$
\mathrm{PMI}(x, y) = \log_2 \frac{P(x, y)}{P(x)\,P(y)}
$$

PMI compares observed co-occurrence with what independence would predict. With $N$ tokens and counts $f$, $\mathrm{PMI} = \log_2 \frac{f(x,y)\, N}{f(x)\, f(y)}$.

**Worked example** ($N = 300$):

| Bigram | $f(x)$ | $f(y)$ | $f(x,y)$ | PMI |
|---|---|---|---|---|
| *data protection* | 20 | 6 | 5 | $\log_2 \frac{5 \cdot 300}{120} = \log_2 12.5 \approx 3.64$ |
| a one-off pair of hapaxes | 1 | 1 | 1 | $\log_2 300 \approx 8.23$ |

The accidental pair beats the genuine unit: **PMI overvalues rare events**. Remedy: combine association thresholds with an **absolute frequency floor** (the notebook ignores bigrams seen fewer than 2 times).

## 5. The Nested Term Problem: C-value and NC-value

### 5.1 The Problem

Longer terms contain shorter (often valid) terms:

$$
\text{data} \subset \text{personal data} \subset \text{sensitive personal data}
\qquad
\text{personal data} \subset \text{transfer of personal data}
$$

Every occurrence of a long term is also counted as an occurrence of each of its sub-strings, so raw frequency systematically **favours short sub-expressions**, including fragments that are never used on their own (*protection obligations* inside *data protection obligations*).

### 5.2 C-value

C-value (Frantzi, Ananiadou and Mima, 2000) rewards length and frequency and **discounts frequency inherited from longer candidates that contain the candidate**:

$$
\text{C-value}(a) =
\begin{cases}
\log_2 |a| \cdot f(a) & \text{if } a \text{ is not nested} \\[4pt]
\log_2 |a| \cdot \left( f(a) - \dfrac{1}{|T_a|} \displaystyle\sum_{b \in T_a} f(b) \right) & \text{otherwise}
\end{cases}
$$

| Symbol | Meaning |
|---|---|
| $\lvert a \rvert$ | length of $a$ in words |
| $f(a)$ | frequency of $a$ |
| $T_a$ | set of **longer** candidates containing $a$ |
| $\lvert T_a \rvert$ | number of such candidates |
| $\frac{1}{\lvert T_a \rvert}\sum_{b \in T_a} f(b)$ | **average** frequency of the containing candidates |

Why each piece:

- $\log_2 |a|$: longer candidates are less likely to recur by accident, so length is a bonus. For $|a| = 1$, $\log_2 1 = 0$, which would erase every single-word candidate: the original method targets multiword terms; the notebook patches this by using weight $1.0$ for unigrams.
- Subtracting the **average** (not the sum) of containing frequencies: if $a$ occurs inside **many different** longer candidates, it is used productively in many combinations - evidence that it is an independent term - so the discount is diluted.

**Worked example** (toy counts; unigram weight 1 as in the notebook):

| Candidate | $f(a)$ | $T_a$ (frequency) | Computation | C-value |
|---|---|---|---|---|
| *data* | 20 | personal data (8), data controller (5), sensitive personal data (2), transfer of personal data (3) | $1 \cdot (20 - 18/4)$ | 15.5 |
| *personal data* | 8 | sensitive personal data (2), transfer of personal data (3) | $\log_2 2 \cdot (8 - 2.5)$ | 5.5 |
| *sensitive personal data* | 2 | none | $\log_2 3 \cdot 2$ | 3.17 |
| *transfer of personal data* | 3 | none | $\log_2 4 \cdot 3$ | 6.0 |
| *data protection obligations* | 3 | none | $\log_2 3 \cdot 3$ | 4.75 |
| *protection obligations* | 3 | data protection obligations (3) | $\log_2 2 \cdot (3 - 3)$ | **0** |

Key reading: a fragment that **only ever occurs inside one longer term** gets C-value 0 - exactly the case raw frequency gets wrong. Note also that *data* still scores high under the unigram patch: C-value controls nesting, not genericity, which is why it needs a specificity signal next to it.

### 5.3 NC-value

Terms appear in characteristic contexts: *pursuant to ___*, *shall ___*, *under Article ___*, *competent ___*. **NC-value** adds this **candidate-external** evidence to C-value's **candidate-internal** evidence (length, frequency, nesting):

1. Take the top C-value candidates as provisional terms.
2. Collect the **context words** (nouns, adjectives, verbs) appearing next to them; weight each context word $w$ by how many of those terms it occurs with: $\mathrm{weight}(w) = t(w)/n$, where $t(w)$ = number of terms $w$ appears with and $n$ = number of terms considered.
3. Re-rank every candidate:

$$
\text{NC-value}(a) = 0.8 \cdot \text{C-value}(a) + 0.2 \sum_{w \in C_a} f_a(w)\, \mathrm{weight}(w)
$$

where $C_a$ is the set of context words of $a$ and $f_a(w)$ how often $w$ appears in the context of $a$. The 0.8 / 0.2 weights are the original paper's empirical choice (high). Neither the lecture source nor the notebook implements NC-value; the formula is given here for completeness.

## 6. Hybrid and Resource-Aware Ranking

### 6.1 Multi-Signal Fusion

No single signal captures "termness". The state of the art combines them:

$$
\text{linguistic constraints} + \text{frequency} + \text{domain specificity} + \text{C-value / NC-value} \longrightarrow \text{combined ranker}
$$

The combiner can be a weighted score, a learned ranker (learning-to-rank over the signals as features), or rules - and is then intersected with external knowledge resources.

### 6.2 Knowledge Resources as Evidence, Not Filters

Glossaries, SKOS vocabularies and termbases such as EuroVoc ([[Knowledge Resources for Text Analytics]]) contribute in four ways:

| Use | Effect |
|---|---|
| Known-term matching | validate candidates or seed the pipeline |
| Normalization and synonyms | map orthographic/lexical variants to one concept |
| Semantic neighbourhoods | use the termbase's structure to find related concepts |
| **Discovery preservation** | a candidate absent from the vocabulary must **not** be suppressed |

The last point is what separates ATE from dictionary lookup: the whole point is to find terms the vocabulary does not yet have. A resource match is a **bonus**, never a gate.

## 7. Supervised, Neural and Generative Approaches

With annotated data, ATE becomes a prediction problem:

| Paradigm | Formulation | Strength | Weakness |
|---|---|---|---|
| **Token classification** | BIO/BILOU tagging with a BERT-style encoder ([[Annotation and Evaluation of Information Extraction]]) | learns domain annotation conventions | flat: cannot natively represent nested terms |
| **Span classification** | score candidate spans directly ([[Advanced Named Entity Recognition]]) | handles overlapping and nested terms | quadratic number of candidate spans |
| **Prompt-based / generative** | LLM with open-label or ontology-guided prompts | no training data, can use definitions | hallucinated terms, span drift, run-to-run variability |

Trade-off: supervised models fit **the annotation scheme they were trained on** and can generalize poorly across domains, where unsupervised statistical baselines are more portable.

## 8. Experimental Design: Systems A-F

The lecture's methodological core: **design comparisons, not just systems**.

| System | Candidate generator | Ranking / model | External knowledge | Research question |
|---|---|---|---|---|
| **A** | raw n-grams | frequency | none | transparent string-statistics baseline |
| **B** | POS patterns | C-value | none | syntax + nesting discount |
| **C** | POS patterns | domain specificity | reference corpus | contribution of corpus contrast |
| **D** | span/token model | supervised score | term annotations | learned legal ATE |
| **E** | hybrid pipeline | combined ranker | legal vocabularies | knowledge-guided fusion |
| **F** | LLM prompt | generative validation | optional retrieval | LLM reasoning for ATE |

Reading the matrix: **B vs C** share the generator and differ only in the ranking signal, so their difference isolates *nesting discount vs reference-corpus contrast*. **A vs B** changes two things at once (generator and ranker) - a confound to keep in mind when interpreting it.

### 8.1 Guidelines for DETECH 2027

1. **Isolate signal contributions** - vary one or two factors at a time.
2. **Standardize the pipeline** - target and reference corpora -> candidate generation -> scoring -> ranking -> gold-standard evaluation, identical across systems.
3. **Treat models as a toolbox** - POS patterns, statistics, neural representations, knowledge bases and LLM prompts are complementary signals; measure which combinations give verifiable precision gains.

The DETECH 2026 proceedings (CEUR-WS Vol. 4215) already span this whole range: rule-based extraction, BERT sequence labeling and domain-adapted transformers (e.g. biomedical), LLM prompting, ontology-enhanced prompting with knowledge-resource grounding, and theory-driven terminology frameworks.

### 8.2 Questions to Settle Before Any Experiment

- What counts as a **term** under the annotation guidelines?
- Are **nested and overlapping** terms allowed in the gold standard?
- Is evaluation over **mentions**, **types**, or a **ranked list** (Precision@k / Recall@k)?
- Which errors matter most for the downstream terminology workflow?

## 9. Exam Cheat Sheet

| Signal | Answers | Axis | Main pitfall |
|---|---|---|---|
| Frequency | common here? | termhood (weak) | generic phrases rank high |
| TF-IDF | salient in a document? | keyword salience | pervasive domain terms get low IDF |
| Domain specificity | more frequent here than in general language? | termhood | bounded by reference-corpus quality |
| PMI | do the words stick together? | unithood | overvalues rare pairs -> frequency floor |
| C-value | frequent independently of longer terms? | termhood + nesting | unigrams need a patch; ignores genericity |
| NC-value | appears in term-like contexts? | termhood + context | needs a first good C-value ranking |
| Knowledge resources | already known concept? | validation | must not suppress new terms |

## 10. Review Questions

1. Distinguish term, named entity and keyword with one legal example each.
2. Define termhood and unithood; which signals measure each?
3. Why must candidate generation favour recall? What happens to a term it misses?
4. Why is high TF-IDF not the same as high termhood?
5. Compute the C-value of a bigram that occurs 3 times, always inside the same trigram. What does the result mean?
6. Why does C-value subtract the **average** frequency of the containing terms instead of the sum?
7. How does NC-value extend C-value, and what kind of evidence does it add?
8. Which pair of Systems A-F isolates the effect of the reference corpus, and why?
9. Why should a hybrid ranker treat EuroVoc matches as a bonus rather than a filter?

## Related

- [[Automatic Term Extraction - Notebook]] - companion walkthrough of the L15 notebook
- [[Legal Information Extraction and Knowledge Graphs]] - Lecture 14: legal NER, SKOS, EuroVoc
- [[Biomedical Named Entity Recognition and Concept Normalization]] - same candidate generation vs ranking split
- [[TF-IDF and Static Embeddings]] - TF-IDF and association strength
- [[Information Retrieval Foundations]] - cross-course bridge ([[Privacy Preserving Information Access]]): TF-IDF and ranked-list evaluation

## Sources

- [[ATA_L15]] (AY 2026/2027)
- [[ATA_L15_notebook]] (AY 2026/2027)
- Frantzi, Ananiadou and Mima (2000), *Automatic recognition of multi-word terms: the C-value/NC-value method*, Int. J. Digital Libraries 3(2) - used for the NC-value formula and weights in Section 5.3
