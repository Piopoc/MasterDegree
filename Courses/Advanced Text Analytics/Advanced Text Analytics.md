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
8. [[Advanced Named Entity Recognition]] - token classification vs span classification, candidate enumeration and the 13-dim span vector, structural NONE imbalance, GLiNER label-conditioning, UniversalNER distillation, hard zero-shot (ZeroNER), threshold tuning, six-paradigm selection table.
9. [[Biomedical Named Entity Recognition and Concept Normalization]] - NER vs normalization, biomedical terminologies (UMLS/MeSH/SNOMED CT) and benchmark corpora (NCBI Disease, BC5CDR), four-stage pipeline (detection, candidate generation, ranking, assignment), rules -> char n-gram TF-IDF -> bi-/cross-encoder progression with hybrid systems, evaluation on three levels (span P/R/F1, Accuracy@1/Recall@k/MRR, strict end-to-end).
10. [[Legal Information Extraction and Knowledge Graphs]] - dual nature of legal text, three legal object classes (entities, nested references/provisions, contextual roles), hybrid rules + gazetteer + learned-model architecture, SKOS labels/hierarchy/mappings/integrity conditions, SKOS vs OWL, ELI/ECLI/CELEX/EuroVoc, text-to-RDF-to-SPARQL pipeline, LOMO, source reconciliation table.
11. [[Automatic Term Extraction]] - term vs entity vs keyword, termhood vs unithood, candidate generation (recall) vs ranking (precision), POS patterns as filters, frequency/TF-IDF/domain specificity/PMI, nested terms with C-value and NC-value (worked examples), resource-aware hybrid ranking, supervised/LLM paradigms, Systems A-F experiment matrix for DETECH 2027.

### Notebook Walkthroughs

Extracted 2026-10-28 into `notebooks/` (tag `notebook`); theory stays in the parent notes above. L11 companions added 2026-11-10 (theory companions - no L11 parent note exists yet, so they are self-contained). L12 parent note + companion added 2026-11-10. L13 parent note + companion added 2026-11-11. L14 parent note + companion added 2026-11-17. L15 parent note + companion added 2026-11-19.

- [[Knowledge Resources for Text Analytics - Notebook]] - spaCy `PhraseMatcher` demos, `broader_concepts` traversal, KB triple store.
- [[TF-IDF and Static Embeddings - Notebook]] - six-document corpus, worked TF-IDF numbers, cosine results, sentence experiment, hand-built dense space, NumPy Skip-gram run and nearest neighbors.
- [[Recurrent Neural Networks and LSTMs - Notebook]] - embedding lookup, `rnn_step` dry run, label-score projection, vanishing toy curve, LSTM gates dry run, bidirectional loop.
- [[Transformers and Contextual Representations - Notebook]] - dot-product similarity, `np.triu` causal mask, random positional encoding.
- [[Fine-Tuning and Domain Adaptation - Notebook]] - L10a fine-tuning mechanics (head-only vs full dry runs, NER head bridge, instruction-tuning format, continued pretraining, LoRA low-rank math) and L10b BERT/MLM companion (causal vs bidirectional visibility matrices, 80/10/10 masking recipe, MLM vocabulary prediction, `[CLS]` classification and NER heads, domain vs task adaptation).
- [[Supervised Named Entity Recognition - Notebook]] - L11a theory companion (span-to-BIO encoding, BIO decoder with repair rule, strict entity-level P/R/F1, shared linear head with manual gradients, subword alignment, toy Transformer full vs head-only training).
- [[spaCy Toy NER - Notebook]] - L11b theory companion (blank-pipeline training workflow, character-offset annotations, `Example`/`nlp.initialize`/`nlp.update`, transition-based BILUO `EntityRecognizer` vs BIO classifier, fitting vs generalization, four error types).
- [[Advanced Named Entity Recognition - Notebook]] - L12 theory companion for both notebooks (spaCy `SpanCategorizer` workflow with `doc.spans["sc"]` and nested spans, candidate enumeration and 13-dim span vectors, toy logistic-regression span classifier, cosine label matching and description-driven labels, optional commented GLiNER demo, threshold sweep, generalist vs generative LLM).
- [[Biomedical Named Entity Recognition and Concept Normalization - Notebook]] - L13 companion for both notebooks (toy six-concept terminology, exact synonym dictionary and `EntityRuler` baseline with `AS` excluded, char n-gram TF-IDF retrieval with type filter, two-stage `normalize_entity`, Accuracy@1/Recall@3 protocol, optional `sentence-transformers` bi-encoder retrieval + cross-encoder reranking with stage-separated metrics).
- [[Legal Information Extraction and Knowledge Graphs - Notebook]] - L14 companion for both notebooks (Part I: ECLI/case/act/provision regexes with named-group parsing and the trailing-`\b` backtracking subtlety, offset-based output schema, `EntityRuler` precedence, merge policy, roles as party x proceeding relations, nested refs in `doc.spans` and `alignment_mode`, lossy ELI-like URIs, strict eval + error taxonomy; Part II: ELI/CELEX registry linking, ELI FRBR ontology, local SKOS hierarchy, empty GLiNER `legal topic`, flawed coverage-based topic normalizer and NIL problem, `rdflib` graph with extracted vs hard-coded elements, SPARQL property paths, CELLAR/EuroVoc, LOMO).
- [[Automatic Term Extraction - Notebook]] - L15 companion (GDPR-style toy corpus + confusable reference corpus, 20 gold types, boundary-stop n-gram generator, frequency/mean TF-IDF/smoothed log-ratio specificity/bigram PMI/C-value with unigram patch, head-noun proxy, MinMax weighted hybrid, P/R/F1@k and k=30 ablation, TODO 2 failure-mode table, optional spaCy/embedding/LLM extensions; caveats: PMI unused in hybrid, head list and seeds leak the gold set, junk n-grams dilute the C-value discount).

> **Reassignment (2026-10-09):** the [[LOTUS Semantic Operators]] note originally created under this course was moved to [[Graph Databases]] at the user's request. Cross-course link kept: the linguistic layer of [[NLP Linguistic Structures and Annotation]] still underpins semantic predicates.
