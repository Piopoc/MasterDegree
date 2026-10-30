---
type: course-note
date: 2026-10-28
tags:
  - course
  - advanced-text-analytics
  - fine-tuning
  - peft
  - domain-adaptation
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L10]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 10) on **adapting pretrained language models: fine-tuning, domain adaptation, and parameter-efficient fine-tuning (PEFT)**. Created 2026-10-28 from `raw/ATA_L10.md`. Use it when a question involves the pretrain -> fine-tune -> inference lifecycle, prompting versus fine-tuning, the three configuration dimensions of any fine-tuning setup (data, objective, updated parameters), task heads over contextual token representations ([CLS] for whole-sequence classification, per-token vectors for sequence labeling), continued pretraining on domain text, or how domain adaptation, full fine-tuning, and PEFT/LoRA combine into a single pipeline. Prerequisite: [[Transformers and Contextual Representations]] covers the pretrained models, contextual representations, and instruction tuning/alignment treatment that this note builds on. The concrete notebook walkthroughs (head-only vs full fine-tuning dry runs, the instruction-tuning format, LoRA parameter counts and merged-weight check, BERT visibility matrices, the 80/10/10 masking recipe, `[CLS]` and NER heads) live in [[Fine-Tuning and Domain Adaptation - Notebook]].

## 1. Why Adaptation Is Needed

Pretraining exposes a model to broad statistical regularities across large unlabeled text corpora, but downstream deployment typically requires specialized behavior: a particular domain, a particular task, a particular output format. **Fine-tuning** closes this gap by continuing gradient-based learning from the pretrained starting point, using targeted training data.

## 2. The Adaptation Lifecycle

Deploying a specialized language model follows a three-stage lifecycle:

```
[ PRETRAIN ]  --->  [ FINE-TUNE ]  --->  [ INFERENCE ]
```

1. **Pretrain** - the model learns general language patterns and broad statistical regularities from large, unlabeled text corpora via self-supervised objectives (e.g. Masked Language Modeling).
2. **Fine-tune** - continued gradient-based learning starting from the pretrained weights, adapting the model to a specific domain, task, or instruction-following behavior.
3. **Inference** - the adapted model is executed on new operational examples.

## 3. Prompting versus Fine-Tuning

A pretrained model possesses vast linguistic knowledge but may lack awareness of the specific target task. Practitioners steer behavior through two paradigms:

| Dimension | Prompting | Fine-Tuning |
|---|---|---|
| Mechanism | changes the input context/instructions | uses targeted training data |
| Model weights | unchanged | updated (all, head-only, or PEFT subset) |
| Primary use case | steering behavior dynamically via context | embedding specialized behavior directly into parameters |

Prompting is the in-context specification interface studied in Lecture 9; fine-tuning is the subject of this lecture: the behavior is written into the weights themselves.

## 4. Three Dimensions of Every Fine-Tuning Setup

There is no single fine-tuning algorithm. Every adaptation setup is defined by answering three questions:

```
                       +-- 1. What data? (domain text, labeled task data, instruction-response pairs)
                       |
FINE-TUNING SETUP -----+-- 2. What objective? (language-model loss, classification loss, token-label loss)
                       |
                       +-- 3. Which parameters? (all weights, head only, small PEFT parameters)
```

### 4.1 What Data?

- **Domain text:** unlabeled text specific to an industry or subject area (used for continued pretraining / domain adaptation).
- **Labeled task data:** supervised examples paired with ground-truth outputs for task execution.
- **Instruction-response pairs:** datasets structured to teach the model to follow user directives (used in **instruction tuning** - the alignment treatment lives in [[Transformers and Contextual Representations]]).

### 4.2 What Objective?

- **Language-model loss:** used in self-supervised settings and instruction tuning, to predict contextually accurate tokens.
- **Classification loss:** applied to whole-sequence categorical predictions.
- **Token-label loss:** applied per token for sequence labeling tasks.

### 4.3 Which Parameters?

- **All weights:** full fine-tuning - every parameter across the entire network is updated.
- **Head only:** the pretrained network is frozen and only an added task-specific output layer is updated.
- **Small PEFT parameters:** only a minimal set of Parameter-Efficient Fine-Tuning parameters is updated (e.g. LoRA), keeping the main model weights frozen.

## 5. Task Heads on Contextual Representations

Pretraining frameworks such as Masked Language Modeling yield **one contextual representation per input token** from the pretrained Transformer backbone. These contextual vectors $h_i$ act as ready-made feature extractors for downstream supervised architectures; what varies is where the task head reads them:

- **Whole-sequence classification:** uses the specialized aggregate token representation - the `**[CLS]**` token vector - to classify entire text sequences.
- **Sequence labeling:** uses the individual contextual token vectors $h_i$ directly, one prediction per token.

### 5.1 Example: Named Entity Recognition

For a token-level sequence labeling task such as NER, a task-specific classification head attaches directly to the token representations:

$$
h_i \longrightarrow \text{Linear Layer} \longrightarrow \{\text{B-GENE},\ \text{I-GENE},\ \text{O},\ \dots\}
$$

The fine-tuning process computes a token-label supervised loss (the BIO-style labels and their evaluation are covered in [[Annotation and Evaluation of Information Extraction]]) and backpropagates gradients to update either the linear task head alone, the entire network, or a small PEFT parameter set - the third dimension from Section 4.3. The NER pipeline itself (tokens -> contextual vectors -> classifier -> labels) is the closing point of Lecture 9; this lecture adds the training story: how the classifier and/or the backbone get updated.

## 6. Domain Adaptation

Before any task-specific fine-tuning, a model can undergo **continued pretraining on domain-specific, unlabeled text**: the same self-supervised language-model objective, but on corpora from the target industry or subject area. The goal is to adapt the core language model representations to specialized nomenclature and structures - terminology, phrasing, factual regularities - that generic pretraining underrepresents.

Data and objective in this stage: *domain text* + *language-model loss* (dimension 1 and 2 in Section 4), with the full backbone as the parameter scope.

## 7. Integrated Adaptation Strategies

Domain adaptation, full fine-tuning, and PEFT are **not mutually exclusive choices** - modern adaptation workflows combine them sequentially:

```
[ Pretrained Base Model ]
           |
           v
[ Domain Adaptation ]   <-- continued pretraining on unlabeled domain text
           |
           v
[ Fine-Tuning / PEFT ]  <-- task fine-tuning (full, head-only, or LoRA/PEFT)
           |
           v
[ Adapted Task Model ]
```

- **Strategy unification:** the stages can be combined modularly to balance computational efficiency with domain precision and task accuracy.
- Instruction tuning (Section 4.1) is the same machinery pointed at instruction-response pairs instead of domain text - the pipeline treatment lives with the Lecture 9 note.

## 8. Take-Home Messages

1. Fine-tuning continues training from the pretrained weights on targeted data; prompting only changes the context - weights stay frozen.
2. Every fine-tuning setup is defined by three choices: what data, what objective, which parameters.
3. Parameter scope comes in three flavors: full fine-tuning, head-only, and PEFT (e.g. LoRA) - trading adaptation capacity against compute and storage cost.
4. Contextual token representations from the pretrained backbone are the input to task heads: the [CLS] vector for whole-sequence classification, per-token vectors for sequence labeling (NER).
5. Domain adaptation is continued pretraining on unlabeled domain text (domain text + language-model loss), placed before task fine-tuning.
6. Domain adaptation, full fine-tuning, and PEFT are complementary, combinable stages - not competing alternatives.

## 9. Suggested Review Questions

1. What are the three stages of the adaptation lifecycle, and what happens in each?
2. How do prompting and fine-tuning differ in mechanism, in what changes in the model, and in typical use case?
3. Name the three configuration dimensions of a fine-tuning setup and the options within each.
4. Which representation feeds a whole-sequence classifier, and which feeds a token-level labeler such as NER?
5. What data and what objective does domain adaptation use? Where does it sit in the pipeline?
6. Why are domain adaptation, full fine-tuning, and PEFT not mutually exclusive?

## Related

- [[Advanced Text Analytics]] - course hub
- [[Transformers and Contextual Representations]] - Lecture 9: the pretrained models, contextual token representations, and the instruction tuning/alignment treatment that this lecture's adaptation machinery extends
- [[Annotation and Evaluation of Information Extraction]] - the BIO-style token labels produced by the NER task head and how the resulting pipeline is evaluated

## Sources

- [[ATA_L10]] (AY 2026/2027)
- Jurafsky, D., & Martin, J. H. (2026). *Speech and Language Processing*, 3rd ed., Chapter 9 (sequence labeling formulations) - cited via the ATA_L10 source above (AY 2026/2027)
- Gururangan, T., et al. (2020). *Don't Stop Pretraining: Adapt Language Models to Domains and Tasks* (domain adaptive pretraining) - cited via the ATA_L10 source above
- Hu, E. J., et al. (2022). *LoRA: Low-Rank Adaptation of Large Language Models* (PEFT) - cited via the ATA_L10 source above
