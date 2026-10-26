---
type: course-note
date: 2026-10-26
tags:
  - course
  - advanced-text-analytics
  - transformers
  - attention
  - contextual-representations
  - nlp
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA09]]"
  - "[[From Recurrence to Contextual Representations]]"
  - "Jurafsky & Martin (2026) - Transformer chapter"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 9 / ATA09) on **Transformer architecture, attention mechanisms, and contextual token representations**. Created 2026-10-26 by merging two `raw/` sources:

- `[[ATA09]]` — detailed lecture companion on Transformers, attention, Q/K/V, scaled dot-product attention, multi-head attention, positional encodings, causal vs bidirectional masking, Transformer blocks, and the NER pipeline (with NumPy notebook walk-through, hand-specified toy vectors, reproducible numbers from untrained random weights).
- `[[From Recurrence to Contextual Representations]]` — theoretical briefing on the shift from recurrent state compression (RNN/LSTM) to direct O(1) pairwise attention, language modeling as next-token probability estimation, autoregressive generation, self-supervised pretraining, post-pretraining alignment (instruction tuning, RLHF), and Transformer model roles (decoder-only / encoder-only / encoder-decoder).

Use this note when a question involves: static vs contextual embeddings, attention as weighted sum, Query/Key/Value roles, scaled dot-product attention, self-attention parallelism vs recurrence, multi-head attention, positional encodings, causal masking, bidirectional encoding, Transformer block composition (LayerNorm, residual, FFN), stacking blocks, decoder-only/encoder-only/encoder-decoder configurations, autoregressive decoding with temperature/top-k/top-p, or the token-to-NER-label pipeline. Central caveat: every printed number in the lecture comes from **untrained random weights** — the values illustrate mechanics, never model quality.

---

## 1. Why Contextual Representations

A static embedding is **type-level**: one vector per word type, identical everywhere the type appears. The word `bank` gets the same vector in "river bank" and "investment bank" — two different meanings that a static embedding cannot tell apart.

A Transformer representation is **token-level and contextual**: the vector for `bank` in "I walked along the pond and saw the bank" is influenced by `pond`, differently from how it would be influenced by `investment`.

The motivating claim of this lecture:

> A Transformer builds the representation of a token by selectively integrating information from other token positions.

RNNs and LSTMs (Lecture 8, see [[Recurrent Neural Networks and LSTMs]]) solve context by propagating a hidden state sequentially. Transformers solve it differently: **attention lets a token directly gather information from other relevant positions in the sequence, without recurrence.**

The conceptual path:

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

## 2. Attention as a Weighted Sum

Before Query/Key/Value, attention is introduced as the simplest possible mechanism: a weighted combination of context vectors.

Suppose we know the importance weights for the context of `bank`:

| token | weight |
|-------|--------|
| I | 0.02 |
| walked | 0.03 |
| pond | 0.55 |
| trees | 0.15 |
| bank | 0.25 |

The weights sum to 1. The contextual representation is:

$$
a_i = \sum_j \alpha_{ij} x_j
$$

where $(x_j)$ is a token vector and $(\alpha_{ij})$ is the weight assigned to token $(j)$ when representing token $(i)$. The weights indicate **how much each context vector contributes**.

Attention = selective information integration.

---

## 3. Where Do the Attention Weights Come From?

In a real model, weights are not hand-specified. They are computed from a similarity score between the current token and each possible context token. The notebook uses the dot product as a first approximation:

$$
\text{score}(x_i, x_j) = x_i \cdot x_j
$$

A larger dot product means the vectors are more aligned. A `softmax` then turns arbitrary scores into attention weights that are positive, normalized, and sum to 1.

For `bank` compared against every context token, the dot products are converted by softmax into a distribution of relevance. The largest score does **not** become a weight of 1: several tokens can contribute to the new representation, each with a different share.

---

## 4. Why Query, Key, and Value?

The simplified version above uses the same vector both to compare and to combine. Transformers make this more flexible by projecting each input representation into three different roles:

$$
q_i = x_i W_Q
$$
$$
k_i = x_i W_K
$$
$$
v_i = x_i W_V
$$

Conceptually:

| Role | Question it answers |
|------|---------------------|
| Query | what am I looking for? |
| Key | how should I be matched? |
| Value | what information should I contribute? |

**Key idea:** matching information and transferred information do not need to use the same representation.

Every token ends up with its own query, key, and value — three different learned views of the same input.

---

## 5. Scaled Dot-Product Attention

To decide how much token \(i\) should attend to token \(j\), Transformers compare the **query of \(i\)** with the **key of \(j\)**:

$$
\text{score}(i, j) = \frac{q_i \cdot k_j}{\sqrt{d_k}}
$$

The division by $(\sqrt{d_k})$ keeps dot-product magnitudes under control as dimensionality grows — large dot products push softmax into regions with tiny gradients.

Because all tokens are stored as rows of a matrix \(X\), every comparison is computed at once with:

$$
Q @ K^T
$$

The result is a square matrix of size (tokens × tokens): rows are queries, columns are keys. Softmax is applied independently across every row, producing the **attention matrix** $(A)$. Each row sums to 1, and entry $(A[i,j])$ means:

> how much token position \(i\) uses the information from token position \(j\).

---

## 6. Using the Weights to Combine the Value Vectors

The attention matrix tells us *where to get information from*. The \(V\) matrix contains the information that will actually be combined:

$$
\text{contextualized output} = A @ V
$$

For each token position, this computes a weighted sum of the value vectors. The output row for `bank` is no longer just a transformed version of the original `bank` vector: it contains a weighted combination of information from `pond`, `near`, and `bank`, according to the learned attention distribution.

This is the central mechanism that creates a **contextual representation**.

---

## 7. Self-Attention Transforms the Whole Sequence in Parallel

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

| Architectural Parameter | Recurrent Pathways (RNN / LSTM) | Desired Direct Pathways |
|-------------------------|----------------------------------|--------------------------|
| Computational Dependency | Strictly sequential: position t depends on hₜ₋₁ | Fully parallelizable across all token positions |
| Interaction Path Length | O(N) sequential steps between distant tokens | O(1) direct pairwise access operations |
| Context Memory Capacity | Lossy: forced compression into fixed vector hₜ | Non-lossy: uncompressed, direct relational mapping |

---

## 8. Multi-Head Attention

A Transformer normally uses several attention **heads** in parallel, not just one. Each head has its own \(W_Q, W_K, W_V\), so each head is free to learn a different pattern of relevance. The outputs of all heads are then concatenated (and, in a real model, passed through a learned output projection).

$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O
$$
$$
\text{where } \text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$

With two heads of dimension 2, concatenation gives a vector of size 4 again — matching the model dimension.

**Caution:** it is tempting to assign a clean linguistic interpretation to every head (e.g. "this head tracks subject-verb agreement"). Some heads do learn recognizable patterns, but the model is not required to organize each head into one clean human-readable function. The useful intuition is simply:

> multiple heads provide multiple learned ways to gather contextual information.

---

## 9. Transformers Need Explicit Position Information

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

$$
X_{\text{input}} = X_{\text{token}} + X_{\text{position}}
$$

The resulting vector represents both *what token is here?* and *where is it in the sequence?*. This positional information can influence all subsequent attention calculations. (Real Transformers use learned or mathematically constructed position encodings; the notebook uses random ones only to show the operation.)

---

## 10. Causal Attention and Masking

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

## 11. Bidirectional Attention for Text Analytics

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

## 12. What Is a Transformer Block?

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
- **Feed-forward network:** transforms each token representation separately with a non-linear transformation. It does not mix positions. $(\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2)$
- **Residual connections:** add the component input back to its output, so the original representation keeps flowing while layers add new information. They also pass uninterrupted gradient signals.
- **Layer normalization:** rescales each token representation into a numerically stable range; mainly important for training deep Transformers reliably. $(\text{Output} = \text{LayerNorm}(x + \text{SubLayer}(x)))$

---

## 13. Why Stack Many Transformer Blocks?

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

## 14. Language Modeling as Probabilistic Prediction

Before dissecting inner neural mechanisms, a language model is best analyzed through a functional black-box abstraction: estimating probability distributions over discrete token sequences.

Formally, given a sequence of tokens $(X = (x_1, x_2, \dots, x_N))$, the joint probability distribution $(P(X))$ over the entire sequence is factorized via the chain rule of probability:

$$
P(X) = \prod_{i=1}^{N} P(x_i \mid x_1, x_2, \dots, x_{i-1})
$$

The core function of a Large Language Model (LLM) is strictly to model these conditional probability distributions $(P(x_i \mid x_{<i}))$.

**Input Context (Prefix):** an ordered sequence of context tokens $(X_{<i} = (x_1, x_2, \dots, x_{i-1}))$.
**Output Distribution:** a categorical probability distribution over the full discrete vocabulary $(V)$, mapping every candidate token $(w \in V)$ to a likelihood score $(P(x_i = w \mid X_{<i}))$.

An LLM does not natively "write prose" or execute deliberate reasoning in a single step. Rather, it computes static categorical distributions over a defined vocabulary. Turning continuous text generation into a controllable process requires explicit inference sampling strategies over these predicted probability distributions.

---

## 15. The Mechanics of Autoregressive Generation

Autoregressive generation converts static, single-step probability distributions $(P(x_t \mid X_{<t}))$ into continuous, multi-token sequences. By appending each predicted output back into the context buffer, the system sequentially constructs complex textual outputs.

The formal autoregressive execution loop proceeds through four stages:

1. **Predict:** Compute the conditional probability distribution over the vocabulary $(V)$ for the current sequence step $(t)$: $(P(x_t \mid x_1, x_2, \dots, x_{t-1}))$
2. **Choose / Sample:** Select a specific token $(x_t \in V)$ from the output probability distribution according to a defined decoding strategy
3. **Append:** Update the running context sequence: $(X_{\le t} = (x_1, x_2, \dots, x_{t-1}, x_t))$
4. **Iterate:** Re-evaluate the model using $(X_{\le t})$ as the new input prefix and repeat steps 1–3 until an end-of-sequence (EOS) token is produced or a sequence length limit is reached.

```
+------------------------+      +------------------------+
|       1. PREDICT       | ---> |    2. CHOOSE/SAMPLE    |
| P(x_t | x_1, ..., x_t-1)|      | Apply Temperature/Sampling|
+------------------------+      +------------------------+
            ^                               |
            |                               v
+------------------------+      +------------------------+
|       4. ITERATE       | <--- |       3. APPEND        |
| Evaluate updated context|      | X_<=t = (X_<t , x_t)   |
+------------------------+      +------------------------+
```

The characteristics of the generated text depend heavily on decoding hyper-parameters. Before passing unnormalized logit outputs \(z_i\) into the softmax function, a temperature parameter \(T > 0\) is applied:

$$
P(x_i) = \frac{\exp(z_i / T)}{\sum_{j=1}^{|V|} \exp(z_j / T)}
$$

- **High Temperature (T > 1.0):** Flattens the probability distribution, increasing entropy and yielding diverse, highly stochastic sampling outcomes.
- **Low Temperature (0 < T < 1.0):** Sharpens the distribution around high-probability modes, reducing entropy and driving generation toward deterministic or greedy selection ($(T \to 0)$).
- **Top-k sampling:** restricts selection to the k most probable tokens.
- **Top-p (nucleus) sampling:** dynamically truncates the candidate set to the smallest set of tokens whose cumulative probability exceeds threshold p.

While raw self-supervised pretraining equips a base model with strong next-token prediction capabilities, minimizing cross-entropy loss over uncurated web text merely incentivizes statistical imitation. To transform raw statistical continuation into structured, helpful assistant behavior, models undergo post-pretraining alignment — specifically instruction tuning and Reinforcement Learning from Human Feedback (RLHF).

| RAW PRETRAINED BASE MODEL | POST-PRETRAINING ALIGNED |
|---------------------------|--------------------------|
| Objective: Cross-entropy loss on web text | Objective: User intent, safety, and task execution |
| Behavior: Statistical imitation | Behavior: Instruction-following |
| Output: Plausible completion | Output: Helpful/Safe assistant |

---

## 16. Capabilities and Limitations of Self-Supervised Sequence Learning

Language model pretraining uses self-supervised learning over vast, unannotated text corpora. By optimizing a simple objective — predicting the next token via cross-entropy loss minimization — the model implicitly extracts complex linguistic structures, world knowledge, and semantic abstractions without human labeling.

Through large-scale next-token prediction, models automatically map complex regularities into their weight parameters:

- **Lexical and Semantic Associations:** roses, dahlias, peonies → flowers; big → enormous
- **Factual Patterns and World Knowledge:** "A Room of One's Own" → Virginia Woolf; √4 → 2
- **Syntactic Regularities:** complex grammatical structures, agreement rules, nested linguistic features

Because the model encodes these relationships directly within its parameter space, **prompting** serves as an in-context task specification interface. Modifying the prefix context $(X_{<i})$ shifts the model's conditional distribution, steering generation to perform specific tasks without modifying underlying model parameters.

**Key caveats:** because self-supervised pretraining extracts statistical correlations from uncurated corpora, base models reflect all flaws, biases, and inaccuracies present in their training data:

- **Inherited Biases:** unfiltered training data embeds historical stereotypes, harmful associations, and toxic linguistic patterns directly into the model's probability predictions.
- **Absence of Truthfulness Guarantees:** cross-entropy loss optimizes for statistical plausibility, not factual accuracy or logical validity. Highly fluent, grammatically flawless text can easily convey completely hallucinated or incorrect information.

Architectural capacity alone is insufficient for reliable deployment. Models require post-pretraining alignment to realign output distributions with human intent, safety guidelines, and task accuracy.

---

## 17. Transformer Components and Model Roles

The Transformer architecture resolves the recurrent bottleneck by replacing step-by-step state propagation with parallelized multi-head self-attention. Instead of squeezing context through intermediate state vectors, attention dynamics enable direct O(1) relational connections between every pair of tokens in a sequence regardless of distance.

Pairwise relevance scores are computed by scaled dot-products between Queries and Keys, passed through a softmax operator, and multiplied by Values:

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

Individual attention layers are integrated into modular Transformer processing blocks (see Section 12). Depending on structural masking and directional attention settings, this modular block can be configured into three primary model architectures:

| Architectural Configuration | Attention Directionality | Operational Specialization | Exemplar Paradigm |
|-----------------------------|--------------------------|----------------------------|-------------------|
| **Decoder-Only** | Unidirectional (Causal masking) | Autoregressive generation & next-token prediction | GPT Series, LLaMA |
| **Encoder-Only** | Bidirectional (Full context access) | Feature extraction, sequence classification | BERT, RoBERTa |
| **Encoder-Decoder** | Cross-Attention (Bidirectional + Causal) | Sequence transduction & translation | T5, BART |

---

## 18. Contextual Representations for Information Extraction

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

## 19. RNN/LSTM versus Transformer

| RNN / LSTM | Transformer |
|------------|-------------|
| context propagated sequentially | context integrated with attention |
| hidden state moves through positions | token positions interact through attention |
| order naturally encoded by recurrence | position must be represented explicitly |
| computation sequential across positions | attention computations parallelizable |
| contextual representation at each position | contextual representation at each position |

The goal is similar — produce representations that depend on context — but the mechanism is different.

---

## Take-Home Messages

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
11. Language modeling factorizes sequence probability into next-token conditionals via the chain rule.
12. Autoregressive generation iterates Predict → Choose/Sample → Append → Iterate.
13. Self-supervised pretraining extracts linguistic structure implicitly; prompting steers generation without parameter changes.
14. Decoder-only, encoder-only, and encoder-decoder Transformer configurations serve different operational specializations.

---

## Key Formulas (Quick Reference)

- Weighted sum: $(\displaystyle a_i = \sum_j \alpha_{ij} x_j)$
- Dot-product score: $(\displaystyle \text{score}(x_i, x_j) = x_i \cdot x_j)$
- Q/K/V projections: $(\displaystyle q_i = x_i W_Q,\; k_i = x_i W_K,\; v_i = x_i W_V)$
- Scaled dot-product attention: $(\displaystyle \text{score}(i,j) = \frac{q_i \cdot k_j}{\sqrt{d_k}})$
- Attention matrix: $(\displaystyle A = \text{row\_softmax}\left(\frac{Q K^T}{\sqrt{d_k}}\right))$
- Multi-head attention: $(\displaystyle \text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O)$
- Transformer input: $(\displaystyle X_{\text{input}} = X_{\text{token}} + X_{\text{position}})$
- FFN: $(\displaystyle \text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2)$
- Residual + Norm: $(\displaystyle \text{Output} = \text{LayerNorm}(x + \text{SubLayer}(x)))$
- Chain rule for language modeling: $(\displaystyle P(X) = \prod_{i=1}^{N} P(x_i \mid x_{<i}))$
- Temperature scaling: $(\displaystyle P(x_i) = \frac{\exp(z_i / T)}{\sum_{j=1}^{|V|} \exp(z_j / T)})$

---

## Suggested Review Questions

1. Why can't a static embedding distinguish "river bank" from "investment bank"?
2. What does each row of the attention matrix \(A\) represent, and why does it sum to 1?
3. What is the difference between query, key, and value in attention?
4. Why divide by \(\sqrt{d_k}\) in scaled dot-product attention?
5. How does self-attention differ from an RNN in terms of computation and parallelism?
6. Why do Transformers require explicit position encodings?
7. What does a causal mask do, and when is bidirectional attention preferable?
8. What are the roles of residual connections and layer normalization in a Transformer block?
9. How do contextual token vectors connect to a downstream NER classifier?
10. What are the three primary Transformer model configurations, and what is each specialized for?
11. How does temperature affect autoregressive sampling?
12. What is the difference between a base model and a post-pretraining aligned model?

---

## Sources & Connections

- `[[ATA09]]` — Lecture 9 companion on Transformers, attention, and contextual representations (NumPy notebook walk-through).
- `[[From Recurrence to Contextual Representations]]` — Theoretical briefing on the shift from recurrent state compression to direct O(1) attention, language modeling, autoregressive generation, and Transformer model roles.
- `[[Recurrent Neural Networks and LSTMs]]` — Lecture 8: the recurrent baseline that Transformers replace.
- `[[TF-IDF and Static Embeddings]]` — Lecture 7: static embeddings and their type-level limitation, which contextual representations resolve.
- Jurafsky & Martin (2026) — Transformer chapter underlying the lecture.