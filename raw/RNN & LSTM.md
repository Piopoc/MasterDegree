# Recurrent Neural Networks (RNNs) and Long Short-Term Memory (LSTMs) in Text Analytics

## Executive Summary

Static word embeddings assign a single fixed vector representation to each word type, failing to capture the dynamic, context-dependent nature of natural language. In contrast, **sequence models** compute representations for individual token occurrences in context, acknowledging that word order significantly alters semantic meaning (e.g., "dog bites man" versus "man bites dog").

Recurrent Neural Networks (RNNs) process natural language sequentially by passing a continuous, learned **hidden state** (h_t) forward across time steps. Rather than instantiating separate neural networks for each token, an RNN reuses a single set of shared weight matrices (\mathbf{U}, \mathbf{W}, and \mathbf{V}) across the entire sequence. This design enables context to propagate incrementally throughout text sequences of arbitrary length.

RNN architectures support key natural language processing (NLP) tasks, including whole-text classification (via pooled or final hidden states) and **sequence labeling** (assigning a label decision to every single token state). Sequence labeling serves as a foundational component for tasks such as Part-of-Speech (POS) tagging and Named Entity Recognition (NER). To overcome basic RNN limitations in managing long-range dependencies, extended variants such as **Long Short-Term Memory (LSTM)** networks introduce explicit memory states and learned gating mechanisms, while **bidirectional models** capture both left and right context. Ultimately, while newer **Transformer** models replace sequential recurrence with attention mechanisms, the foundational goal remains identical: generating deep contextual representations of linguistic tokens.

## The Imperative for Sequence Modeling in Text Analytics

Traditional bag-of-words models and static word embedding techniques process text elements independently, treating language as a set of isolated tokens. However, effective text analytics requires recognizing that text is inherently sequential and context-bound.

- **Significance of Word Order**: Syntax and meaning are critically dependent on token arrangement. For instance, the sequence _"dog bites man"_ communicates a fundamentally different event than _"man bites dog"_, despite containing identical token types.
- **Contextual Token Interpretation**: The precise interpretation of any individual token depends on preceding tokens and, frequently, subsequent tokens within the sentence.
- **Static vs. Dynamic Contextual Embeddings**: Static embeddings generate a single vector per word type, regardless of usage. Sequence models calculate dynamic representations that reflect the specific occurrence of a token within its surrounding context.

The fundamental objective of sequence modeling is to derive a mathematical representation at position t that directly incorporates information from the surrounding sequence.

## Mathematical and Structural Framework of Simple RNNs

### Information Flow and Context Propagation

An RNN processes a sentence as an ordered sequence of input vectors \mathbf{x}_1, \mathbf{x}_2, \dots, \mathbf{x}_n, where each \mathbf{x}_t represents the input token embedding at time step t. Context is propagated incrementally across time steps using a hidden state vector h_t:

1. **Initial Step (****h_1****)**: Computed directly using the first token embedding \mathbf{x}_1.
2. **Second Step (****h_2****)**: Depends on the current input \mathbf{x}_2 and the previous hidden state h_1, thereby indirectly incorporating information from \mathbf{x}_1.
3. **Third Step (****h_3****)**: Depends on \mathbf{x}_3 and h_2, indirectly containing information from both \mathbf{x}_1 and \mathbf{x}_2.
4. **Arbitrary Step (****h_t****)**: In principle, subsequent hidden states can encode information spanning the entire preceding token sequence.

**Key Distinction**: The hidden state h_t is not a literal, symbolic memory store; rather, it functions as a continuous, learned vector representation of the sequence history up to step t.

#### Example Sequence Vectorization:

For the biomedical sequence: \text{x}_1 \text{ (BRCA1)} \longrightarrow \text{x}_2 \text{ (mutations)} \longrightarrow \text{x}_3 \text{ (increase)} \longrightarrow \text{x}_4 \text{ (breast)} \longrightarrow \text{x}_5 \text{ (cancer)}

The network updates hidden states sequentially: h_1 \longrightarrow h_2 \longrightarrow h_3 \longrightarrow h_4 \longrightarrow h_5

### Feedforward Architecture and Parameter Sharing

A core property of the RNN framework is parameter sharing across time steps. Rather than instantiating a distinct neural network for each position in a text, a single recurrent model is applied iteratively.

|   |   |
|---|---|
|Parameter / Layer Matrix|Functional Description|
|\mathbf{x}_t|Input token embedding vector at time step t.|
|h_{t-1}|Hidden state vector carried over from the prior time step t-1.|
|\mathbf{U}|Weight matrix multiplied by the prior hidden state h_{t-1}.|
|\mathbf{W}|Weight matrix multiplied by the current input vector \mathbf{x}_t (feedforward component).|
|\mathbf{V}|Weight matrix applied to the current hidden state h_t to compute output y_t.|
|h_t|Current hidden state, generated by combining \mathbf{U} h_{t-1} and \mathbf{W} \mathbf{x}_t.|
|y_t|Model output prediction at time step t.|

```
                       y_t
                        ▲
                        │  V
                      h_t
                     ▲   ▲
                    ╱     ╲
        U  ┌───────┴┐     └───────┐  W
  h_{t-1} ─┤   +    │             │─ x_t
           └────────┘             
```

_(Architectural concept adapted from Jurafsky & Martin, 2026, Figure 13.2)_

## Applications in Text Analytics and Information Extraction

RNNs process textual sequences for downstream analytical tasks using two primary architectural configurations:

```
                    ┌─────────────────────────────────────────┐
                    │       Text Analytics Applications       │
                    └────────────────────┬────────────────────┘
                                         │
                    ┌────────────────────┴────────────────────┐
                    │                                         │
        ▼                                       ▼
┌───────────────────────────────┐       ┌───────────────────────────────┐
│     Text Classification       │       │       Sequence Labeling       │
├───────────────────────────────┤       ├───────────────────────────────┤
│ Uses final or pooled hidden   │       │ Evaluates each token state    │
│ state to assign a single      │       │ individually to assign a      │
│ label to the entire document. │       │ tag per position (POS, NER).  │
└───────────────────────────────┘       └───────────────────────────────┘
```

### Sequence Labeling Framework

In sequence labeling, the network generates one contextual state h_t per token, enabling a distinct label decision at every sequence position.

1. **Input**: Token embedding e (or \mathbf{x}_t) at position t.
2. **Context Transformation**: RNN layer(s) transform e into context-dependent hidden state h_t.
3. **Scoring & Classification**: Output projection V h_t is passed through a **Softmax** layer over the target tag set, followed by an **Argmax** function to select the highest-probability label (y).

#### Example 1: Part-of-Speech (POS) Tagging Pipeline

As defined by Jurafsky & Martin (2026, Figure 13.7), POS tagging assigns grammatical labels from a predefined set of tags across an input sentence:

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|Sequence Position (t)|Token Word|Input Embedding (e)|Hidden State (h)|Softmax Output Tag|Tag Description|
|**1**|Janet|e_1|h_1|**NNP**|Proper Noun|
|**2**|will|e_2|h_2|**MD**|Modal Verb|
|**3**|back|e_3|h_3|**VB**|Verb|
|**4**|the|e_4|h_4|**DT**|Determiner|
|**5**|bill|e_5|h_5|**NN**|Singular Noun|

#### Example 2: Named Entity Recognition (NER)

Sequence labeling connects directly to Information Extraction (IE) tasks such as Named Entity Recognition. Classifiers evaluate hidden states h_t to assign entity tags, such as:

- `B-GENE` (Beginning of a gene entity)
- `I-DISEASE` (Inside a disease entity)
- `O` (Outside any target entity)

## Advanced Architectures and Future Directions

While basic RNNs establish the foundation for sequence modeling, standard recurrent connections suffer from limitations in retaining long-distance context and incorporating future context.

### Long Short-Term Memory (LSTM) Networks

To manage information flow over long sequences more effectively, LSTMs introduce:

- An **explicit memory state** (cell state) running parallel to the hidden state.
- **Learned gating mechanisms** designed to control the flow of information:
    - _Forgetting Gate_: Determines what information to discard from the memory state.
    - _Writing/Input Gate_: Determines what new information to store in the memory state.
    - _Exposing/Output Gate_: Determines what parts of the memory state to emit into the hidden state.

### Bidirectional Models

Standard RNNs process sequences strictly from left to right (x_1 \to x_n). However, understanding token context in text analytics frequently depends on subsequent tokens. **Bidirectional models** execute dual recurrent runs—processing context from left-to-right and right-to-left simultaneously—and concatenate these representations. This structure is particularly effective for NER and sequence labeling tasks.

### Evolution to Transformers

Modern NLP architectures frequently replace recurrent transitions with self-attention mechanism structures (**Transformers**). However, the fundamental objective remains identical to that of RNNs and LSTMs: converting static tokens into rich, context-dependent sequential representations.

## Key Source Citations

- **Jurafsky, D., & Martin, J. H. (2026).** _Speech and language processing_ (3rd ed.).
    - _Figure 13.2_: Illustrates a simple recurrent neural network as a unrolled feedforward network utilizing weight matrices \mathbf{U}, \mathbf{W}, and \mathbf{V}.
    - _Figure 13.7_: Demonstrates Part-of-Speech tagging as a sequence labeling task utilizing input embeddings, RNN layer hidden states, and a softmax activation layer over grammatical tags (`NNP`, `MD`, `VB`, `DT`, `NN`).