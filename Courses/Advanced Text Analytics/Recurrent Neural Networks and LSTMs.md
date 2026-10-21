---
type: course-note
date: 2026-10-21
tags:
  - course
  - advanced-text-analytics
  - sequence-models
  - rnn
  - lstm
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L08]]"
  - "[[RNN & LSTM]]"
  - "Jurafsky & Martin (2026) - via [[RNN & LSTM]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 8) on **recurrent models for text**: what an RNN adds on top of static embeddings, why simple RNNs lose long-distance information, and how LSTMs, bidirectional runs and a label classifier turn a token sequence into contextual representations for sequence labelling (POS tagging, NER). Created 2026-10-21 by merging [[ATA_L08]] (detailed companion to the lecture slides and the NumPy notebook, with reproducible dry-run numbers) and [[RNN & LSTM]] (concise briefing that carries the Jurafsky & Martin figure references). Use it when a question involves the hidden-state recurrence, parameter sharing across time, vanishing/exploding gradient intuition, LSTM gates, cell state versus hidden state, bidirectional concatenation, or the token-to-label pipeline. Central caveat: every printed number in the lecture comes from **untrained random weights** - the values illustrate the mechanics, never model quality.

## 1. Why Sequence Models

A static embedding assigns one vector to a word **type**, and that vector is identical everywhere the type appears:

```text
BRCA1    -> vector
mutation -> vector
cancer   -> vector
```

The vector for `cancer` cannot tell apart `breast cancer`, `cancer treatment`, and `cancer risk`. Two further limits drive the lecture:

- **Word order carries meaning.** `dog bites man` and `man bites dog` contain the same token types but describe opposite events.
- **A token's role depends on its neighbours.** Interpretation depends on which tokens came before, on their order, and often on which tokens come after.

Text is an **ordered sequence** $\mathbf{x}_1, \mathbf{x}_2, \dots, \mathbf{x}_n$, not a bag. The objective of sequence modelling is a representation at position $t$ that directly incorporates information from the surrounding sequence. The conceptual path of the whole lecture:

```text
static embeddings
      -> ordered sequence of embeddings
      -> RNN
      -> context-dependent hidden states
      -> long-distance problem
      -> LSTM
      -> explicit memory + gates
      -> bidirectional sequence modelling
      -> contextual representations for NER
```

### 1.1 From a sentence to a matrix of vectors

The lecture keeps a six-word biomedical vocabulary and a 4-dimensional embedding table:

```text
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

Looking up the sentence produces `X` of shape `(6, 4)` - one row per position, one column per embedding dimension. Position now matters: $\mathbf{x}_1$ is processed before $\mathbf{x}_2$, and so on. Everything from here starts **from the same embeddings as Lecture 7**, but consumes them as an ordered sequence.

## 2. What an RNN Adds: the Hidden State

At time $t$ the model combines the current embedding with the previous hidden state and produces a new hidden state:

$$h_t = \tanh(W_x x_t + W_h h_{t-1} + b)$$

Read it as a one-liner:

> current token + previous context -> new context

Two things happen at once: $W_x x_t$ carries information from the current word, $W_h h_{t-1}$ carries information from the preceding context, and `tanh` squashes the sum into $(-1, 1)$.

### 2.1 The recurrent weights

With `hidden_dim = 5`:

| Matrix | Shape | Job |
|---|---|---|
| $W_x$ | (5, 4) | embedding space -> hidden-state space |
| $W_h$ | (5, 5) | previous hidden state -> new hidden state |
| $b_h$ | (5,) | offset added to every hidden unit |

**No new matrices are created for later tokens.** One model is applied repeatedly across the sequence - exactly what lets an RNN handle sequences of different lengths (see Section 3).

### 2.2 One step, then the whole sentence

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

Running the loop over all six tokens yields `H` of shape `(6, 5)` - one hidden vector per token:

```text
              h1     h2     h3     h4     h5
brca1      0.193  0.179  0.416  0.165 -0.123
mutations  0.380 -0.141  0.354  0.082 -0.448
increase   0.222  0.320  0.504  0.193 -0.319
breast     0.046  0.439 -0.273 -0.564 -0.021
cancer     0.015  0.567 -0.203 -0.274  0.119
risk       0.168  0.552 -0.085 -0.415  0.033
```

The conceptual difference from Lecture 7 in one box:

```text
Static embedding:
cancer -> same vector whenever the word type appears

RNN hidden state:
cancer at position t -> vector influenced by the sequence processed before t
```

### 2.3 How to read the hidden-state table (important)

The rows are token positions; the columns are **coordinates of the hidden vector**. This trips up most students on the first pass: the entry `0.015` in the `cancer` row does **not** mean "hidden state at time 1". It is the first coordinate of the vector computed when the model processed `cancer`, i.e. of

$$h_t = \tanh(W_x x_t + W_h h_{t-1} + b)$$

with $x_t = \text{embedding}(\texttt{cancer})$ and $h_{t-1}$ the state after `breast`. To remove the ambiguity, rename columns to `dim_1 ... dim_5`. Keep the two counts apart:

```text
6 tokens -> 6 hidden vectors
5 dims   -> each hidden vector has 5 numerical coordinates
```

A dimension has **no predefined linguistic meaning**: nobody decided in advance that `dim_1` encodes gene information. Coordinates are latent features learned during training, meaning is usually spread across several dimensions, and in this untrained notebook `dim_5` for `cancer` has no interpretation at all. Takeaway:

> The hidden dimensions are coordinates of a learned representation, not named linguistic features. What matters is the hidden vector as a whole.

## 3. What Is Shared Across Time

An unrolled RNN looks like a different network at every position. It is not: the same $W_x$, $W_h$, $b_h$ are reused at $t = 1, 2, \dots, n$; only the values $x_t$, $h_{t-1}$, $h_t$ change.

```text
t=1  token=brca1      first hidden value= 0.193
t=2  token=mutations  first hidden value= 0.380
t=3  token=increase   first hidden value= 0.222
t=4  token=breast     first hidden value= 0.046
t=5  token=cancer     first hidden value= 0.015
t=6  token=risk       first hidden value= 0.168
```

Same computation, different state. This **parameter sharing** is what lets the model accept sequences of variable length: extend the sentence and nothing in the architecture changes. In the textbook diagram (Jurafsky & Martin, Figure 13.2) the same structure appears as a feedforward network with a weight matrix $U$ on the previous hidden state, $W$ on the current input, and $V$ producing the output $y_t$ - the naming differs from the notebook ($W_h$, $W_x$, label head), the roles are identical.

## 4. What an RNN Is Used For

| Task | How the hidden state is used |
|---|---|
| Language modelling | $h_t$ predicts the next token |
| Sequence labelling | $h_t$ predicts the label of token $t$ |
| Sequence classification | final or pooled states predict one label for the whole text (text classification) |

Sequence labelling is the relevant family for this course, because it leads directly to NER.

### 4.1 From hidden states to label scores

Take a small BIO label set and one projection matrix:

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

With trained weights, `BRCA1` would score high on `B-GENE` and `breast` high on `B-DISEASE`. With random weights the scores are noise - `argmax` therefore predicts `B-GENE`, `B-DISEASE`, `B-GENE`, `I-DISEASE`, `I-DISEASE`, `B-GENE`, which is meaningless as a model output but architecturally instructive: **the classifier acts on the contextual hidden state, not on the static embedding**.

```text
token -> embedding -> RNN -> 5-dim contextual vector -> W_label -> 4 label scores -> argmax
```

Note what the RNN does **not** do: it never emits `B-GENE` directly. It emits a contextual representation; a classifier on top turns that representation into task-specific scores.

### 4.2 POS tagging as the textbook example

In the textbook's sequence-labelling framing (Jurafsky & Martin, Figure 13.7), each hidden state is projected through a softmax over the tag set and an argmax picks the tag:

| Position | Token | Hidden state | Softmax tag | Description |
|---|---|---|---|---|
| 1 | Janet | $h_1$ | **NNP** | proper noun |
| 2 | will | $h_2$ | **MD** | modal verb |
| 3 | back | $h_3$ | **VB** | verb |
| 4 | the | $h_4$ | **DT** | determiner |
| 5 | bill | $h_5$ | **NN** | singular noun |

Same machinery as the NER scores above - only the tag set and the training data differ. Both are instances of the token-level annotation schemes studied earlier in the course.

## 5. The Problem: Long-Distance Dependencies

A simple RNN can in principle carry information from the start of a sequence to the end. In practice it struggles. The classic example:

> The flights the airline was canceling were full.

To predict `were` correctly the model needs the plural subject `flights`, but `airline` sits in between and offers a competing singular cue. The useful signal must survive several recurrent updates.

### 5.1 Repeated multiplication as intuition for vanishing gradients

The notebook illustrates the effect with a toy curve rather than a training run:

```python
vanishing  = 0.8  ** steps    # factor below 1
exploding  = 1.15 ** steps    # factor above 1

1.0 -> 0.8 -> 0.64 -> 0.512 -> 0.4096 -> 0.3277 -> ...
```

- After about **10 steps** the signal is at **11%** of its original size; after **30 steps** it is under **0.13%**.
- Multiplied the other way, a factor of 1.15 grows the signal by a factor of **66** over 30 steps.

The graph is **intuition, not proof**: real RNN gradients involve matrices, chain rules and many interacting terms. The general point stands - repeated factors below 1 make signals shrink (vanishing gradient), repeated factors above 1 make them grow rapidly (exploding gradient).

### 5.2 The representational problem

Gradients are not the whole story. The hidden state is asked to do two jobs at once:

1. represent information useful **right now**;
2. preserve information that may only become useful **much later**.

While processing `The flights the airline was canceling ...`, the state must keep encoding `flights` while handling the current tokens. An explicit memory mechanism is designed to fix exactly this.

## 6. LSTM: the Big Idea

An LSTM is still a recurrent model: it still consumes $\mathbf{x}_1, \mathbf{x}_2, \dots$ one position at a time. What it adds is a second recurrent vector, the **cell state** (memory state) $c_t$:

```text
h_t -> visible hidden representation
c_t -> longer-term memory pathway
```

### 6.1 Gates as soft switches

Gates are produced by the sigmoid, so they live in $(0, 1)$:

```text
0   -> mostly block information
0.5 -> partially pass it
1   -> mostly let information pass
```

The sigmoid output acts as a **soft mask** over a memory vector. Given memory `[important, irrelevant, important, irrelevant]`, a learned gate might behave like `[0.95, 0.05, 0.90, 0.10]`, keeping some coordinates and strongly reducing others. Because gate values are learned **per dimension**, the model can preserve some information while suppressing other information.

### 6.2 The three decisions

At every position the LSTM makes three conceptually separate decisions:

| Gate | Question it answers |
|---|---|
| Forget gate | How much of the previous memory $c_{t-1}$ should survive? |
| Input (write) gate | How much new information should be written into memory? |
| Output (expose) gate | How much of the updated memory should become visible in $h_t$? |

### 6.3 One LSTM step

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

The line that carries the whole idea is the cell-state update:

```text
new memory
  = part of old memory we decided to keep
  + part of new information we decided to write
```

In a real deep learning library these four matrices live inside the LSTM layer; the notebook writes them out so the unit's internals stay visible.

### 6.4 Dry run on the first token

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

### 6.5 Cell state versus hidden state

| Vector | Role |
|---|---|
| $c_t$ | the longer-term memory carried forward |
| $h_t$ | the current exposed representation used by the rest of the network |

```text
c_t = what the LSTM is remembering internally
h_t = what the LSTM is showing externally at this step
```

```text
previous memory c_(t-1)
        |
    forget / keep
        +
    add new information
        |
       c_t          internal memory
        |
    output gate
        |
       h_t          visible representation
```

For text analytics, $h_t$ is what you pass to the next layer (e.g. an NER classifier); $c_t$ is mostly an internal mechanism for carrying information along the sequence. In Jurafsky and Martin's terminology, the context layer is maintained for future decisions while the hidden state provides the LSTM's output at each time step.

### 6.6 Running the LSTM over the sentence

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

## 7. Bidirectional RNNs and LSTMs

A standard left-to-right model only sees the left context. For `breast` in

```text
BRCA1 mutations increase breast cancer risk
```

a forward RNN knows `BRCA1 mutations increase` but has not yet processed `cancer risk`. For NER the whole sentence is available, so the sequence is additionally run in reverse:

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

One representation per token, informed by both sides. This fits offline text analytics, where the full document is already available; a streaming/real-time tagger cannot use it.

## 8. Why This Matters for Information Extraction

The full sequence-labelling picture:

```text
tokens
   |
static embeddings
   |
BiRNN / BiLSTM
   |
contextual token representations
   |
label classifier
```

Conceptual NER example with gold labels:

```text
      token illustrative_gold_label
0    brca1                  B-GENE
1mutations                       O
2 increase                       O
3   breast               B-DISEASE
4   cancer               I-DISEASE
5     risk                       O
```

A good contextual representation should let a classifier separate `BRCA1 -> B-GENE`, `breast -> B-DISEASE`, `cancer -> I-DISEASE`.

Scope discipline: the neural sequence model does **not** decide what a `GENE`, `DISEASE` or `LEGAL_PROVISION` is. That depends on the annotation scheme, the training examples, and possibly domain resources and expert knowledge. The role of the recurrent model is to supply useful context-sensitive representations. The change relative to Lecture 7:

```text
static embedding:
"cancer" -> one type-level vector

BiLSTM representation:
this occurrence of "cancer" -> vector informed by its sentence context
```

## 9. Final Comparison

| Representation | What it contains |
|---|---|
| Static embedding $x_t$ | information associated with the word type |
| RNN hidden state $h_t$ | current token + preceding sequence |
| LSTM hidden state $h_t$ | contextual representation with gated memory management |
| LSTM cell state $c_t$ | explicit recurrent memory pathway |
| Bidirectional state | left context + right context |

The point of the lecture is not to memorise the internal equations, but to understand **why these architectures were introduced and what representation they produce**.

## 10. Take-Home Messages

1. An RNN processes text sequentially, combining the current token with information carried from earlier positions.
2. The hidden state is contextual: it represents a token occurrence after processing its preceding sequence.
3. The same recurrent parameters are reused at every position - that is what allows variable-length input.
4. Simple RNNs struggle to preserve useful long-distance information; training can suffer from vanishing or exploding gradients.
5. An LSTM adds an explicit memory state and learned gates: the model learns what to forget, what to write, and what to expose.
6. Bidirectional models use both left and right context - particularly useful for NER and other sequence-labelling tasks.
7. For text analytics, the useful output of this machinery is a context-sensitive representation per token.

```text
RNN / LSTM:     context propagated sequentially
Transformer:    context integrated through attention
```

Transformers are out of scope here, but the goal is identical: converting static tokens into rich, context-dependent sequential representations.

## 11. Common Pitfalls

1. **Reading a column name as a time index.** Column labels are coordinates, not steps; the time step is the row.
2. **Interpreting random numbers.** Every weight comes from `rng.normal`; the printed predictions are noise, and no dimension carries meaning before training.
3. **Forgetting to reverse the backward pass twice.** `run_rnn(X[::-1])` returns states in reverse order; one more `[::-1]` restores it.
4. **Assuming a BiRNN shares weights across directions.** The notebook does it for simplicity; real implementations do not.
5. **Expecting the LSTM to output labels.** It outputs vectors; labels come from the classifier on top.
6. **Confusing a global gate with a per-dimension gate.** Gates are vectors - there is no single remember-or-forget switch.
7. **Confusing $h_t$ and $c_t$.** $h_t$ is what downstream layers see; $c_t$ is the internal memory.
8. **Expecting a BiLSTM to help in a streaming setting.** Bidirectional models need the whole sequence.

## 12. Notation Reference

| Symbol | Meaning | Shape in the notebook |
|---|---|---|
| $x_t$ | static embedding of the token at position $t$ | (4,) |
| $h_t$ | RNN or LSTM hidden state at position $t$ | (5,) |
| $c_t$ | LSTM cell state at position $t$ | (5,) |
| $h_{t-1}$ | previous hidden state, input to the recurrence | (5,) |
| $W_{embed}$ | embedding table, one row per word type | (6, 4) |
| $W_x$ | input-to-hidden transform (RNN) | (5, 4) |
| $W_h$ | hidden-to-hidden transform (RNN) | (5, 5) |
| $b_h$ | hidden bias | (5,) |
| $W_{label}$ | hidden-to-label-score transform | (5, 4) |
| $W_{forget}, W_{input}, W_{candidate}, W_{output}$ | LSTM gate transforms | (5, 9) each |
| $z$ | concatenation of $h_{t-1}$ and $x_t$ inside the LSTM | (9,) |
| $H$ | matrix of hidden states, one row per token | (6, 5) |
| $C$ | matrix of cell states, one row per token | (6, 5) |
| $H_{bi}$ | forward and backward states concatenated | (6, 10) |
| $f_t, i_t, o_t$ | forget, input, output gates | (5,) each, values in (0, 1) |

Textbook naming (Jurafsky & Martin): $U$ = hidden-to-hidden, $W$ = input-to-hidden, $V$ = output projection.

## 13. Self-Check

<details>
<summary>1. Why does $h_2$ contain information about token 1 even though the formula only mentions $x_2$?</summary>

Because $h_2 = \tanh(W_x x_2 + W_h h_1 + b)$, and $h_1$ was computed from $x_1$. The preceding token enters through the previous hidden state, not through the current input.

</details>

<details>
<summary>2. In the hidden-state table, what does the entry in the `cancer` row under `dim_2` (0.567) represent?</summary>

The second coordinate of the hidden vector computed at the position of `cancer`. It depends on `embedding("cancer")` and on the state after `breast`. It does **not** refer to time step 2, and with random weights it has no interpretable meaning.

</details>

<details>
<summary>3. What would the forget gate `[0.561, 0.501, 0.560, 0.416, 0.420]` do to a memory of `[1.0, 1.0, 1.0, 1.0, 1.0]`, and why is the output gate not simply 1?</summary>

It scales the memory to `[0.561, 0.501, 0.560, 0.416, 0.420]` - between 42% and 56% kept per coordinate, nothing fully kept or erased. The output gate is not 1 because $h_t$ and $c_t$ play different roles: a memory can be retained for later while being only partially exposed now.

</details>

<details>
<summary>4. In `The flights the airline was canceling were full.`, which cue is farther from `were` in recurrent updates, and what does a bidirectional model change?</summary>

`flights` is farther back, so its contribution has passed through more recurrent updates and had more opportunity to be washed out (`airline` is the closer, competing singular cue). A bidirectional model does not change which token holds the information, but the backward pass lets the model see `were full` while processing `flights`, making right-side evidence available too - whether that helps is an empirical question.

</details>

## Related

- [[Advanced Text Analytics]] - course hub
- [[TF-IDF and Static Embeddings]] - Lecture 7: static embeddings are the input these sequence models consume, and the one-vector-per-type limit they overcome
- [[Annotation and Evaluation of Information Extraction]] - the BIO tag scheme scored in Section 4 and the evaluation metrics used once a sequence labeler is trained
- [[NLP Linguistic Structures and Annotation]] - the token-level annotation targets (POS, entities) that sequence labelling assigns

## Sources

- [[ATA_L08]] (AY 2026/2027)
- [[RNN & LSTM]] (AY 2026/2027)
- Jurafsky, D., & Martin, J. H. (2026). *Speech and Language Processing*, 3rd ed., Figures 13.2 and 13.7 - cited via the RNN & LSTM source above (AY 2026/2027)
