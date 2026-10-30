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

Study note for the [[Advanced Text Analytics]] course (Lecture 7) on **turning text into numerical representations** - the ladder Bag of Words -> TF-IDF -> cosine similarity -> dense static embeddings -> a tiny Skip-gram neural model implemented from scratch with NumPy. Created 2026-10-19 from [[ATA_L07]] (English NotebookLM export of the lecture notebooks). Use it when a question involves sparse lexical features, TF-IDF weighting and normalization, cosine similarity, the distributional hypothesis, Word2Vec/Skip-gram training mechanics, or the known limits of static embeddings. The recurring thesis: **every representation is a trade-off** - lexical schemes keep counts but lose geometry, embeddings gain geometry but stay frozen per word type, and only contextual representations (next lecture) resolve meaning per occurrence. The concrete notebook walkthroughs (corpus, worked TF-IDF numbers, cosine results, the NumPy Skip-gram run) live in [[TF-IDF and Static Embeddings - Notebook]].

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

## 3. TF-IDF

TF-IDF reweights the counts in one line:

$$\text{TF-IDF}(t,d) = \text{TF}(t,d) \times \text{IDF}(t)$$

A term matters more when it is **frequent in one document and rare across the collection**. In scikit-learn, with the default smoothing, for a corpus of $n$ documents and a term appearing in $df$ documents:

$$\text{idf}(t) = \ln\frac{1 + n}{1 + df} + 1$$

After weighting, each document vector is **L2-normalized**. The worked numbers for the lecture corpus (BIO-1's weights, the legal rows, token-pattern quirks) are in [[TF-IDF and Static Embeddings - Notebook#2. TF-IDF Worked Example (BIO-1)]].

## 4. Cosine Similarity

Two documents are compared by the angle between their vectors:

$$\cos(x, y) = \frac{x \cdot y}{\|x\| \times \|y\|}$$

Cosine compares **direction, not size**. A long document and a short one with the same term proportions have the same cosine against a third document, while their raw dot products would differ substantially. This is why cosine is the default choice for sparse text similarity (and why TF-IDF normalizes vectors first). The numeric results on the lecture corpus (including the 0.000 cross-domain pairs) are in [[TF-IDF and Static Embeddings - Notebook#3. Cosine Similarity Results]].

## 5. TF-IDF Is Still Lexical

The lecture tests the obvious weakness with two near-duplicate clinical sentences differing only in their medical content ("heart attack" vs "myocardial infarction"). Their TF-IDF cosine comes out at **0.432**, built almost entirely from the shared sentence frame (`the`, `patient`, `suffered`), while the medical terms contribute zero - neither side has a counterpart for the other's words. Full breakdown in [[TF-IDF and Static Embeddings - Notebook#4. The Sentence Experiment]].

TF-IDF measures surface overlap; it has no way to know that "heart attack" and "myocardial infarction" can denote the same concept. This is the pivot point of the lecture: lexical weighting alone cannot fix a lexical problem.

## 6. Distributional Semantics

The fix is to let **context define meaning**: words occurring in similar contexts acquire similar representations.

```text
aspirin treats pain
ibuprofen treats pain
aspirin reduces inflammation
ibuprofen reduces inflammation
```

Nothing tells the model that aspirin and ibuprofen belong to the same class - the shared frames do the work. This is the **distributional hypothesis**, the principle behind everything that follows (and the same idea that unifies Lecture 5's thesaurus/ontology resources with machine-learned representations).

## 7. Static Embeddings

Instead of one sparse dimension per vocabulary word, each word gets a **dense vector**, e.g.:

```text
BRCA1 -> [0.21, -0.14, 0.77, ...]
TP53  -> [0.18, -0.09, 0.71, ...]
```

The lecture builds a toy 2D space by hand to show what "dense" means geometrically: six words, two domains separating on the first dimension, hand-set cosine similarities (e.g. regulation vs court at 0.9998, BRCA1 vs regulation at -0.139) - full walkthrough in [[TF-IDF and Static Embeddings - Notebook#5. The Hand-Built Dense Space]].

Similarity becomes a **geometric property**: two words are close when their vectors point in similar directions, **even if they never co-occur in a document**. That is exactly what TF-IDF cannot offer, and it is why "different words can be close" appears in the lecture's final comparison table.

## 8. Word2Vec and Skip-gram Intuition

Skip-gram learns vectors by **inverting a prediction task**: given a center word, predict the surrounding words.

```text
center word -> predict surrounding words
```

The embedding is not given to the model - it is an internal parameter shaped as a side effect of solving that task.

The lecture's concrete setup (11 token lists, 22-word vocabulary, window 2, the 90 training pairs) is documented in [[TF-IDF and Static Embeddings - Notebook#6. Skip-gram Lecture Setup]]. Those 90 pairs are the **entire supervision signal** - no entity labels, no semantic tags, only local co-occurrence.

## 9. The Tiny Neural Model

Conceptually the model is two matrices, $W_{in}$ (vocabulary -> embedding) and $W_{out}$ (embedding -> vocabulary): a row of $W_{in}$ **is** the learned embedding, and training a pair means retrieving the center embedding, scoring the vocabulary through $W_{out}$, softmaxing, taking the cross-entropy loss on the true context word, and descending on both matrices (only the center row of $W_{in}$ moves). This is a **full-softmax** model, affordable only because the lecture's vocabulary has 22 entries; real Word2Vec uses **negative sampling** for vocabularies of millions.

The full training-step pseudocode, the numerical-stability details, and the training run (300 epochs at learning rate 0.05, loss 3.092 -> 1.471) are in [[TF-IDF and Static Embeddings - Notebook#7. The Tiny Neural Model]]. Interpretation caveat: a decreasing loss means the model predicts the observed context words of *this specific corpus* better; that is **not the same as learning a general semantic space**.

## 10. How Does the Model Learn Strength of Association?

A subtlety the lecture isolates: in Skip-gram every observed pair is a positive example with the **same target label**, so `(dog, barks)` and `(dog, runs)` look identical from the loss's point of view in a single occurrence. Yet "barks" ends up closer to "dog" than "runs" does, for two reasons:

1. **Pair frequency feeds the gradient directly**: if `(dog, barks)` occurs 100 times and `(dog, runs)` 20 times, the first pair pushes the vectors 100 times and the second only 20.
2. **Negative sampling** gives words negative updates when they are drawn as noise for other pairs.

Association strength is therefore learned from the **aggregate distribution of positive and negative examples**, never from assigning different target values to different positive pairs.

## 11. Nearest Neighbors (and Why to Distrust Them Here)

After training, a word's embedding is simply its row of `W_in`; nearest neighbors are ranked by cosine similarity. The lecture's run and the exercise's `brca1`/`tp53` retraining experiment (0.488 -> 0.708) are in [[TF-IDF and Static Embeddings - Notebook#8. Nearest Neighbors After Training]].

Some of the raw results look right: ibuprofen is aspirin's twin because the corpus gives them identical frames, and BRCA1's neighbors are biomedical words. The lecture's warning should not be skipped: **this corpus is far too small and the neighbors are unstable** - the exercise is about the learning mechanism, not about the quality of the resulting space. The retraining comparison is in any case **illustrative rather than controlled**, since adding sentences also grows the vocabulary and changes the initialization.

## 12. Domain Dependence

Embeddings reflect the corpus they were trained on. The lecture's example is the word **"appeal"**: in general English it can mean attractiveness; in legal language it is a procedure before a court. A general-domain model and a legal-domain model would place "appeal" next to different words. There is no context-free "true" meaning in this framework - only the contexts a particular corpus supplies.

## 13. FastText Intuition

Static word vectors treat a word as an **indivisible token**. FastText builds representations from **character n-grams** instead, which is useful when words share subword structure:

```text
neuroblastoma
glioblastoma
lymphoblastoma
```

The shared suffix `-blastoma` lets information flow between these forms. Subword modeling also helps **rare words and words never seen in training**, as long as their character sequences have been seen somewhere.

## 14. The One-Vector-per-Type Limit

The final limitation of static embeddings is stated with a single pair:

```text
river bank       ->  bank = one vector
investment bank  ->  bank = the same one vector
```

A static embedding assigns **one vector per word type**, so the senses of "bank" are merged into a single point. Nothing in the representation changes when the word appears in a different sentence. Contextual representations, which the course takes up next, produce a **vector per occurrence** instead.

## 15. Sparse against Dense

| Property | TF-IDF | Static embeddings |
|---|---|---|
| Representation | sparse | dense |
| Typical unit | document/term features | word |
| Dimensions | vocabulary-sized | lower-dimensional |
| Learned from context | no | yes |
| Context-sensitive meaning | no | no |

The last row is often missed: static embeddings **learn from context during training, but once trained they are frozen** - the vector for "bank" is the same in every sentence. Both representations are therefore static *with respect to the text being processed*.

## 16. Final Progression

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
