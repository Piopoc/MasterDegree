---
type: course-note
date: 2026-10-29
tags:
  - course
  - advanced-text-analytics
  - notebook
  - fine-tuning
  - bert
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATAL10_notebook]]"
ai-first: true
---

## For Future Agent

Notebook companion for [[Fine-Tuning and Domain Adaptation]] (Lecture 10), created 2026-10-29 from the full lecture-10 notebook recap. It covers both companion notebooks: `L10a` (fine-tuning mechanics - head-only vs full fine-tuning, token-classification heads, instruction tuning, continued pretraining, PEFT/LoRA) and `L10b` (BERT, masked language modeling, contextual embeddings, task-specific heads). Both use only NumPy, pandas and matplotlib with tiny artificial models - deliberately *not* reproducing the scale of a real Transformer: their purpose is to make the **mechanics and data flow** of pretraining and fine-tuning visible at the level of individual parameter updates. Central caveats: every printed number comes from toy data or random weights; **which parameters receive gradients** is the real definition of a fine-tuning strategy (the forward pass is unchanged).

## Part A - `L10a`: Fine-tuning Minimal Companion

### A.0 Stated goals

The notebook makes four ideas concrete:

1. Fine-tuning **starts from parameters that have already been learned** (pretraining).
2. We can train **only a new task head**, or also update the pretrained parameters.
3. **Instruction tuning** is supervised training on *instruction -> desired response* examples.
4. **PEFT methods such as LoRA** adapt a model while keeping the large pretrained matrices frozen.

### A.1 The setup: a toy pretrained encoder

The model is a two-layer linear "encoder + head":

```
h = x · W_enc          (pretrained encoder, 4 → 3 dims)
z = h · W_head         (task head, 3 → 2 classes)
p = softmax(z)
```

- `W_enc_pretrained` is a 4x3 matrix sampled from a normal distribution - we *pretend* it was learned during pretraining.
- The downstream task is a tiny binary classification over 6 artificial rows of `X` with labels `y = [1,1,1,0,0,0]`.
- Utility functions: numerically stable **softmax** (max-shift), **cross-entropy** (with `1e-12` epsilon), and **accuracy** (argmax comparison).

The central question the notebook poses:

> Which parameters should we update with the new labeled data?

### A.2 Strategy A - freeze the encoder, train only the head

```
pretrained encoder:  frozen
classification head: trainable
```

- The encoder is used as a **feature extractor**: `H = X @ W_enc` is computed, but only `W_head` receives gradient updates.
- Training loop: 150 epochs, `learning_rate = 0.5`, manual gradient of softmax + cross-entropy:
  - `grad_logits = (probs - one_hot(y)) / N`
  - `grad_W_head = Hᵀ @ grad_logits`
  - `W_head -= lr · grad_W_head`
- **Verification cell:** the norm of `W_enc - W_enc_before` must be `0`, while the norm of `W_head - W_head_before` must be positive.

This is the key result: *the system was adapted to the task without changing the pretrained encoder at all.*

A loss curve is plotted (epochs vs. cross-entropy), and the reader is invited to vary epochs/learning rate to observe the effect.

### A.3 Strategy B - full fine-tuning

Now **both** `W_enc` and `W_head` receive gradients:

- The forward pass is identical - the difference is purely **which parameters receive gradients**.
- Backpropagation is done by hand through the head into the encoder:
  - `grad_H = grad_logits @ W_headᵀ`
  - `grad_W_enc = Xᵀ @ grad_H`
- Uses `learning_rate = 0.25` (lower than Strategy A - full updates are more aggressive), also 150 epochs.
- Verification: both encoder and head now show a non-zero parameter change.

A comparison table summarizes:

| strategy | encoder updated | head updated | final loss |
|---|---|---|---|
| head only | False | True | ... |
| full fine-tuning | True | True | ... |

**Important caveat (explicitly stated):** full fine-tuning is *not* automatically better. The trade-off is:

```
more trainable parameters
        ↕
more adaptation capacity
        ↕
more memory / storage / risk of overfitting
```

### A.4 Task fine-tuning for token classification (NER bridge)

The same idea applied **at every token position**:

- A pretrained Transformer produces contextual vectors $H = [h_1 \dots h_n]$.
- A linear head maps each vector to scores over NER labels: $z_i = h_i \cdot W_{\mathrm{NER}}$.
- Toy example: tokens `BRCA1 / mutations / increase / risk` with labels `{B-GENE, O}`; `W_ner` is 3x2, producing a per-token probability table.
- Fine-tuning would use gold BIO labels to update `W_ner` (and possibly the Transformer too, depending on the strategy). This is explicitly framed as the bridge to the next lecture on NER.

### A.5 Instruction tuning (SFT)

Training examples pair an instruction with a desired response:

```
Instruction: Translate to Italian: The small dog
Response:    Il piccolo cane
```

- Implemented as a list of `{"instruction": ..., "response": ...}` dicts.
- A `format_example()` function serializes each pair into a template: `<instruction> ... </instruction> <response> ... </response>` - the exact template is an implementation choice.
- For a causal LLM, training still uses the **language-modeling / next-token objective**; the difference from generic pretraining is that examples are deliberately constructed as instruction-response pairs. The response is the supervised target.

**Critical distinction taught here:**

```
Prompting:           change the context → weights remain fixed
Instruction tuning:  instruction-response pairs as training data
                     → gradient descent changes trainable parameters
```

### A.6 Continued pretraining for domain adaptation

Fine-tuning can also mean continuing the *original* pretraining objective on a more specific corpus:

```
general language model
        ↓ continue LM training on legal judgments
legal-domain-adapted model
```

- **No task labels** are required at this stage - supervision comes from the text itself.

A table contrasts the three adaptation types:

| adaptation | example data | typical objective |
|---|---|---|
| continued pretraining | unlabeled biomedical/legal text | language-model loss |
| task fine-tuning | text + class/BIO labels | task-specific supervised loss |
| instruction tuning | instruction + desired response | language-model loss |

### A.7 PEFT and LoRA

Full fine-tuning: $W' = W + \Delta W$.

LoRA freezes $W$ and learns a **low-rank update**:

```
ΔW = B·A        →     W' = W + B·A
```

Only the small matrices `A` and `B` are trainable.

- **Parameter-count example:** for a 768x768 projection with rank 8:
  - full update = 768 x 768 = **589,824** parameters
  - LoRA = 8 x 768 + 768 x 8 = **12,288** parameters ≈ **2.08 %** of the full update
- **Caveat:** this does *not* mean LoRA always matches unconstrained full fine-tuning - we intentionally constrain the update to a low-rank form in exchange for parameter efficiency.
- **Correctness check:** `W@x + B@(A@x)` is verified to be numerically equal to `(W + B@A)@x` via `np.allclose` - i.e., the merged-weight formulation is algebraically equivalent at inference time (this is how LoRA is deployed with zero added latency).

### A.8 Summary table and take-home messages

| method | data | what is learned | weights |
|---|---|---|---|
| continued pretraining | domain text | domain-adapted LM | often many/all |
| task fine-tuning | labeled task examples | task decision function | head only or encoder + head |
| instruction tuning | instruction-response pairs | instruction-following behavior | many/all or PEFT |
| LoRA / PEFT | task/domain/instruction data | small parameter-efficient update | small added parameter set |

**Take-homes:**

1. Fine-tuning means **continuing learning from a pretrained starting point**.
2. The phrase alone is underspecified - always ask: **What data? What objective? Which parameters?**
3. **Task fine-tuning** -> labeled examples + task-specific loss.
4. **Instruction tuning / SFT** -> instruction-response pairs, trained with the LM objective.
5. **Continued pretraining** -> domain text, no task labels necessarily needed.
6. **Full vs. head-only** = update many pretrained parameters vs. freeze the encoder.
7. **LoRA** -> freeze large matrices, learn small low-rank corrections.
8. The choice is an **engineering decision** based on task, available supervision, compute, and domain knowledge.

## Part B - `L10b`: BERT, MLM and Fine-Tuning Companion (Part I)

### B.0 Stated goals and progression

The notebook follows the 45-minute L10 slide deck along this path:

```
Transformer architecture
        ↓
bidirectional Transformer encoder
        ↓
BERT
        ↓
masked language modeling
        ↓
contextual embeddings
        ↓
fine-tuning
        ↓
task-specific heads
        ↓
classification / NER
```

Goals: understand **why BERT is bidirectional**, how **MLM** works, what **contextual embeddings** are, how **pretraining differs from fine-tuning**, and how contextual representations become task predictions.

### B.1 From Transformer to BERT: causal vs. bidirectional visibility

- A causal Transformer (next-token prediction, see [[Transformers and Contextual Representations]]): token *i* can see itself + previous tokens, **not** future tokens.
- BERT **removes the causal restriction**: for a fully available input sentence, every token sees both left and right context -> a **bidirectional Transformer encoder**.
- Demonstrated with binary visibility matrices (`np.tril(np.ones(...))` for causal vs. all-ones for bidirectional) over tokens `BRCA1 / mutations / increase / breast / cancer / risk`.
- Concrete reading: under causal attention, the row for `breast` cannot see `cancer`/`risk`; under bidirectional attention it can. This is especially useful for **information extraction**, where the whole sentence is already known.

### B.2 BERT as a contextual encoder

```
x₁, x₂, …, xₙ  →  bidirectional Transformer encoder  →  h₁, h₂, …, hₙ
```

- Output length = input length; each $h_i$ is a **contextual representation** of token *i*.
- Toy demo: 6 tokens x 4-dim random **input embeddings** (not yet contextual). These are analogous to the token embeddings entering a Transformer; BERT transforms them via bidirectional self-attention and stacked blocks.

### B.3 Why masked language modeling (MLM)?

Because BERT sees both sides, ordinary next-token prediction no longer works as the main objective. Instead:

```
original sentence → hide/corrupt some tokens → bidirectional encoder → predict original tokens
```

This is **self-supervised**: the original text provides the answers.

- Minimal example: `lunch was delicious` -> mask position 2 -> input `lunch was [MASK]`, target `delicious`.
- **Loss is computed only at the selected (masked) prediction positions** - not on every token:
  ```
  lunch  → no MLM loss
  was    → no MLM loss
  [MASK] → predict "delicious"
  ```

### B.4 The original BERT masking recipe (80/10/10)

1. Select ~**15%** of input tokens.
2. For each selected token:
   - **80%** -> replaced with `[MASK]`
   - **10%** -> replaced with a random other token
   - **10%** -> left unchanged
3. The target is **always the original token**.

*Rationale:* reduce the mismatch between pretraining and downstream use, since downstream inputs normally contain no `[MASK]`.

Implemented as `bert_style_mask(tokens, vocabulary, selection_prob, rng)` returning `(corrupted, selected_positions, targets)`. The classroom demo uses `selection_prob=0.45` (inflated for visibility) over an 8-token biomedical sentence with a small vocabulary - a real setup uses the actual recipe on large tokenized corpora.

Data flow:

```
corrupted sequence → BERT → vocabulary distribution at selected positions
                     → cross-entropy loss against original token
```

### B.5 Vocabulary prediction at a masked position

Pattern (repeated throughout the course):

```
representation → linear layer → scores → softmax → probabilities
```

- Toy: 6-dim hidden vector `h_masked` x `W_vocab` (6x5) over vocab `{breast, cancer, gene, risk, therapy}` -> logits -> softmax table.
- Cross-entropy for a single masked token: if the true token is `cancer`, loss = $-\log P(\text{cancer})$ (+ `1e-12` epsilon). High probability on the correct token => low loss.

### B.6 What BERT actually learns during pretraining

Key conceptual point: MLM is **not** useful because we want a `[MASK]`-filling system. Its value is that, to predict missing words, the encoder is forced to learn **useful contextual representations** - all tokens contribute through self-attention, and MLM provides the learning signal. After pretraining, we keep the encoder and apply it to **normal, unmasked text**.

### B.7 Static vs. contextual embeddings

A central course distinction:

```
Word2Vec (static):   word type                → one vector
BERT (contextual):   word occurrence in context → contextual vector
```

- `bank` gets different vectors in *investment bank* vs. *river bank*.
- Toy demo: two 4-dim vectors for the same token string in different contexts; the whole vector changes with context. Individual dimensions are **latent** - we must not interpret `dim_1 = financial meaning`, etc.

### B.8 From pretraining to fine-tuning (transfer learning)

| | Pretraining | Fine-tuning |
|---|---|---|
| Data | large unlabeled corpus | smaller labeled dataset |
| Objective | self-supervised MLM | task-specific objective |
| Result | general contextual encoder | specialized model |

> Knowledge learned during pretraining is reused for a new downstream task = **transfer learning**.

### B.9 Task-specific heads

The pretrained encoder stays the same; the **head** changes with the task:

- **Sequence classification** -> one label for the whole input.
- **Sequence labeling** -> one label per token.

#### B.9.1 Sequence classification with `[CLS]`

- BERT prepends a special `[CLS]` token; its final contextual representation $h_{[\mathrm{CLS}]}$ summarizes the sequence.
- Toy head: 6-dim `[CLS]` vector x `W_classifier` (6x3) over `{positive, neutral, negative}` -> scores -> softmax.
- Same `representation -> linear -> scores` pattern as before; the difference is **one prediction for the entire sequence**.

#### B.9.2 Sequence labeling for NER

- One prediction **per token**: BERT outputs $h_1 \dots h_n$, and the *same* token-level classifier is applied to every contextual vector.
- Toy: 6 tokens x 6-dim `H_bert`, label set `{O, B-GENE, B-DISEASE, I-DISEASE}`, `W_ner` is 6x4 -> per-token score table -> `argmax` per row gives illustrative predictions.
- Contrast with RNNs:
  ```
  RNN:  context propagated sequentially
  BERT: context integrated by bidirectional self-attention
  ```
- Modern basic pattern:
  ```
  tokens → BERT encoder → contextual token vectors → token classifier → NER labels
  ```
  The classifier is task-specific; BERT is the **shared representation machinery**.

### B.10 Fine-tuning choices: a continuum

```
Feature extraction:    freeze BERT, train only the task head
Partial adaptation:     train the head + update some upper encoder layers
Full fine-tuning:      update the head + update the whole encoder
```

- Parameter-count intuition (illustrative toy numbers): encoder = 100,000 params, head = 1,000 params -> trainable fraction ranges from ~1 % (feature extraction) to ~100 % (full fine-tuning).
- Trade-off: **more adaptation ↔ more computation and more risk of overfitting**.

### B.11 Domain adaptation vs. task adaptation

Two *distinct* operations:

```
general pretrained BERT
        ↓ continue MLM pretraining on domain text   ← domain adaptation
domain-adapted encoder
        ↓ fine-tune on labeled task data            ← task adaptation
task model
```

Biomedical NER example:
- **Domain adaptation** (unlabeled text): adapt representations to genes, proteins, diseases, biomedical syntax/terminology.
- **Task adaptation** (labeled examples): learn which spans correspond to which entity types.

These are not the same learning problem.

### B.12 What BERT does *not* solve automatically

A pretrained contextual encoder does **not** define the text-analytics problem. It does not automatically supply:

- the annotation scheme,
- the domain ontology and terminology,
- expert constraints,
- evaluation criteria,
- guaranteed correct predictions.

BERT is one powerful **source of learned representations**, to be combined with annotated data, domain resources, rules, terminologies, ontologies and expert knowledge - consistent with the course methodology.

### B.13 Positioning across the last four lectures

| Lecture | Main representation idea |
|---|---|
| L07 | static embedding: one vector per word type |
| L08 | RNN/LSTM: contextual representation via recurrence |
| L09 | Transformer: contextual representation via attention |
| L10 | BERT: pretrained bidirectional Transformer representations adapted to downstream tasks |

The architecture becomes increasingly reusable, but the downstream task still requires its own supervision, resources, and evaluation.

## Cross-Notebook Synthesis

The two notebooks are complementary: **L10b** explains *where the pretrained parameters come from* (bidirectional encoder + MLM pretraining) and *how they are consumed* (task heads, `[CLS]`, token classifiers); **L10a** zooms in on *what "fine-tuning" actually means* at the level of gradient updates (head-only vs full, instruction tuning, continued pretraining, LoRA).

Recurring patterns across both:

1. **`representation → linear layer → scores → softmax → probabilities`** - the same head pattern appears for MLM vocabulary prediction, sequence classification, and NER.
2. **Which parameters receive gradients** is the real definition of a fine-tuning strategy - the forward pass is unchanged.
3. **Data + objective + trainable parameters** are the three questions that disambiguate any "fine-tuning" claim:
   - domain text / LM loss / many weights -> continued pretraining
   - labeled examples / task loss / head (± encoder) -> task fine-tuning
   - instruction-response pairs / LM loss / many weights or PEFT -> instruction tuning
   - any of the above / low-rank `A`, `B` only -> LoRA
4. **Domain adaptation ≠ task adaptation** - they answer different questions and use different data.
5. **Efficiency ↔ capacity trade-off**: freezing, partial updates, and LoRA all reduce trainable parameters in exchange for constrained adaptation capacity (with LoRA quantified at ≈ 2 % of a 768x768 update for rank 8).
6. **Pretraining is a representation resource, not a solution** - annotation schemes, ontologies, evaluation, and expert knowledge remain the practitioner's responsibility.

## Related

- [[Fine-Tuning and Domain Adaptation]] - parent theory note (adaptation lifecycle, setup dimensions, task heads, PEFT/LoRA)
- [[Transformers and Contextual Representations]] - Lecture 9: the pretrained architecture, causal vs bidirectional masking, static vs contextual embeddings that L10b builds on
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATAL10_notebook]] (AY 2026/2027)