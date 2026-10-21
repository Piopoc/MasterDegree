# Lecture 8: RNNs and LSTMs for text analytics

Companion notes for the lesson in `L08/`. They describe what the slide deck
(`ATA_L08_RNN_LSTM.pdf`, 16 slides) puts on screen and what the notebook
(`ATA_L08_RNN_LSTM_text_focused_companion.ipynb`, 83 cells) actually computes.

The goal of the lecture is deliberately narrow. By the end of it you should be
able to explain three things: what an RNN adds on top of the static embeddings
from Lecture 7, why simple RNNs struggle with long-distance information, and
what an LSTM changes about that.

Out of scope for this lecture: CRFs, Viterbi decoding, Transformers, and a
detailed treatment of backpropagation. Those come later.

The conceptual path of the whole lecture:

```text
static embeddings
      ↓
ordered sequence of embeddings
      ↓
RNN
      ↓
context-dependent hidden states
      ↓
long-distance problem
      ↓
LSTM
      ↓
explicit memory + gates
      ↓
bidirectional sequence modelling
      ↓
contextual representations for NER
```

---

## 1. Setup and dependencies

The notebook uses three libraries and nothing else:

| Library | Role in the lesson |
|---|---|
| `numpy` | vector and matrix arithmetic |
| `pandas` | printing vectors, hidden-state tables, and score tables |
| `matplotlib` | two figures (repeated multiplication, sigmoid) |

The seed is fixed so the printed numbers are reproducible:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

rng = np.random.default_rng(42)
```

Every model in the notebook is **untrained and random**. The values you see are
there to make the machinery visible, not to demonstrate any quality. Section 11
lists the places where this distinction matters.

---

## 2. Why sequence models? (slides 2 and 3)

A static embedding assigns one vector to a word type:

```text
BRCA1    → vector
mutation → vector
cancer   → vector
```

The vector for `cancer` is the same everywhere it appears. It cannot tell you
whether it was used in `breast cancer`, `cancer treatment`, or `cancer risk`.

Text is not an unordered bag of words, so the role of a token depends on:

- which tokens came before it;
- their order;
- sometimes on which tokens come after it.

An RNN starts from the same embeddings as Lecture 7 but consumes them as an
ordered sequence.

### 2.1 A tiny vocabulary and an embedding table

```python
vocab = ["brca1", "mutations", "increase", "breast", "cancer", "risk"]
word_to_id = {word: i for i, word in enumerate(vocab)}

embedding_dim = 4
W_embed = rng.normal(loc=0.0, scale=0.5, size=(len(vocab), embedding_dim))
```

`W_embed` has one row per word type:

```text
word ID → row of W_embed → embedding vector
```

Printed table (seed 42, rounded to 3 decimals):

```text
              d1     d2     d3     d4
brca1      0.152 -0.520  0.375  0.470
mutations -0.976 -0.651  0.064 -0.158
increase  -0.008 -0.427  0.440  0.389
breast     0.033  0.564  0.234 -0.430
cancer     0.184 -0.479  0.439 -0.025
risk      -0.092 -0.340  0.611 -0.077
```

The row for `cancer` is fixed for the whole lesson:

```python
W_embed[word_to_id["cancer"]]   # [ 0.184 -0.479  0.439 -0.025]
```

### 2.2 From a sentence to a sequence of vectors

```python
sentence = ["brca1", "mutations", "increase", "breast", "cancer", "risk"]
ids = [word_to_id[word] for word in sentence]
X = W_embed[ids]
```

`X` has shape `(6, 4)`: one row per position, one column per embedding
dimension.

```text
x₁ = embedding("brca1")
x₂ = embedding("mutations")
x₃ = embedding("increase")
x₄ = embedding("breast")
x₅ = embedding("cancer")
x₆ = embedding("risk")
```

Position matters now. `x₁` is processed before `x₂`, and so on.

---

## 3. What an RNN adds (slides 4 and 5)

The central mechanism is the **hidden state**. At time `t` the model combines
the current embedding with the previous hidden state and produces a new hidden
state:

\[
h_t = \tanh(W_x x_t + W_h h_{t-1} + b)
\]

Read it as a one-liner:

> current token + previous context → new context

Two things happen at once. `W_x @ x_t` carries information from the current
word, `W_h @ h_prev` carries information from the preceding context, and the
sum is squashed by `tanh` into the range (-1, 1).

### 3.1 The recurrent weights

```python
hidden_dim = 5

W_x = rng.normal(0, 0.4, size=(hidden_dim, embedding_dim))   # (5, 4)
W_h = rng.normal(0, 0.4, size=(hidden_dim, hidden_dim))      # (5, 5)
b_h = np.zeros(hidden_dim)                                    # (5,)
```

| Matrix | Shape | Job |
|---|---|---|
| `W_x` | (5, 4) | embedding space → hidden-state space |
| `W_h` | (5, 5) | previous hidden state → new hidden state |
| `b_h` | (5,) | offset added to every hidden unit |

No new matrices are created for later tokens. One model is applied repeatedly
across the sequence, which is exactly what lets an RNN handle sequences of
different lengths.

### 3.2 One recurrent step

```python
def rnn_step(x_t, h_prev, W_x, W_h, b_h):
    input_part = W_x @ x_t
    context_part = W_h @ h_prev
    h_t = np.tanh(input_part + context_part + b_h)
    return h_t
```

### 3.3 First token: no previous context

The first step starts from a zero hidden state:

```python
h0 = np.zeros(hidden_dim)     # [0, 0, 0, 0, 0]
x1 = X[0]
h1 = rnn_step(x1, h0, W_x, W_h, b_h)
```

Dry run with the shipped seed:

```text
Token: brca1
x1 = [ 0.152 -0.52   0.375  0.47 ]
h1 = [ 0.193  0.179  0.416  0.165 -0.123]
```

Since `h0` is empty, `h1` is essentially a transformed version of `BRCA1`.

### 3.4 Second token: context starts to matter

```python
x2 = X[1]
h2 = rnn_step(x2, h1, W_x, W_h, b_h)
```

```text
Current token: mutations
Previous hidden state h1: [ 0.193  0.179  0.416  0.165 -0.123]
New hidden state h2     : [ 0.38  -0.141  0.354  0.082 -0.448]
```

`h2` is no longer a static embedding of `mutations`. It is a representation of
that position after the model has processed `BRCA1 mutations`.

```text
BRCA1
  ↓
 h₁
  ↓
mutations
  ↓
 h₂
```

### 3.5 Forward pass over the whole sentence

```python
def run_rnn(X, W_x, W_h, b_h):
    h_prev = np.zeros(W_h.shape[0])
    states = []
    for x_t in X:
        h_t = rnn_step(x_t, h_prev, W_x, W_h, b_h)
        states.append(h_t.copy())
        h_prev = h_t
    return np.array(states)

H = run_rnn(X, W_x, W_h, b_h)     # shape (6, 5)
```

`H` holds one hidden state per token, so the shape is
`(number of tokens, hidden dimension)`. Printed table:

```text
              h1     h2     h3     h4     h5
brca1      0.193  0.179  0.416  0.165 -0.123
mutations  0.380 -0.141  0.354  0.082 -0.448
increase   0.222  0.320  0.504  0.193 -0.319
breast     0.046  0.439 -0.273 -0.564 -0.021
cancer     0.015  0.567 -0.203 -0.274  0.119
risk       0.168  0.552 -0.085 -0.415  0.033
```

The conceptual difference from Lecture 7:

```text
Static embedding:
cancer → same vector whenever the word type appears

RNN hidden state:
cancer at position t → vector influenced by the sequence processed before t
```

### 3.6 How to read the hidden-state table (important)

The rows are token positions and the columns are coordinates of the hidden
vector. This trips up most students on the first pass.

The entry in column `h1` of the `cancer` row does **not** mean "hidden state at
time 1". It means the first coordinate of the hidden vector computed when the
model processed `cancer`. That vector was produced by

\[
h_t = \tanh(W_x x_t + W_h h_{t-1} + b_h)
\]

so it depends on `x_t = embedding("cancer")` and on `h_(t-1)`, the state after
`breast`:

```text
BRCA1 → mutations → increase → breast → cancer
                                      ↓
                             previous hidden state
                                      +
                             embedding(cancer)
                                      ↓
                              new hidden state
```

To remove the ambiguity, rename the columns to `dim_1 ... dim_5`:

```python
columns=[f"dim_{i+1}" for i in range(hidden_dim)]
```

```text
              dim_1   dim_2   dim_3   dim_4   dim_5
brca1
mutations
increase
breast
cancer
risk
```

Remember the two counts:

```text
6 tokens → 6 hidden vectors
5 dims  → each hidden vector has 5 numerical coordinates
```

A dimension has no predefined linguistic meaning. Nobody decided in advance
that `dim_1` is gene information and `dim_2` is plurality. The coordinates are
latent features learned during training, and in a trained model the meaning is
usually spread across several dimensions rather than sitting in one. In this
notebook the weights are random, so `dim_5` for `cancer` has no interpretation
at all.

What to take away:

> The hidden dimensions are coordinates of a learned representation, not named
> linguistic features. What matters is the hidden vector as a whole.

---

## 4. What is shared across time (slide 6)

An unrolled RNN looks like a different network at every position. It is not.
The same `W_x`, `W_h`, `b_h` are used at `t = 1, 2, ..., n`. Only the values
`x_t`, `h_(t-1)`, and `h_t` change.

```python
h = np.zeros(hidden_dim)
for t, (token, x_t) in enumerate(zip(sentence, X), start=1):
    h = rnn_step(x_t, h, W_x, W_h, b_h)
    print(f"t={t:<2} token={token:<10} first hidden value={h[0]: .3f}")
```

```text
t=1  token=brca1      first hidden value= 0.193
t=2  token=mutations  first hidden value= 0.380
t=3  token=increase   first hidden value= 0.222
t=4  token=breast     first hidden value= 0.046
t=5  token=cancer     first hidden value= 0.015
t=6  token=risk       first hidden value= 0.168
```

Same computation, different state. That parameter sharing is what lets the
model accept sequences of variable length.

---

## 5. What an RNN can be used for (slides 7 and 8)

The same recurrent machinery supports several task families.

| Task | How the hidden state is used |
|---|---|
| Language modelling | `h_t` predicts the next token |
| Sequence labelling | `h_t` predicts the label of token `t` |
| Sequence classification | final or pooled states predict one label for the whole text |

Sequence labelling is the relevant one for this course, since it leads directly
to NER.

### 5.1 From hidden states to label scores

Take a small label set:

```python
labels = ["O", "B-GENE", "B-DISEASE", "I-DISEASE"]
W_label = rng.normal(0, 0.4, size=(hidden_dim, len(labels)))   # (5, 4)
label_scores = H @ W_label                                    # (6, 4)
```

The matrix multiplication does the following:

```text
H         = (6, 5)     6 tokens × 5 hidden dimensions
W_label   = (5, 4)     5 hidden dimensions → 4 label scores
-------------------
scores    = (6, 4)     6 tokens × 4 candidate labels
```

Printed scores with the shipped seed:

```text
                O  B-GENE  B-DISEASE  I-DISEASE
brca1     -0.239   0.248      0.089     -0.113
mutations -0.236   0.034      0.290     -0.106
increase  -0.366   0.201      0.162     -0.083
breast    -0.332  -0.216     -0.359     -0.042
cancer    -0.343  -0.066     -0.305      0.010
risk      -0.462  -0.027     -0.334     -0.101
```

Each row is the evidence available at that position, given everything the model
has read so far. With trained weights, the row for `BRCA1` would carry a high
`B-GENE` score and the row for `breast` a high `B-DISEASE` score. With random
weights the scores are noise.

The full pipeline:

```text
token
  ↓
embedding
  ↓
RNN
  ↓
5-dimensional contextual vector
  ↓
W_label
  ↓
4 label scores
```

Note what the RNN does **not** do: it never emits `B-GENE` directly. It emits a
contextual representation, and a classifier on top turns that representation
into task-specific scores.

### 5.2 Local label predictions

```python
predicted_ids = np.argmax(label_scores, axis=1)
predictions = [labels[i] for i in predicted_ids]
```

```text
brca1        B-GENE
mutations    B-DISEASE
increase     B-GENE
breast       I-DISEASE
cancer       I-DISEASE
risk         B-GENE
```

These predictions are meaningless because the model is untrained. The point is
the architecture: the classifier acts on the contextual hidden state, not on
the static embedding.

```text
tokens
  ↓
embeddings
  ↓
RNN
  ↓
contextual hidden states
  ↓
token-level classifier
```

---

## 6. The problem: long-distance dependencies (slides 9 and 10)

A simple RNN can in principle carry information from the start of a sequence
to the end. In practice it struggles. The classic example:

> The flights the airline was canceling were full.

To predict `were` correctly the model needs to know that the grammatical
subject is the plural `flights`, but `airline` appears in between and provides
a competing singular cue. The useful signal has to survive several recurrent
updates.

### 6.1 Repeated multiplication as an intuition for vanishing gradients

The notebook illustrates this with a toy curve rather than a training run:

```python
steps = np.arange(0, 31)
vanishing = 0.8 ** steps
exploding = 1.15 ** steps
```

```text
1.0 → 0.8 → 0.64 → 0.512 → 0.4096 → 0.3277 → ...
```

After about 10 steps the signal is at 11% of its original size, and after 30
steps it is under 0.13%. Multiplied the other way, a factor of 1.15 grows the
signal by a factor of 66 over 30 steps.

The graph is intuition, not proof. Real RNN gradients involve matrices, chain
rules, and many interacting terms. The general point stands:

- repeated factors below 1 make signals shrink (vanishing gradient);
- repeated factors above 1 make signals grow rapidly (exploding gradient).

### 6.2 The representational problem

Gradients are not the whole story. The hidden state is asked to do two jobs at
once:

1. represent information useful right now;
2. preserve information that may only become useful much later.

While processing `The flights the airline was canceling ...`, the state has to
keep encoding `flights` while also handling the current tokens. This is the
problem an explicit memory mechanism is designed to fix.

---

## 7. LSTM: the big idea (slides 11 to 13)

An LSTM is still a recurrent model. It still consumes `x₁, x₂, x₃, ...` one
position at a time. What it adds is a second recurrent vector, the cell state
(or memory state) `c_t`.

```text
h_t → visible hidden representation
c_t → longer-term memory pathway
```

### 7.1 Gates as soft switches

Gates are produced by the sigmoid function, so they live in (0, 1):

```text
0   → mostly block information
0.5 → partially pass it
1   → mostly let information pass
```

The notebook plots the curve for inputs from -6 to 6, where the sigmoid
saturates at 0 and 1.

The sigmoid is useful because its output acts as a soft mask over a memory
vector. Given a memory like

```text
[important, irrelevant, important, irrelevant]
```

a learned gate might behave like

```text
[0.95, 0.05, 0.90, 0.10]
```

keeping some coordinates and strongly reducing others. Because gate values are
learned per dimension, the model can preserve some information while
suppressing other information.

### 7.2 The three decisions

At every position the LSTM makes three conceptually separate decisions.

| Gate | Question it answers |
|---|---|
| Forget gate | How much of the previous memory `c_(t-1)` should survive? |
| Input gate | How much new information should be written into memory? |
| Output gate | How much of the updated memory should become visible in `h_t`? |

### 7.3 One LSTM step, without memorising the equations

The implementation concatenates the previous hidden state and the current
input into a single vector `z`, so every gate reads the same combined
information:

```python
lstm_hidden_dim = 5
combined_dim = lstm_hidden_dim + embedding_dim      # 5 + 4 = 9

W_forget    = rng.normal(0, 0.3, size=(5, 9))
W_input     = rng.normal(0, 0.3, size=(5, 9))
W_candidate = rng.normal(0, 0.3, size=(5, 9))
W_output    = rng.normal(0, 0.3, size=(5, 9))
b_forget = b_input = b_candidate = b_output = np.zeros(5)
```

In a real deep learning library these four matrices live inside the LSTM
layer. They are written out here so the unit's internals are visible.

The step itself:

```python
def lstm_step(x_t, h_prev, c_prev):
    z = np.concatenate([h_prev, x_t])

    forget_gate = sigmoid(W_forget @ z + b_forget)
    input_gate  = sigmoid(W_input @ z + b_input)
    candidate   = np.tanh(W_candidate @ z + b_candidate)

    c_t = forget_gate * c_prev + input_gate * candidate

    output_gate = sigmoid(W_output @ z + b_output)
    h_t = output_gate * np.tanh(c_t)

    return h_t, c_t, forget_gate, input_gate, output_gate
```

The line that carries the whole idea is the cell-state update:

```text
new memory
  = part of old memory we decided to keep
  + part of new information we decided to write
```

### 7.4 Dry run on the first token

```python
h_prev = np.zeros(lstm_hidden_dim)
c_prev = np.zeros(lstm_hidden_dim)
x_t = X[0]
h_t, c_t, forget_gate, input_gate, output_gate = lstm_step(x_t, h_prev, c_prev)
```

```text
Token: brca1

Forget gate: [0.561 0.501 0.56  0.416 0.42 ]
Input gate : [0.444 0.462 0.503 0.501 0.523]
Output gate: [0.534 0.455 0.5   0.48  0.558]

Cell state c_t  : [ 0.211 -0.095  0.102 -0.19  -0.011]
Hidden state h_t: [ 0.111 -0.043  0.051 -0.09  -0.006]
```

Two observations from this output.

First, the gates are all near 0.5 here, which is what you expect from random
weights with zero bias: the sigmoid of a small random number sits near the
middle of its range. Nothing has been learned yet.

Second, the LSTM does not take one global binary decision. There is no single
"remember this word / forget this word" switch. The gate is a vector, one
value per dimension, so the unit makes continuous, per-dimension decisions.

### 7.5 Cell state versus hidden state

| Vector | Role |
|---|---|
| `c_t` | the longer-term memory carried forward |
| `h_t` | the current exposed representation used by the rest of the network |

A useful way to phrase it:

```text
c_t = what the LSTM is remembering internally
h_t = what the LSTM is showing externally at this step
```

The cell state is updated by keeping some old memory and adding new
information, and the output gate decides which part of that memory becomes
visible:

```text
previous memory c_(t-1)
        ↓
  forget / keep
        +
  add new information
        ↓
       c_t
  internal memory
        ↓
  output gate
        ↓
       h_t
  visible representation
```

For text analytics, `h_t` is the representation you pass to the next layer,
for example a NER classifier. The cell state is mostly an internal mechanism
for carrying information along the sequence. In Jurafsky and Martin's
terminology, the context layer is maintained for future decisions while the
hidden state provides the output of the LSTM at each time step.

---

## 8. Running the LSTM over the sentence

```python
def run_lstm(X):
    h = np.zeros(lstm_hidden_dim)
    c = np.zeros(lstm_hidden_dim)
    hidden_states, cell_states = [], []

    for x_t in X:
        h, c, *_ = lstm_step(x_t, h, c)
        hidden_states.append(h.copy())
        cell_states.append(c.copy())

    return np.array(hidden_states), np.array(cell_states)

H_lstm, C_lstm = run_lstm(X)
```

```text
Hidden-state shape: (6, 5)
Cell-state shape  : (6, 5)
```

Now two sequences of vectors come out. `H_lstm` is what you would feed to the
next layer, `C_lstm` is the internal memory pathway.

```text
              h1     h2     h3     h4     h5
brca1      0.111 -0.043  0.051 -0.090 -0.006
mutations  0.042 -0.059  0.215 -0.156  0.041
increase   0.136 -0.104  0.190 -0.102  0.007
breast     0.039 -0.055  0.098  0.012  0.015
cancer     0.093 -0.011  0.136 -0.064  0.021
risk       0.138 -0.036  0.193 -0.077  0.039
```

The magnitudes are smaller than the RNN's because `h_t` is the product of a
near-0.5 output gate and a small cell state, so the cell state values in
`C_lstm` are worth inspecting too. Again, no semantic interpretation is
possible with random weights.

---

## 9. Bidirectional RNNs and LSTMs (slide 14)

A standard left-to-right model only sees the left context. For `breast` in

```text
BRCA1 mutations increase breast cancer risk
```

a forward RNN knows `BRCA1 mutations increase` but has not yet processed
`cancer risk`. For NER the whole sentence is available, so it pays to also run
the sequence in reverse.

```python
H_forward = run_rnn(X, W_x, W_h, b_h)
H_backward_reversed = run_rnn(X[::-1], W_x, W_h, b_h)
H_backward = H_backward_reversed[::-1]
```

```text
Forward shape : (6, 5)
Backward shape: (6, 5)
```

Two details worth being explicit about:

1. The backward pass runs over the reversed sequence, so its output rows are in
   reverse token order. Reversing them again (`[::-1]`) puts them back in the
   original order.
2. In this notebook the backward pass reuses `W_x` and `W_h`. A real BiRNN uses
   **separate parameters** for the two directions. Sharing them is a
   simplification for demonstration only.

The two rows for the same token then carry complementary information:

```text
forward representation  → influenced by BRCA1 mutations increase breast
backward representation → influenced by breast cancer risk
```

They are combined by concatenation:

\[
h_t = [\overrightarrow{h_t}; \overleftarrow{h_t}]
\]

```python
H_bidirectional = np.concatenate([H_forward, H_backward], axis=1)
```

```text
Bidirectional representation shape: (6, 10)
```

Five dimensions per direction, ten in total. One representation per token,
informed by both sides. This is especially useful for offline text analytics,
where the full document is already available.

---

## 10. Why this matters for information extraction (slide 15)

The full sequence-labelling picture:

```text
tokens
   ↓
static embeddings
   ↓
BiRNN / BiLSTM
   ↓
contextual token representations
   ↓
label classifier
```

The neural sequence model does not decide what a `GENE`, `DISEASE`, or
`LEGAL_PROVISION` is. That depends on the annotation scheme, on the training
examples, and possibly on domain resources and expert knowledge. The role of
the recurrent model is to supply useful context-sensitive representations.

### 10.1 A conceptual NER example

```python
example_labels = ["B-GENE", "O", "O", "B-DISEASE", "I-DISEASE", "O"]
```

```text
        token illustrative_gold_label
0      brca1                  B-GENE
1  mutations                       O
2   increase                       O
3     breast               B-DISEASE
4     cancer               I-DISEASE
5       risk                       O
```

A good contextual representation should let a classifier separate
`BRCA1 → B-GENE`, `breast → B-DISEASE`, and `cancer → I-DISEASE`.

The change relative to Lecture 7:

```text
static embedding:
"cancer" → one type-level vector

BiLSTM representation:
this occurrence of "cancer" → vector informed by its sentence context
```

---

## 11. Final comparison

| Representation | What it contains |
|---|---|
| Static embedding `x_t` | information associated with the word type |
| RNN hidden state `h_t` | current token + preceding sequence |
| LSTM hidden state `h_t` | contextual representation with gated memory management |
| LSTM cell state `c_t` | explicit recurrent memory pathway |
| Bidirectional state | left context + right context |

The point of the lecture is not to memorise the internal equations. It is to
understand why these architectures were introduced and what representation they
produce.

---

## 12. Take-home messages

1. An RNN processes text sequentially, combining the current token with
   information carried from earlier positions.
2. The hidden state is contextual. It represents a token occurrence after
   processing its preceding sequence.
3. The same recurrent parameters are reused at every position.
4. Simple RNNs have difficulty preserving useful long-distance information, and
   training can suffer from vanishing or exploding gradients.
5. An LSTM introduces an explicit memory state and learned gates. The model
   learns what to forget, what to write, and what to expose.
6. Bidirectional models use both left and right context, which is particularly
   useful for NER and other sequence-labelling tasks.
7. For text analytics, the useful output of this machinery is a
   context-sensitive representation for each token.

Next lecture:

```text
RNN / LSTM:
context propagated sequentially

Transformer:
context integrated through attention
```

---

## 13. Dry examples and self-checks

Work through these before the next session. Answers use the shipped seed
(`rng = np.random.default_rng(42)`), so you should reproduce the numbers
exactly if you keep the cell order.

### Exercise 1: shape bookkeeping

Fill in the blanks.

1. `W_embed` has shape `(6, 4)`. What does the 6 stand for and what does the 4
   stand for?
2. `W_h` has shape `(5, 5)`. Why is the first 5 the same number as the second?
3. `label_scores = H @ W_label` gives shape `(6, 4)`. Where do the 6 and the 4
   come from?
4. After concatenation, `H_bidirectional` has shape `(6, 10)`. Why 10 and not 5?

<details>
<summary>Answers</summary>

1. 6 = number of word types in the vocabulary, 4 = embedding dimension
   (one row per word, one column per coordinate of the embedding).
2. The first 5 is the hidden dimension of the incoming hidden state, the second
   5 is the hidden dimension of the outgoing hidden state. They match because
   the hidden state has one fixed size throughout the sequence.
3. The 6 comes from `H` (one row per token) and the 4 from `len(labels)`, the
   number of candidate labels.
4. 10 = 5 dimensions from the forward pass plus 5 from the backward pass.

</details>

### Exercise 2: first versus second step

1. With `h0 = np.zeros(5)`, what does `h1` depend on? Give the formula.
2. What extra information does `h2` carry compared with `h1`?
3. By the same argument, what does `h6` depend on?

<details>
<summary>Answers</summary>

1. `h1 = tanh(W_x @ x1 + W_h @ h0 + b) = tanh(W_x @ x1 + b)`, so it depends on
   `BRCA1` only.
2. `h2` also depends on `h1`, which encodes `BRCA1`. It is the state after
   reading `BRCA1 mutations`.
3. `h6` depends on `x6` and `h5`, and by induction on all six tokens. In
   principle the whole preceding sequence is available to it.

</details>

### Exercise 3: reading the hidden-state table

The table from section 3.5 has a `cancer` row of
`[0.015, 0.567, -0.203, -0.274, 0.119]`.

1. Does `0.015` mean "the hidden state at time 1 for the token cancer"?
2. What does it mean?
3. Where would you look to see how much of `BRCA1` is still present in the
   representation of `risk`?

<details>
<summary>Answers</summary>

1. No. The columns index the coordinates of the hidden vector, not the time
   steps. The time step is the row.
2. It is the first coordinate of the hidden vector computed at the position of
   `cancer`, which depends on `embedding("cancer")` and on the state after
   `breast`.
3. Nowhere directly. There is no dimension you can read off. The influence of
   `BRCA1` is distributed across the vector, and with random weights you
   cannot measure it at all. Interpreting coordinates is only meaningful after
   training.

</details>

### Exercise 4: gates in words

Given the gate vectors printed in section 7.4:

1. What would the forget gate `[0.561, 0.501, 0.560, 0.416, 0.420]` do to a
   memory whose coordinates were `[1.0, 1.0, 1.0, 1.0, 1.0]`?
2. Is this LSTM making a single decision per token or one per dimension?
3. Why is the output gate not simply 1?

<details>
<summary>Answers</summary>

1. It scales the memory down to `[0.561, 0.501, 0.560, 0.416, 0.420]`, that
   is, it keeps between 42% and 56% of each coordinate. Nothing is fully kept
   and nothing is fully erased.
2. One per dimension. Each of the 5 coordinates gets its own gate value, so
   the unit can keep coordinate 3 while suppressing coordinate 4.
3. Because the visible state `h_t` and the internal memory `c_t` play
   different roles. The output gate controls how much of the memory is
   exposed at this step, so a memory can be retained for later while
   remaining partially hidden now.

</details>

### Exercise 5: the long-distance example

Take the sentence `The flights the airline was canceling were full.`

1. At the token `were`, which earlier token carries the information the model
   needs about grammatical number?
2. Which competing cue sits closer to `were`?
3. A forward RNN reads this sentence left to right. Which of the two cues is
   farther from `were` in terms of recurrent updates?
4. How does a bidirectional model change the answer to question 1?

<details>
<summary>Answers</summary>

1. `flights`, the subject of the clause.
2. `airline`, the singular noun inside the relative clause.
3. `flights`. It sits further back, so its contribution has passed through
   more recurrent updates and has had more opportunities to be washed out.
4. It does not change which token holds the information, but the backward pass
   lets the model see `were full` while processing `flights`, so the evidence
   from the right is available too. Whether that helps depends on the task and
   on training, which is exactly the empirical question NER raises.

</details>

### Exercise 6: extending the notebook

Small modifications worth trying, in increasing order of effort.

1. Change `embedding_dim` to 6 and `hidden_dim` to 8. What shapes do `X`, `H`,
   and `label_scores` take now?
2. Change the vocabulary so that the sentence has 10 tokens. Does anything in
   `run_rnn` need to change? (Answer: no, which is the point of parameter
   sharing.)
3. Run the backward pass with a separate set of weights (`W_x_b`, `W_h_b`) and
   confirm the bidirectional shape is still `(n, 2 * hidden_dim)`.
4. Replace the `argmax` with a softmax and compare the resulting probabilities
   with the raw scores. What does the softmax change and what does it leave
   untouched?
5. Train a tiny LSTM on the toy sequence labelling task with `scikit-learn`'s
   `MLPClassifier` acting as the label head, or implement backpropagation by
   hand for the two gate matrices. Once the weights are trained, revisit the
   `cancer` row and describe what the dimensions appear to encode.

---

## 14. Notation reference

| Symbol | Meaning | Shape in the notebook |
|---|---|---|
| `x_t` | static embedding of the token at position `t` | (4,) |
| `h_t` | RNN or LSTM hidden state at position `t` | (5,) |
| `c_t` | LSTM cell state at position `t` | (5,) |
| `h_(t-1)` | previous hidden state, input to the recurrence | (5,) |
| `W_embed` | embedding table, one row per word type | (6, 4) |
| `W_x` | input-to-hidden transform | (5, 4) for the RNN |
| `W_h` | hidden-to-hidden transform | (5, 5) for the RNN |
| `b_h` | hidden bias | (5,) |
| `W_label` | hidden-to-label-score transform | (5, 4) |
| `W_forget`, `W_input`, `W_candidate`, `W_output` | LSTM gate transforms | (5, 9) each |
| `z` | concatenation of `h_(t-1)` and `x_t` inside the LSTM | (9,) |
| `H` | matrix of hidden states, one row per token | (6, 5) |
| `C_lstm` | matrix of cell states, one row per token | (6, 5) |
| `H_bidirectional` | forward and backward states concatenated | (6, 10) |
| `f_t`, `i_t`, `o_t` | forget, input, and output gates | (5,) each, values in (0, 1) |

---

## 15. How the notebook maps onto the slides

| Slide | Topic | Notebook section (the notebook's own numbering) |
|---|---|---|
| 1 | Title and scope | Introductory markdown cell |
| 2 | Why sequence models | 1, 1.1, 1.2 |
| 3 | From embeddings to a sequence | 1.2 |
| 4 | What an RNN adds | 2, 2.1, 2.2 |
| 5 | The hidden state | 2.3, 2.4, 3 |
| 6 | What is shared across time | 4, 4.1 |
| 7 | Uses of an RNN | 5 |
| 8 | RNNs for sequence labelling | 5.1, 5.2 |
| 9 | Long-distance dependencies | 6, 6.1 |
| 10 | Why simple RNNs struggle | 6.2 |
| 11 | LSTM: the big idea | 7 |
| 12 | LSTM gates | 7.1, 7.2 |
| 13 | One LSTM step | 8, 8.1, 8.2, 8.3 |
| 14 | Bidirectional RNNs and LSTMs | 10, 10.1, 10.2 |
| 15 | Why this matters for IE | 11, 11.1 |
| 16 | Takeaway | 12, final comparison table |

---

## 16. Common pitfalls

1. **Reading a column name as a time index.** The `h1 ... h5` column labels in
   the hidden-state table are coordinates, not steps. Rename them to `dim_i`.
2. **Interpreting random numbers.** Every weight in the notebook comes from
   `rng.normal`. The predictions in section 5.2 are noise, and saying that
   `breast` was labelled `I-DISEASE` says nothing about the model.
3. **Forgetting to reverse the backward pass twice.** `run_rnn(X[::-1])`
   returns states in reverse token order. One more `[::-1]` restores the
   original order.
4. **Assuming a BiRNN shares weights across directions.** The notebook does it
   for simplicity; real implementations do not.
5. **Expecting the LSTM to output labels.** It outputs vectors. Labels come
   from the classifier on top.
6. **Confusing a global gate with a per-dimension gate.** Gates are vectors.
   There is no single "remember or forget" switch.
7. **Confusing `h_t` and `c_t`.** `h_t` is what downstream layers see, `c_t` is
   the internal memory.
8. **Expecting a BiLSTM to help in a streaming setting.** Bidirectional models
   need the whole sequence. For text analytics with complete documents that
   is fine, for real-time tagging it is not.
