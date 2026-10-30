---
type: course-note
date: 2026-10-28
tags:
  - course
  - advanced-text-analytics
  - notebook
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L07]]"
ai-first: true
---

## For Future Agent

Notebook companion for [[TF-IDF and Static Embeddings]] (Lecture 7), extracted 2026-10-28: it collects only the passages that document what the lecture's Python notebook actually runs (corpus setup, TF-IDF worked numbers, cosine results, the Skip-gram training run and its nearest neighbors). The representation theory stays in the parent note. Use it when a question is about the concrete numbers or code behavior of the demos.

## 1. The Working Corpus

For the sparse half of the lecture the notebook uses six short documents split between two domains:

```text
BIO-1   : BRCA1 mutation breast cancer
BIO-2   : TP53 mutation breast cancer
BIO-3   : myocardial infarction heart disease
LEGAL-1 : Regulation data protection personal data
LEGAL-2 : Article 6 Regulation data protection
LEGAL-3 : Court applies Regulation
```

The vocabulary has 16 terms. Two implementation details matter:

1. `data` occurs **twice** inside LEGAL-1, so its count is 2, not 1 - counts genuinely matter.
2. The `6` in "Article 6" never becomes a feature: the default scikit-learn token pattern keeps only tokens of at least two word characters (`\b\w\w+\b`-style filtering), which also drops "a" later in the lecture's sentence experiment.

## 2. TF-IDF Worked Example (BIO-1)

```text
df(BRCA1) = 1   ->  idf = ln(7/2) + 1 = 2.253
df(mutation) = df(breast) = df(cancer) = 2  ->  idf = ln(7/3) + 1 = 1.847

raw scores : [2.253, 1.847, 1.847, 1.847]
L2 norm    : sqrt(2.253^2 + 3 x 1.847^2) = 3.913

BRCA1    : 2.253 / 3.913 = 0.576
mutation : 1.847 / 3.913 = 0.472
breast   : 0.472
cancer   : 0.472
```

Reading of the numbers:

- The **gene name gets the largest weight** because only one document contains it (highest IDF).
- The three shared disease terms are discounted.
- The same pattern repeats in BIO-2, where `TP53` takes 0.576 and the shared terms sit at 0.472.
- In the legal rows the situation flips: `regulation` appears in three documents, so it ranks **below** `data` and `protection`; in LEGAL-2 the highest weight belongs to `article` (0.595), which occurs only once in the corpus.

## 3. Cosine Similarity Results

| Pair | Cosine | Why |
|---|---|---|
| BIO-1 vs BIO-2 | 0.669 | share `mutation`, `breast`, `cancer` |
| any BIO vs any LEGAL | 0.000 | no shared terms at all |
| LEGAL-1 vs LEGAL-2 | 0.675 | share `regulation`, `data`, `protection` |
| LEGAL-3 vs LEGAL-1 | 0.138 | only `regulation` in common |
| LEGAL-3 vs LEGAL-2 | 0.181 | only `regulation` in common |

Documents from different domains end up **orthogonal**: BoW and TF-IDF cannot see any similarity between texts that use different words.

## 4. The Sentence Experiment

The notebook tests the lexical weakness with two near-duplicate clinical sentences:

```text
S1: the patient suffered a heart attack
S2: the patient suffered a myocardial infarction
```

Cosine similarity on their TF-IDF vectors gives **0.432**. The breakdown explains why:

- Shared tokens: `the`, `patient`, `suffered`.
- `heart` and `attack` have no counterpart in S2; `myocardial` and `infarction` have none in S1.
- `a` disappears again (two-character token pattern).

So the score comes almost entirely from the **sentence frame**, and the medical content contributes zero. TF-IDF measures surface overlap; it has no way to know that "heart attack" and "myocardial infarction" can denote the same concept. This is the pivot point of the lecture: lexical weighting alone cannot fix a lexical problem.

## 5. The Hand-Built Dense Space

The notebook builds a toy 2D space by hand to show what "dense" means geometrically. Six words, and the two domains separate on the first dimension:

```text
BRCA1 [0.90, 0.85]        TP53 [0.85, 0.65]        cancer [0.75, 0.65]
regulation [-0.75, 0.60]  court [-0.65, 0.50]      privacy [-0.55, 0.70]
```

Cosine similarity between hand-set vectors:

```text
regulation vs court:
   dot   = 0.4875 + 0.3000 = 0.7875
   norms : 0.960 x 0.820 = 0.788
   cos   = 0.9998

BRCA1 vs TP53       : 0.995
BRCA1 vs regulation : -0.139
```

Similarity becomes a **geometric property**: two words are close when their vectors point in similar directions, **even if they never co-occur in a document**. That is exactly what TF-IDF cannot offer, and it is why "different words can be close" appears in the lecture's final comparison table.

## 6. Skip-gram Lecture Setup

Lecture setup:

- Tiny corpus: **11 token lists** mixing three registers - gene sentences (`brca1 mutation breast cancer`), drug sentences (`aspirin treats pain`), legal sentences (`court applies regulation`).
- Vocabulary: **22 word types**.
- Window: **2** - each center word collects neighbors within two positions on either side.

For the sentence `brca1 mutation breast cancer`, the pairs begin:

```text
brca1    -> mutation
brca1    -> breast
mutation -> brca1
mutation -> breast
mutation -> cancer
breast   -> brca1
...
```

Window boundaries cut the counts: a four-token sentence yields 10 pairs, a three-token sentence yields 6, so the whole corpus produces **90 pairs**. Those 90 pairs are the **entire supervision signal** - no entity labels, no semantic tags, only local co-occurrence.

## 7. The Tiny Neural Model

The model is two matrices:

```text
W_in  : 22 x 12   (vocabulary -> embedding)
W_out : 12 x 22   (embedding -> vocabulary)
```

- A row of `W_in` **is** the learned embedding for that word.
- Initialization: both matrices drawn from $\mathcal{N}(0, 0.1^2)$ (mean 0, standard deviation 0.1), fixed seed for reproducibility.

Training step for one pair (center, context):

```text
1. h           = W_in[center]            # retrieve the embedding
2. scores      = h . W_out               # 22 raw scores
3. probs       = softmax(scores)         # distribution over the vocabulary
4. loss       += -log(probs[context])    # cross-entropy on the true word
5. dscores     = probs with 1 subtracted at the context word
6. grad W_out  = outer(h, dscores)
7. grad h      = W_out . dscores
8. W_out       -= learning rate x grad W_out
9. W_in[center] -= learning rate x grad h
```

Implementation details worth memorizing:

- The softmax **subtracts its maximum before exponentiating** (numerical stability).
- Only the row of `W_in` for the current center word is updated.
- This is a **full-softmax** model, affordable only because the vocabulary has 22 entries; real Word2Vec uses **negative sampling** for vocabularies of millions.

Training run: **300 epochs at learning rate 0.05**, average loss falls from **3.092 to 1.471** - fast drop at the start, then flattening. Interpretation caveat: a decreasing loss means the model predicts the observed context words of *this specific corpus* better; that is **not the same as learning a general semantic space**.

## 8. Nearest Neighbors After Training

After training, a word's embedding is simply its row of `W_in`; nearest neighbors are ranked by cosine similarity. The notebook's run:

```text
brca1     : risk 0.566, tp53 0.488, breast 0.462, mutation 0.426, cancer 0.331
aspirin   : ibuprofen 0.998, applies 0.173, breast 0.157, pain 0.127
regulation: personal 0.396, court 0.199, article 0.185, applies 0.172
```

Direct similarity comparisons:

| Pair | Cosine | Reading |
|---|---|---|
| aspirin vs ibuprofen | 0.998 | identical contexts |
| brca1 vs tp53 | 0.488 | shared, but not identical, contexts |
| brca1 vs regulation | -0.261 | different domains |

Some of it looks right: ibuprofen is aspirin's twin because the corpus gives them identical frames, and BRCA1's neighbors are biomedical words. The lecture's warning should not be skipped: **this corpus is far too small and the neighbors are unstable** - the exercise is about the learning mechanism, not about the quality of the resulting space.

The companion exercise notebook extends the corpus with four sentences putting `brca1` and `tp53` into the same frames ("brca1 is a gene", "tp53 is a gene", "brca1/tp53 mutation increases cancer risk"). Retraining with the same procedure lifts their similarity from **0.488 to 0.708**, and they become each other's nearest neighbor. Repeated, similar contexts pull words toward the same region of the space - exactly what the distributional hypothesis predicts. Caveat: the comparison is **illustrative rather than controlled**, since adding sentences also grows the vocabulary and changes the initialization.

## Related

- [[TF-IDF and Static Embeddings]] - parent theory note (representation ladder, formulas, limits)
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L07]] (AY 2026/2027)
