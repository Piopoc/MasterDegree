---
type: course-note
date: 2026-11-11
tags:
  - course
  - advanced-text-analytics
  - notebook
  - biomedical-ner
  - entity-normalization
  - tf-idf
  - sentence-transformers
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L13_Notebooks]]"
ai-first: true
---

## For Future Agent

Notebook companion for Lecture 13: cell-by-cell walkthrough of `ATA_L13_Biomedical_NER_and_Entity_Normalization_companion.ipynb` (main - a complete normalization pipeline built from scratch on a six-concept toy terminology: exact synonym lookup, char n-gram TF-IDF retrieval, contextual disambiguation, Accuracy@1 / Recall@3 evaluation) and `ATA_L13_Optional_BiEncoder_CrossEncoder_Normalization.ipynb` (optional neural extension: `sentence-transformers` bi-encoder retrieval + cross-encoder reranking), extracted 2026-11-11 from `raw/ATA_L13_Notebooks.md`. Central caveats: the neural models are small **general-purpose** ones (`all-MiniLM-L6-v2`, `ms-marco-MiniLM-L-6-v2`) chosen to show architecture, never biomedical normalization quality; candidate Recall@k must be read **before** Accuracy@1 because reranking cannot recover an unretrieved concept; raw bi-encoder and cross-encoder scores are on different scales and must not be compared directly; the ambiguous abbreviation `AS` is deliberately excluded from the `EntityRuler` baseline. Theory note: [[Biomedical Named Entity Recognition and Concept Normalization]].

## 1. Position in the Course

Both notebooks open with the same contrast:

```text
NER:          "breast carcinoma" -> DISEASE
Normalization: "breast carcinoma" -> C_BREAST_CANCER
```

NER detects a span and assigns a type; normalization (entity linking / concept mapping) maps that span to a stable identifier in a terminology (UMLS, SNOMED CT, or here a toy teaching terminology). The two hard properties of biomedical language recur throughout:

- **Many surface forms, one concept**: "breast cancer", "breast carcinoma", "cancer of the breast", "malignant neoplasm of breast" -> `C_BREAST_CANCER`.
- **One surface form, many concepts**: `AS` is a synonym of both *ankylosing spondylitis* and *aortic stenosis* - only context decides.

The main notebook exposes the mechanics of seven things: terminology-backed normalization, exact synonym lookup, candidate generation, TF-IDF ranking, contextual disambiguation, evaluation with Accuracy@1 and Recall@k, and a simple end-to-end NER -> normalization pipeline.

## 2. Notebook 1 - Main Companion

Requirements: `re`, `unicodedata`, `numpy`, `pandas`, `matplotlib`, scikit-learn (`TfidfVectorizer`, `cosine_similarity`). No model downloads.

### 2.1 A Tiny Biomedical Terminology

Six concepts as a list of dictionaries; each concept has a stable ID, preferred name, synonyms, entity type, optional description:

| concept_id | type | preferred | synonyms | description |
|---|---|---|---|---|
| `C_BREAST_CANCER` | DISEASE | breast cancer | breast carcinoma; cancer of the breast; malignant neoplasm of breast | malignant disease affecting breast tissue |
| `C_LUNG_CANCER` | DISEASE | lung cancer | lung carcinoma; pulmonary carcinoma; malignant neoplasm of lung | malignant disease affecting the lung |
| `C_DIABETES` | DISEASE | diabetes mellitus | diabetes; diabetes mellitus | metabolic disease characterized by elevated blood glucose |
| `C_ANKYLOSING_SPONDYLITIS` | DISEASE | ankylosing spondylitis | AS | chronic inflammatory disease causing back pain ... |
| `C_AORTIC_STENOSIS` | DISEASE | aortic stenosis | AS | cardiac disease caused by narrowing of the aortic valve |
| `C_ASPIRIN` | CHEMICAL | aspirin | acetylsalicylic acid | drug used for pain fever inflammation ... |

The two `AS` entries are the heart of the ambiguity example used in both notebooks. Mention-to-concept is many-to-many in both directions (many synonyms -> one concept; one abbreviation -> many concepts).

### 2.2 String Normalization

`normalize_string(text)`: NFKC Unicode normalization -> lowercase/strip -> replace every run of non-alphanumeric characters with a single space -> collapse internal whitespace. So `"Breast Carcinoma"`, `"breast-carcinoma"`, `"  Breast   carcinoma  "` all become `"breast carcinoma"`. The notebook stresses this is an **engineering choice**, not a universal preprocessing ritual.

### 2.3 Exact Synonym Dictionary and Baseline

The lookup table maps each normalized surface form to a **list** of concept IDs (`setdefault(key, []).append(...)`) - lists, not single IDs, because `AS` maps to two concepts.

```python
def exact_normalize(mention):
    key = normalize_string(mention)
    return surface_to_concepts.get(key, [])
```

Testing `"breast carcinoma"`, `"cancer of the breast"`, `"aspirin"`, `"AS"`, `"breast tumour"` already yields the three canonical outcomes:

```text
1 candidate  -> easy exact normalization
>1 candidate -> ambiguity
0 candidates -> unseen surface form
```

Even the rule-based baseline therefore exposes the two central problems: **candidate generation** (which concepts could match?) and **candidate ranking / disambiguation** (which one is correct?).

### 2.4 Optional spaCy `EntityRuler` Baseline

Patterns built from preferred names + synonyms with `label` = entity type and `id` = concept ID; the ambiguous `AS` is **deliberately skipped** because one deterministic rule cannot decide its concept. One pass over *"A patient with breast carcinoma was treated with aspirin."* prints mention, type, and concept ID together.

```text
It can:   mention -> entity type -> concept ID  (known patterns)
It cannot: unseen synonyms, spelling variation not in the rules,
           ambiguous abbreviations, semantically similar but lexically different mentions
```

### 2.5 TF-IDF Candidate Retrieval

Each concept is flattened into one searchable document (preferred + synonyms + description, space-joined); the concept stays structured knowledge - the string is only the retrieval representation.

```python
vectorizer = TfidfVectorizer(analyzer="char_wb", ngram_range=(3, 5), lowercase=True)
concept_matrix = vectorizer.fit_transform(concept_documents)
```

`char_wb` builds character n-grams of length 3-5 inside word boundaries (the Lecture 07 idea: char TF-IDF is robust to small spelling/morphological differences). `retrieve_concepts(query, top_k=3, entity_type=None)` vectorizes the query, computes cosine similarity against every concept row, optionally filters by entity type (mimicking the type constraint inherited from NER), sorts descending, and keeps `top_k`.

Flow: mention -> TF-IDF vector -> cosine vs all concepts -> ranked concepts.

### 2.6 Retrieval Behaviors on the Four Hard Cases

- **Straightforward synonyms** (`"breast carcinoma"`, `"pulmonary carcinoma"`, `"acetylsalicylic acid"`, `"diabetes"`) - correct concept ranks first.
- **Unseen surface form** `"breast tumour"` - exact lookup returns `[]`; TF-IDF still ranks `C_BREAST_CANCER` highly via overlapping character fragments ("breast", tumor/tumour-like pieces, "cancer", "malignant"). Caveat: lexical similarity alone is not genuine semantic disambiguation.
- **Ambiguity** `AS` - exact lookup returns both concepts. Fix: add context to the query. Context A (spine: _"chronic inflammatory back pain ... sacroiliac joints"_) favors `C_ANKYLOSING_SPONDYLITIS`; context B (cardiac: _"narrowing of the aortic valve"_) favors `C_AORTIC_STENOSIS`.
- **The interpretation**: identical mention, different context -> different ranking. A practical normalizer uses `mention + sentence context + concept description`, not the mention string alone: `surface form alone ≠ concept`.

### 2.7 The Two-Stage Normalizer

`normalize_entity(mention, context, entity_type, top_k)`:

1. try `exact_normalize` - if exactly one candidate, return immediately (`method="exact"`);
2. otherwise build `query = f"{mention} {context}"` and fall back to TF-IDF retrieval (`method="retrieval"`).

The result dict records which path was taken - useful for debugging.

### 2.8 Candidate Generation vs Ranking

If the correct concept is not among retrieved candidates, no reranker can recover it. A two-stage architecture must therefore measure **candidate Recall@k** before evaluating top-1 predictions. The idea reappears in the optional notebook with bi-encoder + cross-encoder.

### 2.9 Evaluation Set and Metrics

`EVAL_DATA`: six items with `mention`, `context`, `type`, `gold`:

| mention | gold | notes |
|---|---|---|
| breast carcinoma | `C_BREAST_CANCER` | exact synonym |
| pulmonary carcinoma | `C_LUNG_CANCER` | exact synonym |
| diabetes | `C_DIABETES` | short form |
| AS + context A | `C_ANKYLOSING_SPONDYLITIS` | ambiguity, spine |
| AS + context B | `C_AORTIC_STENOSIS` | ambiguity, cardiac |
| acetylsalicylic acid | `C_ASPIRIN` | CHEMICAL, drug synonym |

`evaluate_retrieval(data, k=3)` runs `mention + context` through `retrieve_concepts` with type filtering and prints two aggregates:

- **Accuracy@1** - is the first-ranked concept correct? (final ranking)
- **Recall@3** - is the gold concept among the top three? (candidate generation diagnostic)

A follow-up loop inspects ranking errors; the notebook lists the error categories: candidate missing, wrong top-ranked concept, abbreviation ambiguity, insufficient context, terminology missing the required synonym, NIL (concept absent from the terminology).

### 2.10 End-to-End: NER Output to Normalization

NER detections are hardcoded (the NER component is assumed to exist):

```python
detected_entities = [
    {"text": "breast carcinoma", "type": "DISEASE"},
    {"text": "aspirin", "type": "CHEMICAL"},
    {"text": "AS", "type": "DISEASE"},
]
```

Each entity goes through `normalize_entity` with the full sentence as context; the result table shows mention, type, concept ID, method (`exact`/`retrieval`). Key point: **if NER misses a mention, the downstream normalizer never sees it** - this is why end-to-end evaluation is stricter than normalization on gold mentions.

### 2.11 Bridge to Neural Retrieval

The TF-IDF retriever is the transparent version of the bi-encoder pattern: mention+context -> encoder -> vector; concept name+synonyms+description -> encoder -> vector; cosine -> rank. Advantage: concept vectors are precomputed once. An optional **commented-out, not executed** `SentenceTransformer` pseudocode block shows the pattern; the notebook warns that a real biomedical system must select and evaluate the embedding model for terminology normalization, not pick one for convenience. The cross-encoder reranking stage (joint `mention + context + candidate` scoring) is introduced conceptually here and implemented in the optional notebook.

## 3. Notebook 2 - Optional Neural Normalization

Goal: **architectural understanding, not state-of-the-art biomedical normalization**. Pipeline:

```text
NER output -> mention + context -> BI-ENCODER -> top-k candidates -> CROSS-ENCODER -> best concept
```

Requirements: `numpy`, `pandas`, `sentence-transformers` (install cell present but commented out).

### 3.1 Structured Concept Documents

Unlike the main notebook's flat concatenation, fields are labeled so the encoder sees their role:

```python
f"Preferred name: {concept['preferred']}. Synonyms: {synonyms}. Description: {concept['description']}."
```

### 3.2 Bi-Encoder Retrieval

```python
bi_encoder = SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
concept_embeddings = bi_encoder.encode(concept_documents,
    convert_to_tensor=True, normalize_embeddings=True)
```

- Query format: `f"Mention: {mention}. Context: {context}"`.
- `normalize_embeddings=True` makes the vectors unit length, so cosine similarity is a plain dot product.
- Concept embeddings are **precomputed** because they do not depend on the query.
- `cos_sim(query_embedding, concept_embeddings)` scores every concept; a DataFrame lists them sorted descending.
- `bi_encoder_retrieve(mention, context, top_k=3, entity_type=None)` mirrors `retrieve_concepts` from the main notebook (type filter, descending sort, `top_k` cut) and keeps an `index` column back into `TERMINOLOGY` so the reranker can rebuild pairs.

Tests: `"breast carcinoma"`, `"acetylsalicylic acid"`, and the unseen `"breast tumour"` (handled via semantic similarity rather than string overlap). For `AS` with both contexts the notebook notes the bi-encoder limitation: query and concept are encoded **independently** - they never attend to each other.

### 3.3 Cross-Encoder Reranking

```python
cross_encoder = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
```

One forward pass per pair `[mention + context] + [candidate concept] -> compatibility score` - more expensive than the bi-encoder, richer interactions. The flow:

1. retrieve candidates with `bi_encoder_retrieve` (e.g. `AS` + spine context, `top_k=3`, `entity_type="DISEASE"`);
2. build `pairs = [(query_text, concept_document(concept)), ...]`;
3. `cross_scores = cross_encoder.predict(pairs)`, attach, re-sort descending.

`retrieve_and_rerank(mention, context, entity_type, top_k)` wraps all three steps and returns early on an empty candidate list.

```text
BI-ENCODER:    query -> vector ; concept -> vector ; compare vectors
CROSS-ENCODER: query + concept -> joint Transformer -> score
```

Comparisons are run on `"breast tumour"`, `AS` + spine context, `AS` + cardiac context - both tables displayed so the reader can see whether the cross-encoder changed the order.

### 3.4 Why Retrieve Then Rerank

A 500,000-concept terminology would require ~500,000 cross-encoder forward passes per mention. The bi-encoder precomputes all concept vectors and retrieves only top-k; the cross-encoder reranks perhaps 10, 50, or 100 of them. And **candidate Recall@k still gates everything**: if the gold concept is not in the bi-encoder top-k, the cross-encoder cannot recover it.

### 3.5 Evaluation of the Neural Pipeline

Same six-item `EVAL_DATA`. Per item: run `bi_encoder_retrieve` (top-3) -> record `bi_encoder_top1`, `gold_in_top3`; run `retrieve_and_rerank` (top-3) -> record `cross_encoder_top1`, `final_correct`. Two aggregates separate the stages:

- **Bi-encoder candidate Recall@3** (retrieval quality)
- **Final cross-encoder Accuracy@1** (reranking quality)

Two traps flagged by the notebook:

- **Do not compare raw scores** - bi-encoder cosine and cross-encoder logits are different scales; each is only meaningful for ranking within its own stage.
- **What training would look like** - the notebook runs inference only; a trained bi-encoder would maximize `similarity(query, gold)` over negatives (contrastive learning), a trained cross-encoder would score `(query, correct)` above `(query, incorrect)` (ranking losses).

### 3.6 Pedagogical Limitation

`all-MiniLM-L6-v2` and `ms-marco-MiniLM-L-6-v2` are small, easy-to-run, general-purpose models - **not** biomedical normalization models. A serious biomedical system must validate model choice and fine-tuning on biomedical normalization corpora.

## 4. How the Two Notebooks Fit Together

| Main notebook | Optional notebook |
|---|---|
| Terminology + exact lookup | same terminology |
| TF-IDF candidate retrieval | bi-encoder retrieval |
| (not implemented) | cross-encoder reranking |
| Accuracy@1 / Recall@3 | Recall@3 + Accuracy@1 after rerank |

Read the main notebook first: it defines the terminology, the evaluation protocol, and the two-stage mindset with a transparent lexical retriever. The optional notebook keeps the same data and metrics and swaps only retrieval/reranking for neural models.

**Suggested reading order:** run the main notebook top to bottom and inspect every printed table -> check exact-dictionary outcomes for `AS` and `breast tumour` -> confirm context A vs B flips the `AS` ranking -> optional notebook sections 0-10 (bi-encoder only) -> sections 11-16 (cross-encoder reranking) -> evaluation cells, comparing Recall@3 against final Accuracy@1.

## 5. Key Points

1. NER and normalization are different tasks: spans+types vs canonical concept IDs.
2. Surface form and concept are many-to-many in both directions - context is what breaks ties.
3. Even a toy dictionary baseline surfaces both central problems: candidate generation and candidate ranking/disambiguation.
4. Normalization is naturally formulated as retrieval + ranking; exact match first, TF-IDF or bi-encoder for candidates, cross-encoder for reranking.
5. Candidate Recall@k diagnoses generation, Accuracy@1 diagnoses final ranking - report both, per stage.
6. Precomputed concept embeddings make bi-encoders scalable; joint encoding makes cross-encoders accurate but expensive - hence retrieve-then-rerank.
7. Bi-encoder and cross-encoder raw scores live on different scales; never compare them directly.
8. The demo models illustrate architecture, never biomedical quality - model selection must be validated on biomedical corpora.

## Related

- [[Biomedical Named Entity Recognition and Concept Normalization]] - Lecture 13 theory note: pipeline, terminologies, method trade-off table, metrics
- [[TF-IDF and Static Embeddings - Notebook]] - Lecture 4: the TF-IDF mechanics reused here in character n-gram form
- [[Advanced Named Entity Recognition]] - Lecture 12: span-based NER feeding the normalization stage
- [[Knowledge Resources for Text Analytics]] - Lecture 2: terminologies, ontologies, mention recognition vs concept normalization
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L13_Notebooks]] (AY 2026/2027)
- NCBI Disease Corpus: https://www.ncbi.nlm.nih.gov/research/bionlp/Data/disease/
- UMLS: https://www.nlm.nih.gov/research/umls/
- BioCreative V CDR: https://biocreative.bioinformatics.udel.edu/tasks/biocreative-v/track-3-cdr/
- Leaman et al. (2013). *DNorm: disease name normalization with pairwise learning to rank* - cited via the ATA_L13_Notebooks source above
- Leaman & Lu (2016). *TaggerOne: joint named entity recognition and normalization with semi-Markov models* - cited via the ATA_L13_Notebooks source above
