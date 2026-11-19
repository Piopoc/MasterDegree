---
type: course-note
date: 2026-11-11
tags:
  - course
  - advanced-text-analytics
  - biomedical-ner
  - entity-normalization
  - terminology
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L13]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 13) on **biomedical NER and concept normalization: the split between mention detection and concept mapping, biomedical terminologies and benchmark corpora, the four-stage normalization pipeline, and the rules -> lexical -> neural progression of normalization methods**. Created 2026-11-11 from `raw/ATA_L13.md`. Use it when a question involves what NER output lacks that normalization supplies (a canonical concept ID), which terminologies define the concept space per domain (MeSH, SNOMED CT, UMLS CUIs, RxNorm, HPO...), the candidate generation vs candidate ranking distinction, the trade-off table across dictionary / TF-IDF char n-gram / bi-encoder / cross-encoder methods, hybrid pipelines (DNorm, TaggerOne, scispacy), or the three-level evaluation protocol (span P/R/F1 for NER, Accuracy@1 / Recall@k / MRR on gold mentions, strict end-to-end correctness). Prerequisites: character n-gram TF-IDF from [[TF-IDF and Static Embeddings]]; the NER machinery from [[Advanced Named Entity Recognition]] (Lecture 12); the mention-vs-concept framing of [[Knowledge Resources for Text Analytics]] (Lecture 2). The runnable walkthroughs (toy terminology, exact lookup, TF-IDF retriever, optional bi-/cross-encoder extension) live in [[Biomedical Named Entity Recognition and Concept Normalization - Notebook]].

## 1. Mention Detection vs. Concept Normalization

Biomedical text mining splits into two connected operations:

- **Named Entity Recognition (NER)** identifies text spans and assigns broad semantic types. It marks _"breast carcinoma"_ as a `DISEASE` entity - but gives no unique database code.
- **Entity Normalization** (a.k.a. entity linking / concept mapping) converts extracted mentions into canonical concept identifiers from standard terminologies - e.g. `C0006142` (UMLS CUI) or a MeSH ID.

```text
Text: "Patient shows signs of chronic breast carcinoma."

1. Mention detection (NER):  [breast carcinoma] -> DISEASE
2. Concept normalization:    [breast carcinoma] -> Concept ID: C0006142
```

| Processing stage | Primary input | Operational goal | Output artifact |
|---|---|---|---|
| **NER** | unstructured text tokens | detect span boundaries and classify broad entity types | labeled character offsets + category label (e.g. `DISEASE`) |
| **Normalization** | extracted mentions + surrounding context | map mentions to standardized knowledge base entries | unique concept identifiers (UMLS CUI, MeSH ID) |

Separating the two allows modular system design: boundary-detection errors can be evaluated independently from concept-mapping errors.

Two properties of biomedical language make normalization hard (and motivate the whole lecture):

1. **Many surface forms, one concept** - _"breast cancer"_, _"breast carcinoma"_, _"cancer of the breast"_, _"malignant neoplasm of breast"_ all denote the same concept.
2. **One surface form, many concepts** - the abbreviation _"AS"_ denotes both _Ankylosing Spondylitis_ and _Aortic Stenosis_; only context decides.

## 2. Structured Domain Knowledge and Benchmark Datasets

### 2.1 Standard Biomedical Terminologies

Terminologies are reference **dictionaries**, not training datasets: they supply target concepts, preferred names, unique identifiers, and synonyms needed for mapping.

| Domain | Resources |
|---|---|
| Diseases | MeSH, OMIM, SNOMED CT, UMLS |
| Chemicals and drugs | RxNorm, MeSH, DrugBank |
| Genes and proteins | NCBI Gene, UniProtKB |
| Phenotypes | Human Phenotype Ontology (HPO) |

**UMLS** is the meta-thesaurus: it integrates multiple source vocabularies into one framework keyed by Concept Unique Identifiers (CUIs).

### 2.2 Standard Benchmark Datasets

Corpora with both mention-boundary annotations and concept-ID tags:

1. **NCBI Disease Corpus** - 793 PubMed abstracts fully annotated for disease mentions, mapped to MeSH or OMIM identifiers; standard benchmark for disease NER + normalization.
2. **BC5CDR Corpus** - built for the BioCreative V Chemical-Disease Relation task: 1,500 PubMed abstracts annotated for chemical mentions, disease mentions, and chemical-induced disease relations.

## 3. The Four-Stage Normalization Pipeline

```text
[Mention Detection] -> [Candidate Generation] -> [Concept Ranking] -> [Identifier Assignment]
```

1. **Mention detection** - identify the target entity span and type from the source text (the NER stage).
2. **Candidate generation** - retrieve a subset of top-k plausible concepts from the terminology, shrinking hundreds of thousands of entries to a manageable list (e.g. k = 10 to 100).
3. **Concept ranking** - score mention vs candidates; advanced models add surrounding sentence context to re-rank.
4. **Identifier assignment** - select the highest-scoring candidate and emit the final canonical identifier.

The architectural consequence: **candidate generation and candidate ranking are different problems**. If the gold concept never enters the top-k, no reranker can recover it - which is why candidate Recall@k is measured separately (Section 5).

## 4. Methodological Progression: Rules, Lexical Search, Neural Encoders

### 4.1 Rule-Based and Dictionary Normalization

String-matching tables map preferred names and known synonyms directly to concept IDs. Surface strings are preprocessed first: lowercasing, punctuation stripping, whitespace normalization.

A common implementation is spaCy's `EntityRuler`, which matches exact surface patterns and attaches pre-assigned concept IDs during tokenization.

- **Advantages**: very fast, fully transparent, high precision on exact matches.
- **Limitations**: fails completely on unseen morphological variants, spelling errors, ambiguous abbreviations.

### 4.2 Lexical Retrieval via TF-IDF Character N-Grams

To tolerate spelling variation and partial matches, each concept is flattened into one document and vectorized with TF-IDF over **character n-grams** (typically n = 3 to 5):

$$\text{concept\_text} = \text{preferred\_name} + \text{" "} + \text{join}(\text{synonyms})$$

Similarity is cosine between the mention vector $\mathbf{v}_q$ and the candidate concept vector $\mathbf{v}_C$:

$$\text{score}(q, C) = \cos(\mathbf{v}_q, \mathbf{v}_C) = \frac{\mathbf{v}_q \cdot \mathbf{v}_C}{\|\mathbf{v}_q\| \|\mathbf{v}_C\|}$$

Character n-grams enable fuzzy matching, absorbing minor typos and morphological changes.

### 4.3 Neural Architectures: Bi-Encoders and Cross-Encoders

- **Bi-encoders** (SapBERT, BioSyn) - pretrained transformer backbones (e.g. PubMedBERT) encode mention and concept description **independently** into dense embeddings; candidate retrieval is fast approximate nearest-neighbor search. Concept vectors can be precomputed once.
- **Cross-encoders** - concatenate mention + surrounding context + full candidate description and process the pair jointly with full transformer attention.

Cross-encoders resolve context-dependent ambiguity: _"AS"_ means _Ankylosing Spondylitis_ next to _"back pain"_ but _Aortic Stenosis_ next to _"aortic valve"_.

### 4.4 Trade-Off Summary

| Method class | Technical approach | Cost | Unseen variants | Context awareness |
|---|---|---|---|---|
| Exact dictionary | table lookup / spaCy `EntityRuler` | very low (O(1)) | none (exact match only) | none |
| Lexical retrieval | char n-gram TF-IDF + cosine | low-medium | high (typos, morphology) | limited to lexical context |
| Dense bi-encoder | pretrained transformer (SapBERT) embeddings | medium (pre-indexed) | high (semantic similarity) | moderate |
| Cross-encoder reranker | full joint transformer attention | high | very high | full context resolution |

### 4.5 Hybrid Normalization Pipelines

Industrial systems combine stages: rule-based lookup handles high-confidence exact matches first; on failure a fast bi-encoder or TF-IDF retriever generates candidates and a cross-encoder re-ranks the top ones. **DNorm, TaggerOne, and scispacy** follow this hybrid design pattern.

## 5. Quantitative Evaluation and Diagnostic Metrics

Three separate evaluation levels - never collapse them into one number.

### 5.1 NER Metrics

Standard span-based precision, recall, F1:

$$P_{\text{NER}} = \frac{|\text{Correct Entity Spans}|}{|\text{Predicted Entity Spans}|} \qquad R_{\text{NER}} = \frac{|\text{Correct Entity Spans}|}{|\text{Gold Entity Spans}|}$$

$$F1_{\text{NER}} = \frac{2 \cdot P_{\text{NER}} \cdot R_{\text{NER}}}{P_{\text{NER}} + R_{\text{NER}}}$$

### 5.2 Normalization Metrics (on Gold Mentions)

Given human-annotated mention spans, three diagnostic metrics:

1. **Accuracy@1** - % of mentions whose top-ranked concept matches the gold concept (final ranking quality).
2. **Candidate Recall@k** - % of mentions where the gold ID appears within the top-k retrieved candidates (candidate generation quality).
3. **Mean Reciprocal Rank (MRR)** - ranking quality across candidate lists:

$$\text{MRR} = \frac{1}{|M|} \sum_{i=1}^{|M|} \frac{1}{\text{rank}_i}$$

where $|M|$ is the number of evaluation mentions and $\text{rank}_i$ the position of the ground-truth concept for mention $i$.

### 5.3 End-to-End Evaluation

A prediction counts as correct only if all three conditions hold simultaneously:

```text
End-to-End Correctness = (Correct Span Boundary) AND (Correct Entity Type) AND (Correct Concept ID)
```

Diagnostic error analysis then separates failures into: **boundary errors** (e.g. extracting _"breast"_ instead of _"breast carcinoma"_), **type assignment errors**, **candidate generation omissions**, and **context disambiguation errors**.

## 6. Take-Home Messages

1. NER detects spans and broad types; normalization maps mentions to canonical IDs - two tasks, two error spaces, two metric families.
2. Terminologies (UMLS/MeSH/SNOMED CT/...) define the concept space; benchmark corpora (NCBI Disease, BC5CDR) provide joint span + ID gold annotations.
3. Normalization is a four-stage pipeline: detection -> candidate generation -> ranking -> assignment.
4. Method choice is a trade-off ladder: exact rules are fast but brittle; char n-gram TF-IDF tolerates morphology; bi-encoders add scalable semantic retrieval; cross-encoders add full context at high cost.
5. Hybrid systems are the practical norm (DNorm, TaggerOne, scispacy): exact match first, retrieve then rerank.
6. Evaluate the stages separately - Accuracy@1, Recall@k, MRR on gold mentions - then report strict end-to-end correctness requiring span + type + ID all correct.

## 7. Suggested Review Questions

1. What does NER output lack, and which artifact does normalization add? Why does separating them help system design?
2. Which two properties of biomedical language make normalization hard, and which example in the lecture demonstrates each?
3. What are the four pipeline stages, and what is the role of candidate generation in shrinking the search space?
4. Why can a cross-encoder disambiguate _"AS"_ when exact lookup and a bi-encoder struggle?
5. Why must candidate Recall@k be read before Accuracy@1 in a two-stage architecture?
6. What three conditions must simultaneously hold for an end-to-end prediction to count as correct?

## Related

- [[Advanced Text Analytics]] - course hub
- [[Biomedical Named Entity Recognition and Concept Normalization - Notebook]] - companion walkthroughs for both L13 notebooks
- [[Knowledge Resources for Text Analytics]] - Lecture 2: terminologies, controlled vocabularies, ontologies; mention recognition vs concept normalization
- [[TF-IDF and Static Embeddings]] - Lecture 4: the TF-IDF and cosine machinery reused here in character n-gram form
- [[Advanced Named Entity Recognition]] - Lecture 12: the span-based NER models that feed the normalization pipeline
- [[Automatic Term Extraction]] - Lecture 15: the same candidate generation vs ranking split, applied to discovering domain terms

## Sources

- [[ATA_L13]] (AY 2026/2027)
