---
type: course-note
date: 2026-10-19
tags:
  - course
  - advanced-text-analytics
  - text-representation
  - embeddings
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L07]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 7) on **turning text into numerical representations** - the ladder Bag of Words -> TF-IDF -> cosine similarity -> dense static embeddings -> a tiny Skip-gram neural model implemented from scratch with NumPy. Created 2026-10-19 from [[ATA_L07]] (English NotebookLM export of the lecture notebooks). Use it when a question involves sparse lexical features, TF-IDF weighting and normalization, cosine similarity, the distributional hypothesis, Word2Vec/Skip-gram training mechanics, or the known limits of static embeddings. The recurring thesis: **every representation is a trade-off** - lexical schemes keep counts but lose geometry, embeddings gain geometry but stay frozen per word type, and only contextual representations (next lecture) resolve meaning per occurrence.

## 1. Why Representation Matters

A text analytics model cannot consume words directly: something must produce a vector first. The lecture is built as a ladder, and the running question at every step is *what does this conversion keep, and what does it throw away?*

```text
Bag of Words -> TF-IDF -> cosine similarity -> dense embeddings -> tiny neural Skip-gram
```

- Counts survive in Bag of Words.
- Weights survive in TF-IDF.
- Geometry survives in embeddings.

The final step is implemented from scratch with NumPy, deliberately avoiding `gensim`, so that the vocabulary, the training pairs, the two weight matrices, and the gradient updates are all visible.

## 2. Bag of Words

Bag of Words (BoW) represents a document by the counts of its terms: the vocabulary contributes one dimension per distinct term, and each document becomes a row of counts. The lecture's opening example:

```text
dog bites man    ->  [bites: 1, dog: 1, man: 1]
man bites dog    ->  [bites: 1, dog: 1, man: 1]
```

The two rows are **identical**. Whoever does the biting changed, and BoW cannot say so. It preserves lexical counts and nothing else: word order is gone, syntactic roles are gone. The toy example states the limit precisely - this is the first entry in the "what is lost" thread of the lecture.

## 3. The Working Corpus

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

## 4. TF-IDF

TF-IDF reweights the counts in one line:

$$\text{TF-IDF}(t,d) = \text{TF}(t,d) \times \text{IDF}(t)$$

A term matters more when it is **frequent in one document and rare across the collection**. In scikit-learn, with the default smoothing, for a corpus of $n$ documents and a term appearing in $df$ documents:

$$\text{idf}(t) = \ln\frac{1 + n}{1 + df} + 1$$

After weighting, each document vector is **L2-normalized**.

### Worked example (BIO-1, corpus of six documents)

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

## 5. Cosine Similarity

Two documents are compared by the angle between their vectors:

$$\cos(x, y) = \frac{x \cdot y}{\|x\| \times \|y\|}$$

Cosine compares **direction, not size**. A long document and a short one with the same term proportions have the same cosine against a third document, while their raw dot products would differ substantially. This is why cosine is the default choice for sparse text similarity (and why TF-IDF normalizes vectors first).

Results on the lecture corpus:

| Pair | Cosine | Why |
|---|---|---|
| BIO-1 vs BIO-2 | 0.669 | share `mutation`, `breast`, `cancer` |
| any BIO vs any LEGAL | 0.000 | no shared terms at all |
| LEGAL-1 vs LEGAL-2 | 0.675 | share `regulation`, `data`, `protection` |
| LEGAL-3 vs LEGAL-1 | 0.138 | only `regulation` in common |
| LEGAL-3 vs LEGAL-2 | 0.181 | only `regulation` in common |

Documents from different domains end up **orthogonal**: BoW and TF-IDF cannot see any similarity between texts that use different words.

## 6. TF-IDF Is Still Lexical

The lecture tests the obvious weakness with two near-duplicate clinical sentences:

```text
S1: the patient suffered a heart attack
S2: the patient suffered a myocardial infarction
```

Cosine similarity on their TF-IDF vectors gives **0.432**. The breakdown explains why:

- Shared tokens: `the`, `patient`, `suffered`.
- `heart` and `attack` have no counterpart in S2; `myocardial` and `infarction` have none in S1.
- `a` disappears again (two-character token pattern).

So the score comes almost entirely from the **sentence frame**, and the medical content contributes zero. TF-IDF measures surface overlap; it has no way to know that "heart attack" and "myocardial infarction" can denote the same concept. This is the pivot point of the lecture: lexical weighting alone cannot fix a lexical problem.

## 7. Distributional Semantics

The fix is to let **context define meaning**: words occurring in similar contexts acquire similar representations.

```text
aspirin treats pain
ibuprofen treats pain
aspirin reduces inflammation
ibuprofen reduces inflammation
```

Nothing tells the model that aspirin and ibuprofen belong to the same class - the shared frames do the work. This is the **distributional hypothesis**, the principle behind everything that follows (and the same idea that unifies Lecture 5's thesaurus/ontology resources with machine-learned representations).

## 8. Static Embeddings

Instead of one sparse dimension per vocabulary word, each word gets a **dense vector**, e.g.:

```text
BRCA1 -> [0.21, -0.14, 0.77, ...]
TP53  -> [0.18, -0.09, 0.71, ...]
```

The lecture builds a toy 2D space by hand to show what "dense" means geometrically. Six words, and the two domains separate on the first dimension:

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

## 9. Word2Vec and Skip-gram Intuition

Skip-gram learns vectors by **inverting a prediction task**: given a center word, predict the surrounding words.

```text
center word -> predict surrounding words
```

The embedding is not given to the model - it is an internal parameter shaped as a side effect of solving that task.

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

## 10. The Tiny Neural Model

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

## 11. How Does the Model Learn Strength of Association?

A subtlety the lecture isolates: in Skip-gram every observed pair is a positive example with the **same target label**, so `(dog, barks)` and `(dog, runs)` look identical from the loss's point of view in a single occurrence. Yet "barks" ends up closer to "dog" than "runs" does, for two reasons:

1. **Pair frequency feeds the gradient directly**: if `(dog, barks)` occurs 100 times and `(dog, runs)` 20 times, the first pair pushes the vectors 100 times and the second only 20.
2. **Negative sampling** gives words negative updates when they are drawn as noise for other pairs.

Association strength is therefore learned from the **aggregate distribution of positive and negative examples**, never from assigning different target values to different positive pairs.

## 12. Nearest Neighbors (and Why to Distrust Them Here)

After training, a word's embedding is simply its row of `W_in`; nearest neighbors are ranked by cosine similarity. The lecture's run:

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

## 13. Domain Dependence

Embeddings reflect the corpus they were trained on. The lecture's example is the word **"appeal"**: in general English it can mean attractiveness; in legal language it is a procedure before a court. A general-domain model and a legal-domain model would place "appeal" next to different words. There is no context-free "true" meaning in this framework - only the contexts a particular corpus supplies.

## 14. FastText Intuition

Static word vectors treat a word as an **indivisible token**. FastText builds representations from **character n-grams** instead, which is useful when words share subword structure:

```text
neuroblastoma
glioblastoma
lymphoblastoma
```

The shared suffix `-blastoma` lets information flow between these forms. Subword modeling also helps **rare words and words never seen in training**, as long as their character sequences have been seen somewhere.

## 15. The One-Vector-per-Type Limit

The final limitation of static embeddings is stated with a single pair:

```text
river bank       ->  bank = one vector
investment bank  ->  bank = the same one vector
```

A static embedding assigns **one vector per word type**, so the senses of "bank" are merged into a single point. Nothing in the representation changes when the word appears in a different sentence. Contextual representations, which the course takes up next, produce a **vector per occurrence** instead.

## 16. Sparse against Dense

| Property | TF-IDF | Static embeddings |
|---|---|---|
| Representation | sparse | dense |
| Typical unit | document/term features | word |
| Dimensions | vocabulary-sized | lower-dimensional |
| Learned from context | no | yes |
| Context-sensitive meaning | no | no |

The last row is often missed: static embeddings **learn from context during training, but once trained they are frozen** - the vector for "bank" is the same in every sentence. Both representations are therefore static *with respect to the text being processed*.

## 17. Final Progression

```text
Bag of Words
    |
    v
TF-IDF
    |
    v
Static embeddings
    |
    v
Contextual embeddings
```

Take-home messages:

1. Bag of Words and TF-IDF are explicit **lexical** representations.
2. Cosine similarity compares **vector geometry**, not document length.
3. Static embeddings are **dense, learned** word representations.
4. A simple neural context-prediction task can learn an embedding matrix from co-occurrence alone.
5. Embeddings **depend on their training corpus** (domain dependence).
6. Static embeddings still assign **one vector to each word type** - the gap that contextual embeddings close.

## Related

- [[Advanced Text Analytics]] - course hub
- [[NLP Linguistic Structures and Annotation]] - the word/token distinction that defines the vocabulary and the token pattern filters seen here

## Sources

- [[ATA_L07]] (AY 2026/2027)
