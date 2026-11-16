# Supervised Named Entity Recognition — Theory Companion

**Source notebook:** `ATA_L11_Named_Entity_Recognition_I_Supervised_NER_companion.ipynb`  
**Course:** Advanced Text Analytics — Lecture 11  
**Writing guide:** ASD-STE100 Simplified Technical English (target: about 80%)  
**Document type:** Theory notes. This document does not replace the notebook.

---

## Summary

The notebook shows supervised NER as a sequence of clear steps. BIO changes entity spans into one label for each token. A decoder changes the labels back into spans. Evaluation uses an exact match of start, end, and type. A linear head classifies contextual vectors. A small Transformer learns the contextual vectors and the head together. The notebook uses a small dataset to show the mechanics, not to get a high score.

---

## 0. About this document

### 0.1 Purpose

This document explains the theory in the source notebook. It answers three questions for each part:

1. What is the problem?
2. Why does the notebook use this method?
3. What does the method do?

The document does not copy the code. It gives the model of the method that the code implements. The code shows *how*. This document explains *why*.

### 0.2 Language note

ASD-STE100 is an English standard for technical text. This document uses the standard as a guide. The target is about 80% compliance. We keep these items unchanged:

- code names, for example `B-GENE`;
- mathematical symbols, for example \(h_i\);
- standard machine-learning terms, for example "Transformer".

We define these terms in Section 0.3. We use short sentences, active voice, and one meaning for each word. We avoid idioms and metaphors. We use "must" for a requirement and "can" for a possibility.

### 0.3 Terms

#### Text and labels

| Term | Meaning in this document |
|---|---|
| Annotation | A label that a human gives to a text. |
| Gold label | The correct label from the annotation. Also "reference label". |
| Prediction | The label that the model gives. |
| Token | A small unit of text. A token can be a word, a part of a word, or a symbol. |
| Subword | A part of a word. A tokenizer can split a word into subwords. |
| Entity | A text span with a type, for example a gene name. |
| Span | A sequence of tokens. We identify a span by a start position and an end position. |
| Boundary | The start and the end of a span. |
| Label | A class name, for example `GENE` or `O`. |
| Class | One of the possible labels. |
| BIO | A label scheme with `B-`, `I-`, and `O`. |
| Token classification | The task of giving one label to each token. |

#### Model and training

| Term | Meaning in this document |
|---|---|
| Classifier | A model that gives a class to an input. |
| Feature | A number or a vector that describes an input. |
| Vector | An ordered list of numbers. |
| Matrix | A table of numbers. |
| Embedding | A vector that represents a token or a position. |
| Hidden state | A vector that a neural network computes for a token. |
| Logit | A raw score before softmax. |
| Probability | A number between 0 and 1. |
| Softmax | A function that turns logits into probabilities. |
| Loss | A number that measures the error of the model. |
| Cross-entropy | A loss function for classification. |
| Gradient | A vector that shows the direction to change a parameter. |
| Parameter | A value that the model learns, for example a weight. |
| Weight | A parameter in a matrix. |
| Bias | A parameter that we add to a linear function. |
| Learning rate | A small number that controls the size of a parameter update. |
| Epoch | One full pass over the training data. |
| Batch | A small group of examples. |
| Padding | Extra tokens that make all sequences the same length. |
| Mask | A signal that tells the model which positions to ignore. |
| Encoder | A neural network that reads tokens and gives contextual vectors. |
| Transformer | A neural network architecture that uses self-attention. |
| Self-attention | A method that lets each token use information from other tokens. |
| BERT | A pretrained Transformer encoder for text. |
| GELU | A smooth activation function. |
| Adam | An optimizer for neural networks. |
| Head | A small layer on top of the encoder. The head gives the task output. |
| Fine-tuning | Training a pretrained model on a new task. |
| Frozen | Not trainable. The parameters do not change. |
| Inference | The use of a trained model to make predictions. |
| Generalization | The ability to work on new data. |
| Overfitting | A model fits the training data but does not generalize. |

#### Evaluation

| Term | Meaning in this document |
|---|---|
| True positive (TP) | A correct predicted entity. |
| False positive (FP) | An incorrect extra predicted entity. |
| False negative (FN) | A gold entity that the model does not find. |
| Precision | The fraction of predicted entities that are correct. |
| Recall | The fraction of gold entities that the model finds. |
| F1 | The harmonic mean of precision and recall. |

---

## 1. The NER task

### 1.1 Definition

Named Entity Recognition (NER) has two outputs for each entity:

1. the text span;
2. the entity type.

Example:

```text
Text:     BRCA1 mutations increase breast cancer risk
Entity 1: BRCA1          → GENE
Entity 2: breast cancer  → DISEASE
```

A model that gives only the type is not sufficient. A model that gives only the span is not sufficient. NER needs both. This is the central idea of the lecture.

### 1.2 Why NER is a structured problem

The output is not one label for the full sentence. The output is a set of spans. The number of spans is not fixed. The spans can have different lengths. Two entities can be adjacent. Therefore, we cannot use a simple sentence classifier.

We need a method that does two steps:

1. give a label to each token;
2. rebuild the spans from the labels.

BIO connects these two steps.

### 1.3 Supervised learning

In supervised NER, we have a training set of texts with gold entity annotations. The training loop has four steps:

1. The model reads a text and gives a prediction.
2. We compare the prediction with the gold annotation.
3. The comparison gives a loss.
4. The loss gives gradients. The gradients change the parameters.

The notebook shows this loop at two levels:

- a small classifier with fixed features;
- a small Transformer with trainable features.

The first level isolates the classifier. The second level shows the full model.

### 1.4 The full pipeline

```text
raw text
   ↓
tokenizer
   ↓
tokens
   ↓
encoder (contextual vectors)
   ↓
NER head (scores for each label)
   ↓
label sequence
   ↓
decoder (spans)
   ↓
evaluation
```

The notebook isolates each step. This is the main didactic idea. Do not read the notebook as a production system. Read it as a sequence of separate mechanisms.

---

## 2. BIO labels

### 2.1 Why we need a label for each token

A neural network needs a fixed output format. A set of spans has a variable format. BIO changes the set of spans into one label for each token. Then the problem becomes a token classification problem. The network can give one label to each token.

### 2.2 The three symbols

- `O`: the token is outside an entity.
- `B-TYPE`: the token is the first token of an entity of type TYPE.
- `I-TYPE`: the token is inside an entity of type TYPE, but it is not the first token.

Example:

| Token | BIO label | Meaning |
|---|---|---|
| BRCA1 | `B-GENE` | First token of a GENE entity. |
| mutations | `O` | Outside an entity. |
| increase | `O` | Outside an entity. |
| breast | `B-DISEASE` | First token of a DISEASE entity. |
| cancer | `I-DISEASE` | Inside the same DISEASE entity. |
| risk | `O` | Outside an entity. |

### 2.3 Boundaries

The `B-` label marks the start. The `I-` label marks the continuation. The `O` label or a new `B-` label marks the end. Therefore, BIO stores the boundary.

If we used only `GENE` and `DISEASE`, we would know the type but not the boundary. Example: "breast cancer" and "cancer" can both have the label `DISEASE`. Without `B-` and `I-`, we cannot know if the two tokens form one entity or two. BIO solves this problem.

### 2.4 Adjacent entities

If two entities of the same type are adjacent, the second entity starts with `B-`, not `I-`.

Example:

```text
BRCA1 BRCA2
B-GENE B-GENE
```

The `B-` label always starts a new entity. This is a rule of BIO. A model that gives `I-` here makes an invalid sequence.

### 2.5 Invalid sequences

BIO can produce invalid sequences:

- `I-GENE` at the first position;
- `I-GENE` after `O`;
- `I-GENE` after `B-DISEASE`;
- `I-GENE` after `I-DISEASE`.

A real model can make these errors. The decoder must handle them. The notebook uses a simple repair rule: an invalid `I-TYPE` starts a new entity of type TYPE.

This rule is a design choice. Other decoders can drop the token, use a constrained model, or use a transition system. The important point: the training labels are valid, but the predictions can be invalid.

---

## 3. Label encoding

### 3.1 Strings to integers

A neural network works with numbers. We make a label list:

```text
["O", "B-GENE", "I-GENE", "B-DISEASE", "I-DISEASE"]
```

We give each label an integer. The mapping must be fixed. The same mapping must be used in training and inference. If the mapping changes, the model output has no meaning.

### 3.2 Two directions

We need two maps:

- `label_to_id`: label → integer. We use it to make training targets.
- `id_to_label`: integer → label. We use it to read the model output.

The notebook defines both. This is a small point, but it prevents a common error: the model predicts index 2, and the code reads the wrong label.

### 3.3 The output layer

The model does not output a label string. It outputs one score for each label. For a token \(i\), the output is a vector:

\[
z_i \in \mathbb{R}^{K}
\]

Here \(K\) is the number of labels. In the notebook, \(K=5\). The index of the largest score gives the predicted label.

---

## 4. Decoding BIO

### 4.1 The state machine

The decoder reads the label sequence from left to right. It keeps two state values:

- `start`: the start position of the current entity, or none;
- `type`: the type of the current entity, or none.

For each token, the decoder does one of three actions:

1. `O`: close the current entity, if one is open.
2. `B-TYPE`: close the current entity, then start a new entity at this token.
3. `I-TYPE`: continue the current entity if the type matches. If the type does not match, repair the sequence and start a new entity.

At the end of the sequence, the decoder closes the last entity.

### 4.2 End position

The decoder uses an end position that is exclusive. If an entity starts at token 3 and ends at token 5, the span is `[3, 5)`. This span includes tokens 3 and 4. This convention is common in Python and in spaCy. It makes the length equal to `end - start`.

### 4.3 Why the decoder matters

The model gives labels. The evaluation and the application need spans. The decoder connects the two. A decoder can change the result.

The notebook shows this with the prediction:

```text
Gold:       B-GENE O O B-DISEASE I-DISEASE O
Prediction: B-GENE O O O         B-DISEASE O
```

The model finds `cancer` as a DISEASE, but the gold span is `breast cancer`. The predicted span is shorter than the gold span. This is a **boundary error**. The model has the correct type, but the wrong boundary.

---

## 5. Entity-level evaluation

### 5.1 Exact match

For strict NER evaluation, a predicted entity is correct only if three items match:

1. start position;
2. end position;
3. type.

We compare sets of triples `(start, end, type)`. The text is not part of the comparison, because the positions define the text.

### 5.2 TP, FP, FN

- **TP**: the triple is in both gold and prediction.
- **FP**: the triple is in prediction but not in gold.
- **FN**: the triple is in gold but not in prediction.

The notebook uses set operations:

\[
TP = |G \cap P|, \quad FP = |P - G|, \quad FN = |G - P|
\]

Here \(G\) is the gold set and \(P\) is the predicted set.

### 5.3 Precision, recall, F1

\[
\text{Precision} = \frac{TP}{TP + FP}
\]

\[
\text{Recall} = \frac{TP}{TP + FN}
\]

\[
F_1 = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}
\]

Definitions:

- **Precision**: of all entities that the model predicts, how many are correct?
- **Recall**: of all gold entities, how many does the model find?
- **F1**: one number that balances precision and recall.

If the denominator is zero, the notebook returns 0.0. This is a convention. It prevents a division by zero.

### 5.4 Error types

A strict comparison gives a clear error taxonomy:

| Error type | What happens | Effect on TP, FP, FN |
|---|---|---|
| Missed entity | A gold entity is not predicted. | One FN. |
| Extra entity | A predicted entity is not in gold. | One FP. |
| Boundary error | The type is correct, but the start or the end is wrong. | One FP and one FN. |
| Type error | The span is correct, but the type is wrong. | One FP and one FN. |

The notebook's example is a boundary error. The gold entity is `breast cancer` → DISEASE. The prediction is `cancer` → DISEASE. The predicted triple is not in gold. Therefore, the model gets one FP and one FN.

### 5.5 Why token accuracy is not sufficient

Token accuracy is the fraction of tokens with the correct label. It can be high even when the entity result is bad.

Reason: most tokens are often `O`. A model that predicts `O` for every token can get high accuracy on a corpus with few entities. But its recall is zero.

Therefore, we must use entity-level metrics for NER. The notebook shows both numbers to make this point. Token accuracy is useful for debugging, but it is not the main metric.

### 5.6 Strict vs partial match

The notebook uses strict match. Some evaluation scripts use partial match: an entity is correct if the span overlaps and the type is correct. Partial match gives higher scores. It can hide boundary errors.

The choice must be explicit. For a fair comparison, use the same rule for all systems. Report the rule with the score.

---

## 6. The token classification head

### 6.1 Contextual vectors

Assume an encoder gives one vector for each token:

\[
H = [h_1, \ldots, h_n], \quad h_i \in \mathbb{R}^{d}
\]

The vector \(h_i\) is contextual: it depends on the token and on the other tokens in the sentence. In the notebook, \(H\) is a small fixed matrix. In a real system, a Transformer computes \(H\).

### 6.2 Linear classifier

We use one shared linear layer for all tokens:

\[
z_i = h_i W + b
\]

- \(W \in \mathbb{R}^{d \times K}\) is a weight matrix.
- \(b \in \mathbb{R}^{K}\) is a bias vector.
- \(z_i \in \mathbb{R}^{K}\) is the vector of logits for token \(i\).

The layer is shared: we use the same \(W\) and \(b\) for every position. This is important. It lets the model use the same decision rule at every token. It also keeps the number of parameters small.

### 6.3 Softmax

Softmax turns logits into probabilities:

\[
p_{i,k} = \frac{\exp(z_{i,k})}{\sum_{j=1}^{K} \exp(z_{i,j})}
\]

The probabilities for one token sum to 1. The predicted label is the index with the largest probability.

The notebook subtracts the maximum logit before `exp`. This is a numerical trick. It does not change the result. It prevents overflow.

### 6.4 Cross-entropy

For one token with gold label \(y_i\), the loss is:

\[
\ell_i = -\log p_{i,y_i}
\]

The mean loss over the batch is:

\[
L = \frac{1}{n} \sum_{i=1}^{n} \ell_i
\]

If the model gives a high probability to the gold label, the loss is small. If the model gives a low probability, the loss is large. This is the training signal.

### 6.5 Gradient descent

The notebook computes the gradient by hand:

\[
\frac{\partial L}{\partial z_i} = \frac{1}{n}(p_i - e_{y_i})
\]

Here \(e_{y_i}\) is a one-hot vector, and \(Z\) is the matrix of all logits. Then:

\[
\frac{\partial L}{\partial W} = H^T \frac{\partial L}{\partial Z}
\]

\[
\frac{\partial L}{\partial b} = \sum_i \frac{\partial L}{\partial z_i}
\]

The update is:

\[
W \leftarrow W - \eta \frac{\partial L}{\partial W}
\]

\[
b \leftarrow b - \eta \frac{\partial L}{\partial b}
\]

\(\eta\) is the learning rate. The notebook uses \(\eta=0.5\) and 250 full-batch steps. The loss decreases.

### 6.6 Frozen encoder, trainable head

In this part, \(H\) does not change. Only \(W\) and \(b\) change. This is a **frozen encoder + trainable head** setup.

The encoder is a feature extractor. The head learns the NER decision. This setup is useful for teaching because we can see the classifier alone. It is also a real option when the encoder is big and the labeled data is small.

### 6.7 The simplification

The notebook uses three labels: `O`, `GENE`, `DISEASE`. It does not use `B-` and `I-`. Therefore, this classifier cannot represent boundaries. It can only give a type to each token.

The notebook says this explicitly. The goal is to isolate the classifier mechanics. The next part adds the Transformer and the BIO labels.

---

## 7. Subword alignment

### 7.1 Why tokenizers split words

BERT-like tokenizers use a fixed vocabulary of subwords. A rare word can be split into several subwords. Example:

```text
interleukin-6  →  inter   ##leukin   -   6
```

The `##` prefix shows that the subword continues the previous subword. The model sees subwords, but the annotation is at the word level. We must transfer the label.

### 7.2 Two common strategies

| Subword | Label all subwords | Label first only |
|---|---|---|
| inter | `B-PROTEIN` | `B-PROTEIN` |
| ##leukin | `I-PROTEIN` | `IGNORE` |
| - | `I-PROTEIN` | `IGNORE` |
| 6 | `I-PROTEIN` | `IGNORE` |

Strategy 1 gives a label to every subword. Strategy 2 gives a label to the first subword and ignores the other subwords.

### 7.3 Consistency

The strategy is an implementation choice. The important requirement is consistency:

- If training ignores the extra subwords, evaluation must also ignore them.
- If training labels all subwords, evaluation must decode at the word level or merge the subword predictions.

If the two phases use different rules, the metrics are not comparable. The notebook makes this point.

### 7.4 Special tokens and padding

A BERT input can have special tokens, for example `[CLS]` and `[SEP]`. These tokens have no entity label. Padding tokens also have no entity label.

We use an ignore index, for example `-100`, so the loss does not use these positions. The notebook uses `IGNORE_INDEX = -100` for padding.

---

## 8. The small Transformer

### 8.1 Architecture

```text
input token IDs
   ↓
token embeddings + position embeddings
   ↓
Transformer encoder layer
   ├─ multi-head self-attention
   ├─ residual + layer norm
   ├─ feed-forward network
   └─ residual + layer norm
   ↓
contextual hidden states
   ↓
linear NER head
   ↓
logits for each BIO label
```

The notebook uses one layer, two heads, \(d_{model}=16\), and an FFN size of 32. The model is small on purpose. The goal is to see all parts.

### 8.2 Token embeddings

An embedding table maps each token ID to a vector of size \(d_{model}\). The table is a parameter matrix of size \(V \times d_{model}\), where \(V\) is the vocabulary size.

Training changes the rows for the tokens in the training data. Tokens that never occur in the training data keep their initial values or use `<UNK>`.

### 8.3 Positional embeddings

Self-attention compares tokens, but it has no built-in order. If we permute the input tokens, the attention output permutes in the same way. Therefore, we add a position vector to each token vector:

\[
x_i = E_{tok}(t_i) + E_{pos}(i)
\]

The notebook learns the position embeddings. Other models use fixed sinusoidal encodings or relative positions. The result is the same in principle: the model gets information about the order.

### 8.4 Self-attention

For each token, the model makes three vectors:

\[
q_i = x_i W_Q, \quad k_i = x_i W_K, \quad v_i = x_i W_V
\]

These are the query, key, and value vectors.

The attention score between token \(i\) and token \(j\) is:

\[
a_{ij} = \frac{q_i \cdot k_j}{\sqrt{d_k}}
\]

Here \(d_k\) is the dimension of the key vectors. The division by \(\sqrt{d_k}\) keeps the scores in a useful range.

We apply softmax over \(j\):

\[
\alpha_{ij} = \frac{\exp(a_{ij})}{\sum_{j'} \exp(a_{ij'})}
\]

The output for token \(i\) is:

\[
o_i = \sum_j \alpha_{ij} v_j
\]

This output mixes information from all tokens. For NER, this is useful: the model can use the left context and the right context. The notebook uses an encoder, so attention is bidirectional. There is no causal mask.

### 8.5 Multi-head attention

One attention head can learn one type of relation. Multi-head attention uses \(h\) heads in parallel. Each head has its own \(W_Q, W_K, W_V\). The outputs are concatenated and projected:

\[
\text{MultiHead}(X) = \text{Concat}(head_1, \ldots, head_h) W_O
\]

Here \(W_O\) is the output projection matrix. The notebook uses two heads. The model can learn two relation patterns.

### 8.6 Feed-forward network

After attention, each position goes through a small network:

\[
\text{FFN}(x) = W_2 \, \text{GELU}(W_1 x + b_1) + b_2
\]

The same FFN is used for every position. It adds non-linearity and capacity. The notebook uses a hidden size of 32.

### 8.7 Residual and layer norm

Each sublayer has a residual connection and layer normalization:

\[
x \leftarrow \text{LayerNorm}(x + \text{Sublayer}(x))
\]

The residual connection helps the gradient flow. Layer normalization makes training more stable. The notebook uses post-layer normalization (`norm_first=False`).

### 8.8 Padding mask

A batch can have sentences of different lengths. We pad the short sentences with a special token. The padding token must not affect the result.

The notebook uses two mechanisms:

1. a padding mask with `True` at padding positions; the attention layer ignores these positions;
2. an ignore index `-100` for the labels; the loss ignores these positions.

These two mechanisms are necessary. The mask controls the attention. The ignore index controls the loss.

### 8.9 NER head and loss

The encoder gives a hidden state \(h_i \in \mathbb{R}^{16}\) for each token. The NER head is a linear layer:

\[
z_i = h_i W_{ner} + b_{ner}, \quad z_i \in \mathbb{R}^{5}
\]

The five scores match the five BIO labels. We apply cross-entropy at each non-padding position:

\[
L = -\sum_{i \in \text{valid}} \log P(y_i \mid x)
\]

The notebook uses `nn.CrossEntropyLoss(ignore_index=-100)`. This function combines softmax and cross-entropy. It is numerically stable. By default, PyTorch uses the mean over the valid positions. The sum in the formula and the mean differ by a constant factor. The factor changes the gradient size, but not the direction of the update.

### 8.10 Full training

The notebook passes `model.parameters()` to Adam. Therefore, all parameters are trainable:

- token embeddings;
- position embeddings;
- attention matrices;
- feed-forward matrices;
- layer normalization parameters;
- NER head.

This is full training of the toy model. It is not pretraining. The model starts from random values. The notebook trains on six sentences for 250 epochs. The loss becomes very small. The model memorizes the data. This is expected. The goal is to show the mechanics, not to get good generalization.

### 8.11 Frozen encoder experiment

Exercise 4 sets `requires_grad=False` for all parameters, then `True` for the NER head. The optimizer uses only the trainable parameters.

This is head-only training. It is the same idea as Section 6, but now the encoder is a Transformer. In a real BERT fine-tuning setup, we start from pretrained values. We can update all parameters (full fine-tuning) or a subset (parameter-efficient fine-tuning).

---

## 9. What changes during training

### 9.1 Parameter groups

| Component | Role | Trainable in full training | Trainable in head-only training |
|---|---|---|---|
| Token embeddings | Map token IDs to vectors. | Yes | No |
| Position embeddings | Give order information. | Yes | No |
| Attention matrices | Compute Q, K, V, and output. | Yes | No |
| FFN matrices | Add non-linearity. | Yes | No |
| Layer norm | Normalize activations. | Yes | No |
| NER head | Map hidden state to labels. | Yes | Yes |

### 9.2 Pretrained vs random start

The notebook starts from random values. A real system starts from a pretrained encoder. The loss and the gradient flow are the same in principle. The difference is the starting point.

A pretrained start usually gives better features and better generalization, especially with small labeled data. The notebook says this in Section 7. Do not read the toy result as a production result.

---

## 10. Limits and correct interpretation

### 10.1 What the notebook shows

- how to turn spans into BIO labels;
- how to decode BIO labels;
- how to compute entity-level metrics;
- how a linear head classifies tokens;
- how a small Transformer trains end-to-end;
- how subword alignment works in principle.

### 10.2 What the notebook does not show

- a real dataset;
- a train/dev/test split;
- a pretrained encoder;
- hyperparameter tuning;
- statistical significance;
- a comparison with a baseline;
- production performance.

The notebook says this. Do not read the very small training loss as a good NER system. It is memorization.

### 10.3 What to add for a real project

1. A clear annotation guideline.
2. A train/dev/test split.
3. A pretrained encoder, for example BERT or a biomedical model.
4. Entity-level precision, recall, and F1 on the test set.
5. An error analysis with the four error types.
6. A baseline, for example a dictionary or an EntityRuler.
7. A check of token alignment and subword alignment.
8. A fixed random seed and repeated runs.

---

## 11. Key points

1. **NER predicts a span and a type.** Both are necessary.
2. **BIO turns spans into token labels.** `B-` starts an entity. `I-` continues it. `O` is outside.
3. **The decoder rebuilds spans.** It must handle invalid label sequences.
4. **Evaluation must be entity-level.** A correct type with a wrong boundary is not a correct entity under strict match.
5. **Token accuracy can hide NER errors.** Most tokens are often `O`.
6. **A shared linear head maps each contextual vector to label scores.** Softmax gives probabilities. Cross-entropy gives the loss.
7. **A frozen encoder with a trainable head is a valid setup.** It isolates the classifier.
8. **Subword alignment is a practical requirement.** Training and evaluation must use the same rule.
9. **A Transformer gives one contextual vector for each token.** Attention mixes the left and right context.
10. **Full training updates all parameters.** Head-only training updates only the head.
11. **The toy model memorizes six sentences.** This shows the mechanics, not generalization.

---

## Appendix A: Shapes

| Object | Shape | Meaning |
|---|---|---|
| \(H\) | \(n \times d\) | One vector for each of \(n\) tokens. |
| \(W\) | \(d \times K\) | Weight matrix for \(K\) labels. |
| \(b\) | \(K\) | Bias for \(K\) labels. |
| Logits | \(n \times K\) | One score for each token and label. |
| Probabilities | \(n \times K\) | Softmax output. |
| Input IDs | \(B \times L\) | \(B\) sentences with \(L\) positions. |
| Token embeddings | \(B \times L \times d_{model}\) | One vector for each position. |
| Encoder output | \(B \times L \times d_{model}\) | Contextual hidden states. |
| NER logits | \(B \times L \times K\) | One score for each position and label. |
| Labels | \(B \times L\) | Gold label for each position. |

---

## Appendix B: Writing rules used

This document uses these ASD-STE100 rules as a guide:

- Use short sentences. Most sentences have fewer than 25 words.
- Use active voice. Name the actor.
- Use one term for one concept. Do not use "tag" for "label".
- Define technical terms before use.
- Use "must" for a requirement and "can" for a possibility.
- Avoid idioms, metaphors, and phrasal verbs.
- Use a list for a sequence of steps.
- Use a table for a set of parallel items.
- Use "for example" instead of "e.g.".
- Use "that is" instead of "i.e.".

The document keeps code names and mathematical symbols unchanged. A full dictionary check is not part of this document.
