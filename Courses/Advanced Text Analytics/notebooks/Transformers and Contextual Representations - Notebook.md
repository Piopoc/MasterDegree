---
type: course-note
date: 2026-10-28
tags:
  - course
  - advanced-text-analytics
  - notebook
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA09]]"
ai-first: true
---

## For Future Agent

Notebook companion for [[Transformers and Contextual Representations]] (Lecture 9), extracted 2026-10-28: it collects only the NumPy walkthrough steps of the lecture's notebook (dot-product similarity, the `np.triu` causal mask, the random positional encoding). The attention theory stays in the parent note. Central caveat: every printed number comes from **untrained random weights or hand-set toy vectors** - the values illustrate mechanics, never model quality.

## 1. Dot-Product Similarity as a First Attention Score

The notebook computes attention weights for the token `bank` against its context as raw dot products between hand-specified toy vectors:

$$\text{score}(x_i, x_j) = x_i \cdot x_j$$

A larger dot product means the vectors are more aligned. A `softmax` then turns the arbitrary scores into attention weights that are positive, normalized, and sum to 1. For `bank` compared against every context token, the largest score does **not** become a weight of 1: several tokens contribute to the new representation, each with a different share (the hand-set weight table - `pond` at 0.55, `bank` at 0.25, etc. - is in the parent note, Section 2).

## 2. Scaled Scores and the Attention Matrix

All tokens are rows of a matrix $X$, so every comparison is computed at once as $Q @ K^T$: a square (tokens x tokens) matrix, rows are queries, columns are keys. Each score is divided by $\sqrt{d_k}$ to keep dot-product magnitudes under control as dimensionality grows. Softmax is applied independently across every row, producing the attention matrix $A$ (each row sums to 1). The contextualized output is then $A {@} V$ - one weighted sum of value vectors per token position.

## 3. The Causal Mask with `np.triu`

For the left-to-right language-model setting, future positions must be unreachable. The notebook builds the causal mask with `np.triu` (upper-triangular, offset 1), adds it (with future entries set to $-\infty$) to the scaled scores, and applies row-wise softmax, which converts the masked positions to zero probability. For three positions the allowed pattern is:

```text
token 1 -> token 1
token 2 -> token 1, token 2
token 3 -> token 1, token 2, token 3
```

Result: **causal self-attention** - `pond` cannot attend to `near` or `bank`; `near` cannot attend to `bank`; `bank` can attend to all three positions.

## 4. Random Positional Encoding (Demo Only)

Self-attention alone does not know order: `dog bites man` and `man bites dog` produce the same set of pairwise comparisons. The notebook demonstrates the addition

```text
X_input = X_token + X_position
```

using **random** position vectors of the same dimensionality as the token embeddings, purely to show the elementwise addition operation. Real Transformers use learned or mathematically constructed (e.g. sinusoidal) position encodings; the random ones carry no distance structure and are a demonstration shortcut only.

## Related

- [[Transformers and Contextual Representations]] - parent theory note (Q/K/V, multi-head attention, blocks, model roles, decoding)
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA09]] (AY 2026/2027)
