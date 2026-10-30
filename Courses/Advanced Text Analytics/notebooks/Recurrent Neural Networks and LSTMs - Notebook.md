---
type: course-note
date: 2026-10-28
tags:
  - course
  - advanced-text-analytics
  - notebook
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L08]]"
ai-first: true
---

## For Future Agent

Notebook companion for [[Recurrent Neural Networks and LSTMs]] (Lecture 8), extracted 2026-10-28: it collects only the passages documenting what the lecture's NumPy notebook actually runs (embedding lookup, the `rnn_step` dry run, label-score projection, the LSTM step and gate dry run, the bidirectional loop). The architecture theory stays in the parent note. Central caveat, repeated in the lecture: **every printed number comes from untrained random weights** - the values illustrate the mechanics, never model quality.

## 1. Embedding Lookup

Six-word biomedical vocabulary, 4-dimensional embeddings:

```python
vocab = [brca1, mutations, increase, breast, cancer, risk]
embedding_dim = 4     ->   W_embed has shape (6, 4)
```

```text
              d1     d2     d3     d4
brca1      0.152 -0.520  0.375  0.470
mutations -0.976 -0.651  0.064 -0.158
increase  -0.008 -0.427  0.440  0.389
breast     0.033  0.564  0.234 -0.430
cancer     0.184 -0.479  0.439 -0.025
risk      -0.092 -0.340  0.611 -0.077
```

Looking up the sentence produces `X` of shape `(6, 4)` - one row per position, one column per embedding dimension. Position now matters: $\mathbf{x}_1$ is processed before $\mathbf{x}_2$, and so on.

## 2. The RNN Step

With `hidden_dim = 5`, the recurrent weights are $W_x$ (5, 4), $W_h$ (5, 5), $b_h$ (5,). One step:

```python
def rnn_step(x_t, h_prev, W_x, W_h, b_h):
    return np.tanh(W_x @ x_t + W_h @ h_prev + b_h)
```

The first step starts from a zero hidden state, so $h_1$ is essentially a transformed `BRCA1`:

```text
Token: brca1
x1 = [ 0.152 -0.52   0.375  0.47 ]
h1 = [ 0.193  0.179  0.416  0.165 -0.123]
```

The second step already has context: $h_2$ is the representation of `mutations` **after reading `BRCA1 mutations`**, not a static embedding of the word type:

```text
Current token: mutations
Previous hidden state h1: [ 0.193  0.179  0.416  0.165 -0.123]
New hidden state h2     : [ 0.38  -0.141  0.354  0.082 -0.448]
```

Running the loop over all six tokens yields `H` of shape `(6, 5)`:

```text
              h1     h2     h3     h4     h5
brca1      0.193  0.179  0.416  0.165 -0.123
mutations  0.380 -0.141  0.354  0.082 -0.448
increase   0.222  0.320  0.504  0.193 -0.319
breast     0.046  0.439 -0.273 -0.564 -0.021
cancer     0.015  0.567 -0.203 -0.274  0.119
risk       0.168  0.552 -0.085 -0.415  0.033
```

How to read the table: rows are token positions; columns are **coordinates of the hidden vector**. The entry `0.015` in the `cancer` row is the first coordinate of the vector computed when the model processed `cancer`, **not** "hidden state at time 1". To remove the ambiguity, rename columns to `dim_1 ... dim_5`. Keep the two counts apart: 6 tokens -> 6 hidden vectors; 5 dims -> each hidden vector has 5 numerical coordinates. A dimension has **no predefined linguistic meaning** - coordinates are latent features learned during training, and in this untrained notebook `dim_5` for `cancer` has no interpretation at all.

## 3. Parameter Sharing Across Time

The same $W_x$, $W_h$, $b_h$ are reused at every position; only $x_t$, $h_{t-1}$, $h_t$ change. The notebook prints the first hidden coordinate at each step to make this visible:

```text
t=1  token=brca1      first hidden value= 0.193
t=2  token=mutations  first hidden value= 0.380
t=3  token=increase   first hidden value= 0.222
t=4  token=breast     first hidden value= 0.046
t=5  token=cancer     first hidden value= 0.015
t=6  token=risk       first hidden value= 0.168
```

Same computation, different state.

## 4. Label-Score Projection (Random Weights)

```python
labels = ["O", "B-GENE", "B-DISEASE", "I-DISEASE"]
W_label = rng.normal(0, 0.4, size=(5, 4))   # hidden dims -> labels
label_scores = H @ W_label                   # (6, 5) @ (5, 4) = (6, 4)
```

```text
H         = (6, 5)     6 tokens x 5 hidden dimensions
W_label   = (5, 4)     5 hidden dimensions -> 4 label scores
scores    = (6, 4)     6 tokens x 4 candidate labels
```

Each row is the evidence available at that position, given everything read so far:

```text
                O  B-GENE  B-DISEASE  I-DISEASE
brca1     -0.239   0.248      0.089     -0.113
mutations -0.236   0.034      0.290     -0.106
increase  -0.366   0.201      0.162     -0.083
breast    -0.332  -0.216     -0.359     -0.042
cancer    -0.343  -0.066     -0.305      0.010
risk      -0.462  -0.027     -0.334     -0.101
```

With trained weights, `BRCA1` would score high on `B-GENE` and `breast` high on `B-DISEASE`. With random weights the scores are noise - `argmax` therefore predicts `B-GENE`, `B-DISEASE`, `B-GENE`, `I-DISEASE`, `I-DISEASE`, `B-GENE`, which is meaningless as a model output but architecturally instructive: **the classifier acts on the contextual hidden state, not on the static embedding**. Note what the RNN does **not** do: it never emits `B-GENE` directly. It emits a contextual representation; a classifier on top turns that representation into task-specific scores.

## 5. Vanishing/Exploding Toy Curve

The notebook illustrates repeated multiplication with a toy curve rather than a training run:

```python
vanishing  = 0.8  ** steps    # factor below 1
exploding  = 1.15 ** steps    # factor above 1

1.0 -> 0.8 -> 0.64 -> 0.512 -> 0.4096 -> 0.3277 -> ...
```

- After about **10 steps** the signal is at **11%** of its original size; after **30 steps** it is under **0.13%**.
- Multiplied the other way, a factor of 1.15 grows the signal by a factor of **66** over 30 steps.

The graph is **intuition, not proof**: real RNN gradients involve matrices, chain rules and many interacting terms. The general point stands - repeated factors below 1 make signals shrink (vanishing gradient), repeated factors above 1 make them grow rapidly (exploding gradient).

## 6. One LSTM Step

The implementation concatenates previous hidden state and current input into one vector $z$ (shape 9 = 5 + 4), so every gate reads the same combined information. Four gate matrices of shape (5, 9) are involved:

```python
def lstm_step(x_t, h_prev, c_prev):
    z = np.concatenate([h_prev, x_t])

    forget_gate = sigmoid(W_forget    @ z + b_forget)
    input_gate  = sigmoid(W_input     @ z + b_input)
    candidate   = np.tanh (W_candidate @ z + b_candidate)

    c_t = forget_gate * c_prev + input_gate * candidate

    output_gate = sigmoid(W_output @ z + b_output)
    h_t = output_gate * np.tanh(c_t)
    return h_t, c_t, forget_gate, input_gate, output_gate
```

The line that carries the whole idea is the cell-state update: new memory = part of old memory we decided to keep + part of new information we decided to write. In a real deep learning library these four matrices live inside the LSTM layer; the notebook writes them out so the unit's internals stay visible.

### Dry run on the first token

```text
Token: brca1

Forget gate: [0.561 0.501 0.56  0.416 0.42 ]
Input gate : [0.444 0.462 0.503 0.501 0.523]
Output gate: [0.534 0.455 0.5   0.48  0.558]

Cell state c_t  : [ 0.211 -0.095  0.102 -0.19  -0.011]
Hidden state h_t: [ 0.111 -0.043  0.051 -0.09  -0.006]
```

Two observations:

1. The gates sit **near 0.5**, exactly what random weights with zero bias produce: the sigmoid of a small random number lands mid-range. Nothing has been learned yet.
2. The LSTM does **not** take one global binary decision. There is no single "remember this word / forget this word" switch: the gate is a **vector**, one value per dimension, so decisions are continuous and per-dimension.

## 7. Running the LSTM Over the Sentence

The loop returns **two** sequences of shape `(6, 5)`:

```text
Hidden-state shape: (6, 5)
Cell-state shape  : (6, 5)

              h1     h2     h3     h4     h5
brca1      0.111 -0.043  0.051 -0.090 -0.006
mutations  0.042 -0.059  0.215 -0.156  0.041
increase   0.136 -0.104  0.190 -0.102  0.007
breast     0.039 -0.055  0.098  0.012  0.015
cancer     0.093 -0.011  0.136 -0.064  0.021
risk       0.138 -0.036  0.193 -0.077  0.039
```

`H_lstm` is what you would feed to the next layer; `C_lstm` is the internal memory pathway. Magnitudes are smaller than the RNN's because $h_t$ is the product of a near-0.5 output gate and a small cell state - which is why the cell-state values are worth inspecting too. With random weights, no semantic interpretation is possible.

## 8. Bidirectional Run

```python
H_forward          = run_rnn(X, W_x, W_h, b_h)
H_backward_reversed = run_rnn(X[::-1], W_x, W_h, b_h)
H_backward          = H_backward_reversed[::-1]   # restore token order
H_bidirectional     = np.concatenate([H_forward, H_backward], axis=1)   # (6, 10)
```

Three details worth being explicit about:

1. The backward pass runs over the reversed sequence, so its rows come out in reverse token order; one more `[::-1]` restores the original order.
2. A real BiRNN uses **separate parameters** for the two directions. The notebook reuses $W_x$, $W_h$ for simplicity - a demonstration shortcut only.
3. The two rows for the same token carry **complementary** information:

```text
forward representation  -> influenced by BRCA1 mutations increase breast
backward representation -> influenced by breast cancer risk

h_t = [h_forward ; h_backward]      # 5 + 5 = 10 dimensions
```

One representation per token, informed by both sides.

## Related

- [[Recurrent Neural Networks and LSTMs]] - parent theory note (recurrence, gates, bidirectionality, pitfalls)
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L08]] (AY 2026/2027)
