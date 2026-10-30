---
type: course
date: 2026-10-09
tags:
  - course
  - advanced-text-analytics
ai-first: true
---

## For Future Agent

Course index for **Advanced Text Analytics (ATA)** - the hub note linking all study notes for this course. Created 2026-10-09 when the vault's `Courses/` structure was bootstrapped from `raw/` material. Course identity details (institution, instructor) are **TBD** - no source states them yet. Related: [[Privacy Preserving Information Access]], [[Game Theory]], [[Graph Databases]].

## Course Identity

| Field | Value |
|---|---|
| Course | Advanced Text Analytics |
| Institution | TBD (not stated in sources) |
| Academic Year | 2026/2027 (assumed - same as other tracked courses; speculation) |
| Instructor | TBD (not stated in sources) |
| Status | active |

## Notes

1. [[NLP Linguistic Structures and Annotation]] - word vs. token, annotation pipeline, phrase/clause hierarchy, dependency vs. constituency grammar.
2. [[Knowledge Resources for Text Analytics]] - lists/gazetteers, terminologies, controlled vocabularies, taxonomies, thesauri, ontologies, knowledge bases/graphs; mention recognition vs concept normalization.
3. [[Annotation and Evaluation of Information Extraction]] - span annotation, BIO/BILOU, inter-annotator agreement (Cohen's kappa), P/R/F1, exact vs relaxed matching, micro vs macro averaging, relation and pipeline evaluation.
4. [[TF-IDF and Static Embeddings]] - Bag of Words limits, TF-IDF weighting and L2 normalization, cosine similarity, distributional hypothesis, hand-built dense space, Skip-gram from scratch in NumPy, association strength, nearest-neighbor caveats, FastText subwords, one-vector-per-type limit.
5. [[Recurrent Neural Networks and LSTMs]] - hidden-state recurrence, parameter sharing across time, label-score pipeline (POS/NER), vanishing/exploding gradient intuition, LSTM gates and cell state, bidirectional models, notation reference and self-check.
6. [[Transformers and Contextual Representations]] - Transformer architecture, attention as weighted sum, Query/Key/Value, scaled dot-product attention, multi-head attention, positional encodings, causal vs bidirectional masking, Transformer block composition, stacking blocks, language modeling, autoregressive generation, self-supervised pretraining, decoder-only/encoder-only/encoder-decoder configurations, and the NER pipeline.
7. [[Fine-Tuning and Domain Adaptation]] - adaptation lifecycle (pretrain/fine-tune/inference), prompting vs fine-tuning, the three setup dimensions (data/objective/parameters), task heads on contextual representations ([CLS] classification, token-level NER), domain adaptation, PEFT/LoRA, integrated strategy pipeline.

### Notebook Walkthroughs

Extracted 2026-10-28 into `notebooks/` (tag `notebook`); theory stays in the parent notes above.

- [[Knowledge Resources for Text Analytics - Notebook]] - spaCy `PhraseMatcher` demos, `broader_concepts` traversal, KB triple store.
- [[TF-IDF and Static Embeddings - Notebook]] - six-document corpus, worked TF-IDF numbers, cosine results, sentence experiment, hand-built dense space, NumPy Skip-gram run and nearest neighbors.
- [[Recurrent Neural Networks and LSTMs - Notebook]] - embedding lookup, `rnn_step` dry run, label-score projection, vanishing toy curve, LSTM gates dry run, bidirectional loop.
- [[Transformers and Contextual Representations - Notebook]] - dot-product similarity, `np.triu` causal mask, random positional encoding.
- [[Fine-Tuning and Domain Adaptation - Notebook]] - L10a fine-tuning mechanics (head-only vs full dry runs, NER head bridge, instruction-tuning format, continued pretraining, LoRA low-rank math) and L10b BERT/MLM companion (causal vs bidirectional visibility matrices, 80/10/10 masking recipe, MLM vocabulary prediction, `[CLS]` classification and NER heads, domain vs task adaptation).

> **Reassignment (2026-10-09):** the [[LOTUS Semantic Operators]] note originally created under this course was moved to [[Graph Databases]] at the user's request. Cross-course link kept: the linguistic layer of [[NLP Linguistic Structures and Annotation]] still underpins semantic predicates.
