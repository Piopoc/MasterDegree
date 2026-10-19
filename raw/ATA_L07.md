# Lecture 7: From features to representations

Source notebook: `ATA_L07_TFIDF_and_Static_Embeddings.ipynb` (Advanced Text Analytics).
The companion file `ATA_L07_Exercises_TFIDF_and_Static_Embeddings.ipynb` contains the
exercises that go with this lecture.

Machine learning models need numbers, so the lecture starts from a single question:
what do we turn a text into, and what does each conversion keep or throw away? The
answer is built as a ladder:

```text
Bag of Words → TF-IDF → cosine similarity → dense embeddings → tiny neural Skip-gram
```

The last step is implemented from scratch with NumPy. The lecture avoids `gensim`
so that nothing is hidden inside a library: the vocabulary, the training pairs, the
two weight matrices, and the gradient updates are all visible.

## Why representation matters

A text analytics system cannot feed words into a model directly. Something has to
produce a vector first. Different schemes preserve different information. Counts
survive in Bag of Words, weights survive in TF-IDF, and geometry survives in
embeddings. What all of them lose, and why, is the thread that runs through the
whole lecture.

## Bag of Words

Bag of Words represents a document by the counts of its terms. The vocabulary gives
one dimension per distinct term, and each document becomes a row of counts. The
lecture's first example uses two sentences:

```text
dog bites man    →  [bites: 1, dog: 1, man: 1]
man bites dog    →  [bites: 1, dog: 1, man: 1]
```

The two rows are identical. Whoever does the biting has changed, and Bag of Words
cannot say so. It preserves lexical counts and nothing else: order is gone, syntactic
roles are gone. This is a small example, but it states the limit precisely.

## The working corpus

For the rest of the sparse half of the lecture, the notebook uses six short documents
deliberately split between two domains:

```text
BIO-1   : BRCA1 mutation breast cancer
BIO-2   : TP53 mutation breast cancer
BIO-3   : myocardial infarction heart disease
LEGAL-1 : Regulation data protection personal data
LEGAL-2 : Article 6 Regulation data protection
LEGAL-3 : Court applies Regulation
```

The vocabulary has 16 terms. Two details are worth noticing. First, `data` appears
twice inside LEGAL-1, so its count is 2, not 1; counts genuinely matter. Second, the
"6" in "Article 6" never becomes a feature, because the default scikit-learn token
pattern keeps only tokens of at least two word characters.

## TF-IDF

TF-IDF reweights the counts. The idea is written in one line:

```text
TF-IDF(t,d) = TF(t,d) × IDF(t)
```

A term matters more when it is frequent in one document and rare across the
collection. In scikit-learn, with the default smoothing, the IDF for a corpus of
`n` documents and a term seen in `df` documents is:

```text
idf(t) = ln((1 + n) / (1 + df)) + 1
```

After weighting, each document vector is L2-normalized.

A dry worked example makes the numbers concrete. Take BIO-1 ("BRCA1 mutation breast
cancer") in a corpus of six documents:

```text
df(BRCA1) = 1   →  idf = ln(7/2) + 1 = 2.253
df(mutation) = df(breast) = df(cancer) = 2  →  idf = ln(7/3) + 1 = 1.847

raw scores : [2.253, 1.847, 1.847, 1.847]
L2 norm    : sqrt(2.253² + 3 × 1.847²) = 3.913

BRCA1    : 2.253 / 3.913 = 0.576
mutation : 1.847 / 3.913 = 0.472
breast   : 0.472
cancer   : 0.472
```

The gene name gets the largest weight because only one document contains it. The
three shared disease terms are discounted. The same pattern repeats in BIO-2, where
`TP53` takes 0.576 and the shared terms sit at 0.472. In the legal rows the situation
flips: `regulation` appears in three documents, so it ranks below `data` and
`protection`, and in LEGAL-2 the highest weight belongs to `article` (0.595), which
occurs only once in the corpus.

## Cosine similarity

Two documents are compared by the angle between their vectors:

```text
cos(x, y) = (x · y) / (||x|| × ||y||)
```

Cosine compares direction, not size. A long document and a short one with the same
term proportions have the same cosine against a third document, while their raw dot
products would differ a lot. This is why cosine is the default choice for sparse text
similarity.

On the lecture corpus the picture is clean. BIO-1 and BIO-2 score 0.669, because they
share `mutation`, `breast`, and `cancer`. Every biomedical document scores 0.000
against every legal document. Inside the legal side, LEGAL-1 and LEGAL-2 score 0.675,
while LEGAL-3 is weaker against both (0.138 and 0.181) because it shares only
one term, `regulation`, with each of them. Documents from different domains end up
orthogonal: BoW and TF-IDF cannot see any similarity between texts that use different
words.

## TF-IDF is still lexical

The lecture then tests the obvious weakness:

```text
S1: the patient suffered a heart attack
S2: the patient suffered a myocardial infarction
```

Any clinician would call these near-duplicates. Cosine similarity on their TF-IDF
vectors gives 0.432. The breakdown explains why. The only shared tokens are "the",
"patient", and "suffered"; "heart" and "attack" have no counterpart in S2, and
"myocardial" and "infarction" none in S1. ("a" disappears, again because of the
two-character token pattern.) So the score comes almost entirely from the sentence
frame, and the medical content contributes zero. TF-IDF measures surface overlap;
it has no way to know that "heart attack" and "myocardial infarction" can denote the
same concept.

## Distributional semantics

The fix suggested by the lecture is to let context define meaning. Words that occur
in similar contexts acquire similar representations:

```text
aspirin treats pain
ibuprofen treats pain
aspirin reduces inflammation
ibuprofen reduces inflammation
```

Nothing tells the model that aspirin and ibuprofen belong to the same class. The
shared frames do the work. This is the distributional hypothesis, and it is the
principle behind everything that follows.

## Static embeddings

Instead of one sparse dimension per vocabulary word, each word gets a dense vector,
for example:

```text
BRCA1 → [0.21, -0.14, 0.77, ...]
TP53  → [0.18, -0.09, 0.71, ...]
```

The lecture builds a toy space by hand to show what "dense" means geometrically. Six
words sit in a two-dimensional plane, and the two domains separate on the first
dimension:

```text
BRCA1 [0.90, 0.85]    TP53 [0.85, 0.65]    cancer [0.75, 0.65]
regulation [-0.75, 0.60]    court [-0.65, 0.50]    privacy [-0.55, 0.70]
```

Cosine similarity between hand-set vectors:

```text
regulation [-0.75, 0.60] vs court [-0.65, 0.50]:
   dot   = 0.4875 + 0.3000 = 0.7875
   norms : 0.960 × 0.820 = 0.788
   cos   = 0.9998

BRCA1 vs TP53       : 0.995
BRCA1 vs regulation : -0.139
```

Similarity becomes a geometric property. Two words are close when their vectors
point in similar directions, even if they never co-occur in a document. That is the
property TF-IDF cannot offer, and the reason "different words can be close" appears
in the final comparison table.

## Word2Vec and Skip-gram intuition

Skip-gram learns vectors by inverting a prediction task: given a center word, predict
the surrounding words.

```text
center word → predict surrounding words
```

The embedding is not given to the model. It is an internal parameter that gets shaped
as a side effect of solving that prediction task. The lecture uses a tiny corpus of
11 token lists mixing three registers: gene sentences ("brca1 mutation breast
cancer"), drug sentences ("aspirin treats pain"), and legal sentences ("court applies
regulation"). The vocabulary contains 22 word types.

With a window of 2, each center word collects its neighbors within two positions on
either side. For the sentence `brca1 mutation breast cancer`, the pairs begin with:

```text
brca1    → mutation
brca1    → breast
mutation → brca1
mutation → breast
mutation → cancer
breast   → brca1
...
```

Window boundaries cut the counts down: a four-token sentence yields 10 pairs and a
three-token sentence yields 6, so the whole corpus produces 90 pairs. Those 90 pairs
are the entire supervision signal. No entity labels, no semantic tags, only local
co-occurrence.

## The tiny neural model

The model is two matrices:

```text
W_in  : 22 × 12   (vocabulary → embedding)
W_out : 12 × 22   (embedding → vocabulary)
```

A row of `W_in` becomes the learned embedding for that word. Initialization draws
both matrices from a normal distribution with mean 0 and standard deviation 0.1,
with a fixed seed so the run is reproducible.

For one pair (center, context), the training step does the following:

```text
1. h         = W_in[center]                # retrieve the embedding
2. scores    = h · W_out                   # 22 raw scores
3. probs     = softmax(scores)             # distribution over the vocabulary
4. loss     += -log(probs[context])        # cross-entropy on the true word
5. dscores   = probs with 1 subtracted at the context word
6. grad W_out = outer(h, dscores)
7. grad h     = W_out · dscores
8. W_out     -= learning rate × grad W_out
9. W_in[center] -= learning rate × grad h
```

The softmax subtracts its maximum before exponentiating for numerical stability.
Only the row of `W_in` for the current center word is updated. This is a full-softmax
model, which is affordable only because the vocabulary has 22 entries; real Word2Vec
uses negative sampling for vocabularies of millions.

After 300 epochs at learning rate 0.05, the average loss falls from 3.092 to 1.471.
The curve drops quickly at the start and then flattens. A decreasing loss means the
model is getting better at predicting the observed context words of this specific
corpus, which is not the same as learning a general semantic space.

## How does the model learn strength of association?

One subtlety gets its own passage in the lecture. In Skip-gram, every observed pair is
a positive example with the same target label, so `(dog, barks)` and `(dog, runs)`
look identical from the loss's point of view in a single occurrence. The word "barks"
ends up closer to "dog" for two reasons. First, pair frequency feeds the gradient
directly: if `(dog, barks)` occurs 100 times and `(dog, runs)` 20 times, the first
pair pushes the vectors 100 times and the second only 20. Second, negative sampling
gives words negative updates when they are drawn as noise for other pairs. So the
strength of association is learned from the aggregate distribution of positive and
negative examples, never from assigning different target values to different positive
pairs.

## Nearest neighbors, and why to distrust them here

After training, the embedding of a word is simply its row of `W_in`. Nearest
neighbors are ranked by cosine similarity. On the lecture's run they come out as:

```text
brca1       : risk 0.566, tp53 0.488, breast 0.462, mutation 0.426, cancer 0.331
aspirin     : ibuprofen 0.998, applies 0.173, breast 0.157, pain 0.127
regulation  : personal 0.396, court 0.199, article 0.185, applies 0.172
```

Some of it looks right: ibuprofen is aspirin's twin because the corpus gives them
identical frames, and BRCA1's neighbors are biomedical words. The lecture adds a
warning that should not be skipped. This corpus is far too small, and the neighbors
are unstable. The exercise is about the learning mechanism, not about the quality of
the resulting space.

Direct similarity comparisons tell the same story:

```text
aspirin  vs ibuprofen  : 0.998   (identical contexts)
brca1    vs tp53       : 0.488   (shared, but not identical, contexts)
brca1    vs regulation : -0.261  (different domains)
```

The lecture's exercise notebook extends the corpus with four sentences that put
`brca1` and `tp53` into the same frames ("brca1 is a gene", "tp53 is a gene",
"brca1/tp53 mutation increases cancer risk"). Retraining with the same procedure
lifts their similarity from 0.488 to 0.708, and they become each other's nearest
neighbor. Repeated, similar contexts pull words toward the same region of the space,
which is exactly what the distributional hypothesis predicts. The comparison is
illustrative rather than controlled, since adding sentences also grows the vocabulary
and changes the initialization.

## Domain dependence

Embeddings reflect the corpus they were trained on. The lecture's example is the word
"appeal": in general English it can mean attractiveness, while in legal language it
refers to a procedure before a court. A general-domain model and a legal-domain model
would place "appeal" next to different words. There is no context-free "true" meaning
of the word in this framework, only the contexts a particular corpus supplies.

## FastText intuition

Static word vectors treat a word as an indivisible token. FastText builds
representations from character n-grams instead, which is useful when words share
subword structure:

```text
neuroblastoma
glioblastoma
lymphoblastoma
```

The shared suffix "-blastoma" lets information flow between these forms. Subword
modeling also helps rare words and words never seen in training, as long as their
character sequences have been seen somewhere.

## The one-vector-per-type limit

The final limitation of static embeddings is stated with a single word:

```text
river bank        →  bank = one vector
investment bank   →  bank = the same one vector
```

A static embedding assigns one vector per word type, so the senses of "bank" are
merged into a single point. Nothing in the representation changes when the word
appears in a different sentence. Contextual representations, which the course takes
up next, produce a vector per occurrence instead.

## Sparse against dense

The lecture summarizes the two families in a table:

| Property | TF-IDF | Static embeddings |
|---|---|---|
| Representation | sparse | dense |
| Typical unit | document/term features | word |
| Dimensions | vocabulary-sized | lower-dimensional |
| Learned from context | no | yes |
| Context-sensitive meaning | no | no |

The last row is worth a second look, because it is often missed. Static embeddings
learn from context during training, but once trained they are frozen: the vector for
"bank" is the same in every sentence. Both representations are therefore static with
respect to the text being processed.

## Final progression

The lecture closes the loop it opened:

```text
Bag of Words
    ↓
TF-IDF
    ↓
Static embeddings
    ↓
Contextual embeddings
```

Take-home messages from the lecture:

1. Bag of Words and TF-IDF are explicit lexical representations.
2. Cosine similarity compares vector geometry.
3. Static embeddings are dense learned word representations.
4. A simple neural context-prediction task can learn an embedding matrix.
5. Embeddings depend on their training corpus.
6. Static embeddings still assign one vector to each word type.
