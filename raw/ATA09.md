# ATA09 — Transformers and Contextual Representations

**Course:** Advanced Text Analytics (ATA)  
**Lecture:** L09 / ATA09  
**Source:** `L09/ATA_L09_Transformers_Contextual_Representations_companion.ipynb`  
**Topic:** Transformer architecture, attention, and contextual token representations  
**Length of reference material:** ~45-minute slide deck + companion notebook (numpy/pandas/matplotlib only, no full language model implementation)

---

## What this lecture is about

The central claim of ATA09 is simple:

> A Transformer builds the representation of a token by selectively integrating information from other token positions.

The motivating example is the word **bank**. A static embedding assigns one vector to the word type `bank`, no matter the context:

```
river bank
investment bank
```

These are two different meanings, but a static embedding cannot tell them apart. RNNs and LSTMs (Lecture 8) solve this by propagating context sequentially through a hidden state. Transformers solve it differently: **attention lets a token directly gather information from other relevant positions in the sequence**, without recurrence.

The notebook walks through the mechanism step by step, using tiny, hand-specified vectors so every computation stays inspectable:

```
static token embeddings
        ↓
attention as a weighted sum
        ↓
attention scores + softmax
        ↓
query / key / value
        ↓
self-attention
        ↓
multi-head attention
        ↓
position information
        ↓
causal masking
        ↓
contextual token representations
```

---

## 1. Why contextual representations?

A static embedding is *type-level*: one vector per word type. A Transformer representation is *token-level and contextual*: the vector for `bank` in "I walked along the pond and saw the bank" should be influenced by `pond`, differently from how it would be influenced by `investment`.

The notebook uses the sentence:

```
I walked along the pond and saw the bank
```

The point is not linguistic realism; it is to observe how the representation of `bank` can be pulled toward the context word `pond`.

---

## 2. Attention as a weighted sum

Before query/key/value, the notebook introduces the simplest possible attention mechanism: a weighted combination of context vectors.

Suppose we already know the importance weights for the context of `bank`:

| token | weight |
|-------|--------|
| I | 0.02 |
| walked | 0.03 |
| pond | 0.55 |
| trees | 0.15 |
| bank | 0.25 |

The weights sum to 1. The contextual representation is:

\[
a_i = \sum_j \alpha_{ij} x_j
\]

where \(x_j\) is a token vector and \(\alpha_{ij}\) is the weight assigned to token \(j\) when representing token \(i\). The weights indicate **how much each context vector contributes**.

The notebook builds 3-dimensional toy vectors for each token, stacks them into a matrix, multiplies each row by its attention weight, and sums the weighted rows. The result is one new vector: the contextual representation of `bank`.

**The mechanism, not the number:**

```
context vectors
      ×
importance weights
      ↓
weighted sum
      ↓
new contextual vector
```

Attention = selective information integration.

---

## 3. Where do the attention weights come from?

In a real model, weights are not hand-specified. They are computed from a similarity score between the current token and each possible context token. The notebook uses the dot product as a first approximation:

\[
\text{score}(x_i, x_j) = x_i \cdot x_j
\]

A larger dot product means the vectors are more aligned. A `softmax` then turns the arbitrary scores into attention weights that are positive, normalized, and sum to 1.

For `bank` compared against every context token, the dot products are converted by softmax into a distribution of relevance. The largest score does **not** become a weight of 1: several tokens can contribute to the new representation, each with a different share.

---

## 4. Why Query, Key, and Value?

The simplified version above uses the same vector both to compare and to combine. Transformers make this more flexible by projecting each input representation into three different roles:

\[
q_i = x_i W_Q
\]
\[
k_i = x_i W_K
\]
\[
v_i = x_i W_V
\]

Conceptually:

| Role | Question it answers |
|------|---------------------|
| Query | what am I looking for? |
| Key | how should I be matched? |
| Value | what information should I contribute? |

**Key idea:** matching information and transferred information do not need to use the same representation.

The notebook demonstrates this on the short sequence `pond → near → bank`, with 4-dimensional input vectors projected into \(d_k = 3\) query/key dimensions and \(d_v = 3\) value dimensions using randomly initialized matrices \(W_Q, W_K, W_V\). Every token ends up with its own query, key, and value — three different learned views of the same input.

---

## 5. Scaled dot-product attention

To decide how much token \(i\) should attend to token \(j\), Transformers compare the **query of \(i\)** with the **key of \(j\)**:

\[
\text{score}(i, j) = \frac{q_i \cdot k_j}{\sqrt{d_k}}
\]

The division by \(\sqrt{d_k}\) keeps dot-product magnitudes under control as dimensionality grows (large dot products push softmax into regions with tiny gradients).

Because all tokens are stored as rows of a matrix \(X\), every comparison is computed at once with:

\[
Q @ K^T
\]

The result is a square matrix of size (tokens × tokens): rows are queries, columns are keys. Softmax is applied independently across every row, producing the **attention matrix** \(A\). Each row sums to 1, and entry \(A[i,j]\) means:

> how much token position \(i\) uses the information from token position \(j\).

---

## 6. Using the weights to combine the Value vectors

The attention matrix tells us *where to get information from*. The \(V\) matrix contains the information that will actually be combined:

\[
\text{contextualized output} = A @ V
\]

For each token position, this computes a weighted sum of the value vectors. The output row for `bank` is no longer just a transformed version of the original `bank` vector: it contains a weighted combination of information from `pond`, `near`, and `bank`, according to the learned attention distribution.

This is the central mechanism that creates a **contextual representation**.

---

## 7. Self-attention transforms the whole sequence in parallel

Self-attention contrasts with the RNN from Lecture 8. An RNN conceptually computes:

```
h₁ → h₂ → h₃ → ...
```

because each step depends on the previous hidden state. Self-attention instead computes all query-key comparisons from the sequence matrix at once:

```
X
↓
Q, K, V
↓
all token-to-token comparisons
↓
one contextual output vector per token
```

The output has the same number of token positions as the input, and the dimensionality can be projected back to the model dimension, which makes it possible to stack Transformer blocks repeatedly.

---

## 8. Multi-head attention

A Transformer normally uses several attention **heads** in parallel, not just one. Each head has its own \(W_Q, W_K, W_V\), so each head is free to learn a different pattern of relevance. The outputs of all heads are then concatenated (and, in a real model, passed through a learned output projection).

The notebook demonstrates two independent heads on the same input sequence. Because their parameters differ, the two heads assign different attention weights to the same sequence.

**Caution:** it is tempting to assign a clean linguistic interpretation to every head (e.g. "this head tracks subject-verb agreement"). Some heads do learn recognizable patterns, but the model is not required to organize each head into one clean human-readable function. The useful intuition is simply:

> multiple heads provide multiple learned ways to gather contextual information.

With two heads of dimension 2, concatenation gives a vector of size 4 again — matching the model dimension.

---

## 9. Transformers need explicit position information

An RNN naturally knows order because it processes the sequence one position at a time. Self-attention performs comparisons between vectors and, by itself, does not inherently know the difference between:

```
dog bites man
man bites dog
```

Therefore the Transformer input representation combines:

```
token embedding + position representation
```

The positional vector has the **same dimensionality** as the token embedding, because the two are added:

\[
X_{\text{input}} = X_{\text{token}} + X_{\text{position}}
\]

The resulting vector represents both *what token is here?* and *where is it in the sequence?*. This positional information can influence all subsequent attention calculations. (Real Transformers use learned or mathematically constructed position encodings; the notebook uses random ones only to show the operation.)

---

## 10. Causal attention and masking

The Jurafsky & Martin chapter underlying this lecture first presents the Transformer as a **left-to-right language model**. In that setting, token position \(i\) may use itself and earlier positions, but not future tokens — otherwise next-token prediction would be trivial.

This is implemented with a **causal mask**. For three positions, the allowed attention pattern is:

```
token 1 → token 1
token 2 → token 1, token 2
token 3 → token 1, token 2, token 3
```

Future positions receive \(-∞\) before softmax, which converts them to zero probability. The notebook builds this mask with `np.triu`, adds it to the scaled scores, and applies row-wise softmax.

Result: **causal self-attention** — `pond` cannot attend to `near` or `bank`; `near` cannot attend to `bank`; `bank` can attend to all three positions.

---

## 11. Bidirectional attention for text analytics

For many information-extraction tasks the full sentence is already available. In NER, for example:

```
BRCA1 mutations increase breast cancer risk
```

the representation of `breast` can benefit from seeing the following word `cancer`. An encoder-style Transformer can therefore allow a token to use both left and right context.

| | Causal Transformer | Bidirectional encoder |
|---|---|---|
| Visibility | future positions hidden | all input positions visible |
| Typical use | left-to-right language modeling | encoding for downstream tasks |

The core attention computation is identical; only the visibility constraint changes. This is analogous in purpose to the bidirectional RNN/LSTM from Lecture 8, but with attention instead of recurrence.

---

## 12. What is a Transformer block?

Self-attention is the central token-mixing component, but not the whole block. A simplified block contains:

```
LayerNorm
↓
Multi-head self-attention
↓
Residual connection
↓
LayerNorm
↓
Feed-forward network
↓
Residual connection
```

Component roles:

- **Multi-head self-attention:** mixes information across positions (token-mixing). Token \(i\) receives information from token \(j\).
- **Feed-forward network:** transforms each token representation separately with a non-linear transformation. It does not mix positions.
- **Residual connections:** add the component input back to its output, so the original representation keeps flowing while layers add new information.
- **Layer normalization:** rescales each token representation into a numerically stable range; mainly important for training deep Transformers reliably.

---

## 13. Why stack many Transformer blocks?

One self-attention layer already produces contextualized representations, but Transformers stack many blocks. Conceptually:

```
initial token + position representation
           ↓
       block 1
           ↓
       block 2
           ↓
       block 3
           ↓
          …
           ↓
richer contextual representation
```

Each layer receives the representations created by the previous layer, so contextualization is **built layer by layer**. Earlier layers may capture relatively local or surface relationships; later layers can integrate more complex information. We should not assume every layer has one clean linguistic interpretation, but the key idea holds:

> contextualization is built layer by layer.

---

## 14. Contextual representations for Information Extraction

For this course, this is the destination. Given:

```
BRCA1 mutations increase breast cancer risk
```

the Transformer produces one contextual vector per token position. These vectors can then be passed to a task-specific classifier. A classifier trained on annotated data might learn:

| token | illustrative gold label |
|-------|-------------------------|
| BRCA1 | B-GENE |
| mutations | O |
| increase | O |
| breast | B-DISEASE |
| cancer | I-DISEASE |
| risk | O |

**Pipeline:**

```
tokens
   ↓
token + position embeddings
   ↓
Transformer blocks
   ↓
contextual token vectors
   ↓
task-specific classifier
   ↓
NER labels
```

This is the direct bridge to Lecture 10: **fine-tuning and domain adaptation**.

---

## 15. RNN/LSTM versus Transformer

| RNN / LSTM | Transformer |
|---|---|
| context propagated sequentially | context integrated with attention |
| hidden state moves through positions | token positions interact through attention |
| order naturally encoded by recurrence | position must be represented explicitly |
| computation sequential across positions | attention computations parallelizable |
| contextual representation at each position | contextual representation at each position |

The goal is similar — produce representations that depend on context — but the mechanism is different.

---

## Take-home messages

1. Static embeddings are type-level; Transformer representations are contextual.
2. Attention is fundamentally a weighted combination of information from other token positions.
3. Attention weights come from learned relevance comparisons.
4. Query and key are used for matching; value carries the information to be combined.
5. Multi-head attention provides several learned contextualization mechanisms in parallel.
6. Transformers need explicit positional information because they do not use recurrence.
7. Causal attention hides future positions; bidirectional attention can use the whole input.
8. A Transformer block combines attention, feed-forward processing, residual connections, and normalization.
9. Stacking blocks progressively builds richer contextual representations.
10. For text analytics, contextual token vectors become inputs to task-specific components such as NER classifiers.

---

## Key formulas (quick reference)

- Weighted sum: \(\displaystyle a_i = \sum_j \alpha_{ij} x_j\)
- Dot-product score: \(\displaystyle \text{score}(x_i, x_j) = x_i \cdot x_j\)
- Q/K/V projections: \(\displaystyle q_i = x_i W_Q,\; k_i = x_i W_K,\; v_i = x_i W_V\)
- Scaled dot-product attention: \(\displaystyle \text{score}(i,j) = \frac{q_i \cdot k_j}{\sqrt{d_k}}\)
- Attention matrix: \(\displaystyle A = \text{row\_softmax}\left(\frac{Q K^T}{\sqrt{d_k}}\right)\)
- Contextualized output: \(\displaystyle A @ V\)
- Transformer input: \(\displaystyle X_{\text{input}} = X_{\text{token}} + X_{\text{position}}\)

---

## Suggested review questions

1. Why can't a static embedding distinguish "river bank" from "investment bank"?
2. What does each row of the attention matrix \(A\) represent, and why does it sum to 1?
3. What is the difference between query, key, and value in attention?
4. Why divide by \(\sqrt{d_k}\) in scaled dot-product attention?
5. How does self-attention differ from an RNN in terms of computation and parallelism?
6. Why do Transformers require explicit position encodings?
7. What does a causal mask do, and when is bidirectional attention preferable?
8. What are the roles of residual connections and layer normalization in a Transformer block?
9. How do contextual token vectors connect to a downstream NER classifier?
