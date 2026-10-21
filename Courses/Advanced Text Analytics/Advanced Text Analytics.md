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

> **Reassignment (2026-10-09):** the [[LOTUS Semantic Operators]] note originally created under this course was moved to [[Graph Databases]] at the user's request. Cross-course link kept: the linguistic layer of [[NLP Linguistic Structures and Annotation]] still underpins semantic predicates.

## Raw Sources for This Course

- [[NLP]] - "Linguistic Structures and Computational Annotations in NLP" (English)
- [[ATA_L05]] - "Knowledge Resources for Text Analytics" (Lecture 5, English)
- [[ATA_L06]] - "Annotation and Evaluation of Information Extraction" (Lecture 6, English)
- [[ATA_L07]] - "From features to representations" - TF-IDF and static embeddings (Lecture 7, English)
- [[ATA_L08]] - "RNNs and LSTMs for text analytics" - lecture slides + NumPy notebook companion (Lecture 8, English)
- [[RNN & LSTM]] - concise briefing on RNNs/LSTMs in text analytics, with Jurafsky & Martin figure references (Lecture 8, English)

*Note: [[LOTUS]] (the LOTUS paper summary) now serves [[Graph Databases]].*
