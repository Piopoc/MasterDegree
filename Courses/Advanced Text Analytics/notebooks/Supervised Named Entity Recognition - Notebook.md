---
type: course-note
date: 2026-11-10
tags:
  - course
  - advanced-text-analytics
  - notebook
  - ner
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L11_Supervised_NER]]"
ai-first: true
---

## For Future Agent

Notebook companion for Lecture 11, part I: theory of the supervised NER companion notebook (`ATA_L11_Named_Entity_Recognition_I_Supervised_NER_companion.ipynb`), extracted 2026-11-10 from the ASD-STE100 theory notes in raw. It covers the pipeline from spans to BIO labels and back, entity-level evaluation, the shared linear classification head (manual softmax/cross-entropy/gradients), subword alignment, and the small Transformer toy model. Central caveats, repeated in the lecture: the toy classifier omits `B-`/`I-` on purpose, and the Transformer memorizes six sentences - **every printed number shows mechanics, never model quality**.

## 1. The NER Task

NER outputs two things per entity - the text span **and** the entity type. Neither alone is enough:

```text
Text:     BRCA1 mutations increase breast cancer risk
Entity 1: BRCA1          -> GENE
Entity 2: breast cancer  -> DISEASE
```

The output is a variable set of (possibly adjacent) spans, not one sentence label, so a plain sentence classifier cannot represent it. The notebook therefore uses a two-step method: give one label to each token, then rebuild the spans from the labels - BIO is the bridge between the two steps.

Supervised training loop (shown at two levels: a small fixed-feature classifier, then a small trainable Transformer):

1. the model reads a text and predicts;
2. we compare the prediction with the gold annotation;
3. the comparison gives a loss;
4. gradients from the loss change the parameters.

Full pipeline as the notebook isolates it:

```text
raw text -> tokenizer -> tokens -> encoder (contextual vectors)
   -> NER head (scores per label) -> label sequence -> decoder (spans) -> evaluation
```

Read the notebook as a sequence of separate mechanisms, not as a production system.

## 2. BIO Labels

BIO turns a variable span set into one label per token (a token classification problem with a fixed output format):

| Symbol | Meaning |
|---|---|
| `O` | token is outside an entity |
| `B-TYPE` | first token of an entity of type TYPE |
| `I-TYPE` | continuation token of the same entity |

Worked example:

| Token | BIO label |
|---|---|
| BRCA1 | `B-GENE` |
| mutations | `O` |
| increase | `O` |
| breast | `B-DISEASE` |
| cancer | `I-DISEASE` |
| risk | `O` |

Why `B-`/`I-` are necessary: with only type labels, "breast cancer" and "cancer" would both be `DISEASE` and the boundary would be lost. Adjacent same-type entities must each start with `B-` (`BRCA1 BRCA2` = `B-GENE B-GENE`); an `I-` there is an invalid sequence.

Invalid predictions do happen (`I-GENE` at position 0, after `O`, or after `B-DISEASE`). The notebook's decoder uses a repair rule: an invalid `I-TYPE` starts a new entity of type TYPE. Other designs (drop the token, constrained model, transition system) are possible - the key point is that **training labels are valid but predictions can be invalid**, so the decoder must handle it.

## 3. Label Encoding and the Output Layer

- A fixed label list (e.g. `["O", "B-GENE", "I-GENE", "B-DISEASE", "I-DISEASE"]`, $K=5$) is mapped to integers; the mapping must be identical in training and inference.
- Two directions are defined: `label_to_id` (to build targets) and `id_to_label` (to read model output) - forgetting one is a classic bug (predicting index 2 but reading the wrong label).
- The model never outputs label strings: for each token $i$ it outputs scores $z_i \in \mathbb{R}^{K}$; the argmax index is the predicted label.

## 4. Decoding BIO to Spans

The decoder scans labels left to right keeping two states (`start`, `type`):

1. `O` - close the current entity, if open;
2. `B-TYPE` - close the current entity, then start a new one at this token;
3. `I-TYPE` - continue if the type matches, else repair and start a new entity.

At sequence end the last entity is closed. End positions are **exclusive**: an entity from token 3 to token 5 is `[3, 5)` (tokens 3 and 4), the Python/spaCy convention, so length = `end - start`.

Why the decoder matters - a boundary error from the notebook:

```text
Gold:       B-GENE O O B-DISEASE I-DISEASE O
Prediction: B-GENE O O O         B-DISEASE O
```

The model finds `cancer` as DISEASE but the gold span is `breast cancer`: correct type, wrong boundary.

## 5. Entity-Level Evaluation

Strict exact match: a predicted entity is correct only if **start, end and type** all match. Sets of triples $(start, end, type)$ are compared (the text itself is not - positions define it):

$$TP = |G \cap P|, \quad FP = |P - G|, \quad FN = |G - P|$$

$$\text{Precision} = \frac{TP}{TP+FP}, \quad \text{Recall} = \frac{TP}{TP+FN}, \quad F_1 = \frac{2 \cdot P \cdot R}{P + R}$$

Zero denominator -> 0.0 (notebook convention). Error taxonomy under strict match:

| Error type | What happens | Effect |
|---|---|---|
| Missed entity | gold entity not predicted | one FN |
| Extra entity | predicted entity not in gold | one FP |
| Boundary error | right type, wrong start/end | one FP + one FN |
| Type error | right span, wrong type | one FP + one FN |

Why token accuracy is not enough: most tokens are often `O`, so a model that predicts `O` everywhere can score high accuracy on a sparse corpus while recall is zero. Token accuracy is a debugging number; the headline metric must be entity-level. Strict vs partial (overlap) matching must be stated explicitly and applied uniformly - partial matching hides boundary errors and inflates scores.

## 6. The Token Classification Head

Assume an encoder gives contextual vectors $H = [h_1, \ldots, h_n]$, $h_i \in \mathbb{R}^{d}$ (in this part $H$ is a small **fixed** matrix). One shared linear layer for all tokens:

$$z_i = h_i W + b, \quad W \in \mathbb{R}^{d \times K},\ b \in \mathbb{R}^{K}$$

Sharing $W, b$ across positions means one decision rule everywhere and few parameters. Then:

- **Softmax** $p_{i,k} = \exp(z_{i,k}) / \sum_j \exp(z_{i,j})$ - the notebook shifts by the max logit before `exp` (numerical stability, no change in result);
- **Cross-entropy** $\ell_i = -\log p_{i,y_i}$, batch loss $L = \frac{1}{n}\sum_i \ell_i$;
- **Manual gradients**: $\partial L/\partial z_i = \frac{1}{n}(p_i - e_{y_i})$, then $\partial L/\partial W = H^T \partial L/\partial Z$ and $\partial L/\partial b = \sum_i \partial L/\partial z_i$;
- **Update** $W \leftarrow W - \eta \frac{\partial L}{\partial W}$ with $\eta = 0.5$, 250 full-batch steps - the loss decreases.

This part is a **frozen encoder + trainable head** setup: $H$ never changes, only $W$ and $b$. It isolates the classifier for teaching, and it is also a real option when the encoder is big and labels are scarce. Simplification stated in the notebook: this classifier uses only `O`, `GENE`, `DISEASE` - no `B-`/`I-`, so it **cannot represent boundaries**; the next part adds the Transformer and full BIO.

## 7. Subword Alignment

BERT-like tokenizers split rare words into subwords (`interleukin-6 -> inter ##leukin - 6`, `##` = continuation), but annotations are word-level, so labels must be transferred:

| Strategy | Rule | Trade-off |
|---|---|---|
| Label all subwords | first gets `B-`, rest get `I-` | more signal, needs merging at decode |
| Label first only | first keeps its label, rest get `IGNORE` (-100) | simpler loss, information discarded |

The choice is an implementation detail; the hard requirement is **consistency** - training and evaluation must use the same rule, else metrics are not comparable. Special tokens (`[CLS]`, `[SEP]`) and padding have no entity label; the notebook uses `IGNORE_INDEX = -100` so the loss skips them.

## 8. The Small Transformer

Architecture as the notebook builds it (deliberately tiny: 1 layer, 2 heads, $d_{model}=16$, FFN 32):

```text
input token IDs
   -> token embeddings + position embeddings
   -> Transformer encoder layer
        multi-head self-attention -> residual + layer norm
        feed-forward network      -> residual + layer norm
   -> contextual hidden states
   -> linear NER head -> logits per BIO label
```

- **Token embeddings**: a $V \times d_{model}$ lookup table; rows of training tokens are updated, never-seen tokens keep init values (or `<UNK>`).
- **Positional embeddings**: self-attention is order-blind (permuting input permutes output the same way), so $x_i = E_{tok}(t_i) + E_{pos}(i)$; the notebook learns them (others use sinusoidal or relative positions).
- **Self-attention**: $q_i = x_i W_Q$, $k_i = x_i W_K$, $v_i = x_i W_V$; scores $a_{ij} = q_i \cdot k_j / \sqrt{d_k}$, softmax over $j$ gives $\alpha_{ij}$, output $o_i = \sum_j \alpha_{ij} v_j$. Bidirectional (encoder, no causal mask) - for NER both left and right context matter.
- **Multi-head**: $h$ parallel heads with their own $W_Q, W_K, W_V$, concatenated and projected, $\text{MultiHead}(X) = \text{Concat}(head_1, \ldots, head_h) W_O$; the notebook uses 2 heads.
- **FFN**: $\text{FFN}(x) = W_2\,\text{GELU}(W_1 x + b_1) + b_2$, shared across positions, hidden size 32.
- **Residual + layer norm**: $x \leftarrow \text{LayerNorm}(x + \text{Sublayer}(x))$ (post-norm, `norm_first=False`).
- **Padding**: two mechanisms that are both necessary - a padding mask so attention ignores pad positions, and `ignore_index=-100` so the loss does. The mask controls attention; the ignore index controls the loss.
- **Head + loss**: $z_i = h_i W_{ner} + b_{ner} \in \mathbb{R}^{5}$, `nn.CrossEntropyLoss(ignore_index=-100)` (fused softmax + cross-entropy, numerically stable; PyTorch averages over valid positions - the sum-vs-mean only rescales gradients, not their direction).

Full training passes `model.parameters()` to Adam, so embeddings, attention, FFN, layer norms and the head **all** update. 250 epochs on six sentences drive the loss to near zero - that is **memorization by design**, the goal is showing mechanics, not generalization. Exercise 4 instead sets `requires_grad=False` everywhere except the head (head-only training - the same idea as section 6, now on a Transformer; a real BERT setup would start from pretrained weights and choose full vs parameter-efficient fine-tuning).

## 9. What Changes During Training

| Component | Full training | Head-only training |
|---|---|---|
| Token embeddings | trainable | frozen |
| Position embeddings | trainable | frozen |
| Attention matrices | trainable | frozen |
| FFN matrices | trainable | frozen |
| Layer norm | trainable | frozen |
| NER head | trainable | trainable |

The notebook starts from **random** values; a real system starts from a pretrained encoder - same loss and gradient flow, different starting point, and a pretrained start usually generalizes much better with small labeled data.

## 10. Limits and Correct Interpretation

**The notebook shows:** spans -> BIO, BIO -> decoding, entity-level metrics, the linear head, end-to-end training of a small Transformer, subword alignment in principle.

**It does not show:** a real dataset, train/dev/test split, pretrained encoder, hyperparameter tuning, significance, baselines, production performance. The tiny final training loss is memorization, not a good NER system.

**To add in a real project:** annotation guideline; train/dev/test split; a pretrained (e.g. biomedical) encoder; entity-level P/R/F1 on test; error analysis over the four error types; a dictionary/EntityRuler baseline; alignment checks; fixed seeds and repeated runs.

## Key Points

1. NER predicts a **span and a type** - both are necessary.
2. BIO turns spans into token labels; the decoder rebuilds spans and must repair invalid sequences.
3. Evaluation must be entity-level and strict: correct type + wrong boundary = not a correct entity.
4. Token accuracy can hide NER errors (most tokens are `O`).
5. A shared linear head maps each contextual vector to label scores; softmax -> probabilities, cross-entropy -> loss.
6. Frozen encoder + trainable head is a valid, didactic setup.
7. Subword alignment must obey one rule in both training and evaluation.
8. A Transformer gives one contextual vector per token; attention mixes left and right context.
9. Full training updates everything; head-only updates just the head.
10. The toy model memorizes six sentences - mechanics, not generalization.

## Related

- [[Annotation and Evaluation of Information Extraction]] - parent theory note on span annotation, BIO/BILOU, agreement, P/R/F1, exact vs relaxed matching
- [[Transformers and Contextual Representations]] - Lecture 9: the architecture this toy Transformer instantiates
- [[Fine-Tuning and Domain Adaptation]] - Lecture 10: head-only vs full fine-tuning on a pretrained encoder
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L11_Supervised_NER]] (AY 2026/2027)