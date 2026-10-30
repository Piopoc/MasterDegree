# Adapting Pretrained Language Models via Fine-Tuning and Domain Adaptation

## Executive Summary

Adapting pretrained language models for specialized applications is a critical phase in modern machine learning workflows. While **pretraining** exposes a model to broad statistical regularities across large text corpora, downstream implementation typically requires specialized behavior. **Fine-tuning** addresses this gap by continuing gradient-based learning from a pretrained starting point.

Unlike **prompting**—which alters context while leaving model parameters untouched—fine-tuning updates model weights using targeted training data. There is no single fine-tuning algorithm; rather, any setup is defined by three fundamental choices: the target **data**, the optimization **objective**, and the scope of **updated parameters**.

Key methodologies include adding task-specific prediction heads to contextual token representations (e.g., utilizing `[CLS]` tokens for whole-sequence classification or token vectors for sequence labeling like Named Entity Recognition), performing **instruction tuning**, and executing **domain adaptation**. Crucially, domain adaptation, full fine-tuning, and Parameter-Efficient Fine-Tuning (**PEFT**) are complementary strategies that can be combined to optimize language models for specialized domains and tasks.

## 1. The Language Model Adaptation Lifecycle

The process of deploying specialized language models follows a three-stage lifecycle:

```
[ PRETRAIN ]  --->  [ FINE-TUNE ]  --->  [ INFERENCE ]
```

1. **Pretrain:** The model learns general language patterns and broad statistical regularities from large, unlabeled text corpora (e.g., via self-supervised objectives like Masked Language Modeling).
2. **Fine-tune:** The model undergoes continued gradient-based learning starting from the pretrained weights to adapt to a specific domain, task, or instruction-following behavior.
3. **Inference:** The adapted model is executed on new operational examples.

### Prompting vs. Fine-Tuning

Although a pretrained model possesses vast linguistic knowledge, it may lack awareness of specific target tasks. To steer model behavior, practitioners utilize two main paradigms:

|   |   |   |
|---|---|---|
|Metric / Dimension|Prompting|Fine-Tuning|
|**Mechanism**|Changes the input context/instructions|Uses targeted training data|
|**Model Weights**|Remaining entirely **unchanged**|**Updated** (all, head-only, or PEFT subset)|
|**Primary Use Case**|Steering behavior dynamically via context|Embedding specialized behavior directly into parameters|

## 2. Structural Dimensions of Fine-Tuning Setups

Because fine-tuning is not a single uniform algorithm, every adaptation setup is configured by answering three key questions:

```
                       ┌── 1. What Data? (Domain text, Labeled task data, Instruction-response pairs)
                       │
FINE-TUNING SETUP ─────┼── 2. What Objective? (Language-model loss, Classification loss, Token-label loss)
                       │
                       └── 3. Which Parameters? (All weights, Head only, Small PEFT parameters)
```

### 1. What Data?

- **Domain Text:** Unlabeled text specific to an industry or subject area (used for continued pretraining/domain adaptation).
- **Labeled Task Data:** Supervised examples paired with ground-truth outputs for task execution.
- **Instruction–Response Pairs:** Datasets structured specifically to teach the model to follow user directives (used in **Instruction Tuning**).

### 2. What Objective?

- **Language-Model Loss:** Used in self-supervised settings or instruction tuning to predict contextually accurate tokens.
- **Classification Loss:** Applied to whole-sequence categorical predictions.
- **Token-Label Loss:** Applied per token for sequence labeling tasks.

### 3. Which Parameters?

- **All Weights:** Full fine-tuning, where every parameter across the entire network is updated.
- **Head Only:** Freezing the pretrained network and updating only an added task-specific output layer.
- **Small PEFT Parameters:** Updating only a minimal set of Parameter-Efficient Fine-Tuning parameters (e.g., LoRA), keeping the main model weights frozen.

## 3. Architecture Adaptation & Task Fine-Tuning

### Contextual Representations & Sequence Modeling

Pretraining frameworks such as Masked Language Modeling (MLM) yield **one contextual representation per input token** derived from a pretrained Transformer backbone. These contextual vectors (h_i) act as rich feature extractions for downstream supervised architectures.

- **Whole-Sequence Classification:** Utilizes specialized aggregate token representations, specifically the `**[CLS]**` token vector, to classify entire text sequences.
- **Sequence Labeling:** Utilizes individual contextual token vectors (h_i) directly to generate token-level predictions.

### Example: Named Entity Recognition (NER)

For token-level sequence labeling tasks like Named Entity Recognition, the adaptation architecture attaches a task-specific classification head directly to the token representations:

\text{Contextual Vector } (h_i) \longrightarrow \text{Linear Layer} \longrightarrow \{\text{B-GENE}, \text{I-GENE}, \text{O}, \dots\}

In this setup, the fine-tuning process calculates a token-label supervised loss and backpropagates gradients to update either the linear task head alone, the entire network, or a small PEFT parameter set.

## 4. Integrated Adaptation Strategies

Rather than operating as mutually exclusive choices, modern model adaptation workflows combine multiple approaches sequentially:

```
[ Pretrained Base Model ]
           │
           ▼
[ Domain Adaptation ]  <-- Continued pretraining on unlabeled domain text
           │
           ▼
[ Fine-Tuning / PEFT ] <-- Task fine-tuning (Full, Head-Only, or LoRA/PEFT)
           │
           ▼
[ Adapted Task Model ]
```

- **Domain Adaptation:** Before performing task-specific fine-tuning, a model can undergo continued pretraining on domain-specific, unlabeled text to adapt its core language model representations to specialized nomenclature and structures.
- **Strategy Unification:** Domain adaptation, full fine-tuning, and Parameter-Efficient Fine-Tuning (PEFT/LoRA) can be combined modularly to balance computational efficiency with domain precision and task accuracy.

## Key References

- **Formulations for Sequence Labeling & NER:** Jurafsky & Martin, _Speech and Language Processing_, Chapter 9.
- **Domain Adaptive Pretraining Strategy:** Gururangan et al., 2020.
- **Parameter-Efficient Fine-Tuning (LoRA):** Hu et al., 2022.