# Lecture 13: Biomedical NER and Entity Normalization

## Detailed guide to the two notebooks

This document explains the content of the two Python notebooks in the `L13` folder:

1. `ATA_L13_Biomedical_NER_and_Entity_Normalization_companion.ipynb` (main notebook)
2. `ATA_L13_Optional_BiEncoder_CrossEncoder_Normalization.ipynb` (optional neural extension)

Both belong to the *Advanced Text Analytics* course, Lecture 13. The main notebook builds a complete normalization pipeline from scratch using a small hand-made terminology, exact lookup, and TF-IDF retrieval. The optional notebook takes the last ideas of the main one (neural retrieval and reranking) and turns them into working code with `sentence-transformers`.

---

## The central distinction: NER vs normalization

Both notebooks open with the same contrast:

```text
NER:
"breast carcinoma" → DISEASE

Normalization:
"breast carcinoma" → C_BREAST_CANCER
```

NER (named entity recognition) detects a span of text and assigns it an entity type. Normalization (also called entity linking or concept mapping) maps that span to a stable identifier inside a terminology such as UMLS, SNOMED CT, or (in these notebooks) a small toy terminology built for teaching.

Two properties of biomedical language make normalization hard:

- **Many surface forms, one concept.** "breast cancer", "breast carcinoma", "cancer of the breast", and "malignant neoplasm of breast" all denote the same concept (`C_BREAST_CANCER`).
- **One surface form, many concepts.** The abbreviation `AS` is a synonym of both *ankylosing spondylitis* and *aortic stenosis*. Only context can decide which one is meant.

The main notebook is deliberately small and transparent. Its goal is to expose the mechanics of seven things:

1. terminology-backed normalization
2. exact synonym lookup
3. candidate generation
4. TF-IDF ranking
5. contextual disambiguation
6. evaluation with Accuracy@1 and Recall@k
7. a simple end-to-end NER to normalization pipeline

---

# Notebook 1: main companion

## 0. Setup

Imports: `re`, `unicodedata`, `numpy`, `pandas`, `matplotlib`, and from scikit-learn the `TfidfVectorizer` plus `cosine_similarity`. Nothing else is required; the notebook runs on a plain Python environment with pandas and scikit-learn.

## 1. A tiny biomedical terminology

A concept in a terminology normally has at least:

- a stable identifier
- a preferred name
- one or more synonyms
- an entity type
- optionally, a textual description

The notebook defines six concepts as a list of dictionaries. The identifiers are toy identifiers created only for teaching:

| concept_id | type | preferred | synonyms | description |
|---|---|---|---|---|
| `C_BREAST_CANCER` | DISEASE | breast cancer | breast carcinoma; cancer of the breast; malignant neoplasm of breast | malignant disease affecting breast tissue |
| `C_LUNG_CANCER` | DISEASE | lung cancer | lung carcinoma; pulmonary carcinoma; malignant neoplasm of lung | malignant disease affecting the lung |
| `C_DIABETES` | DISEASE | diabetes mellitus | diabetes; diabetes mellitus | metabolic disease characterized by elevated blood glucose |
| `C_ANKYLOSING_SPONDYLITIS` | DISEASE | ankylosing spondylitis | AS | chronic inflammatory disease causing back pain and affecting the spine and sacroiliac joints |
| `C_AORTIC_STENOSIS` | DISEASE | aortic stenosis | AS | cardiac disease caused by narrowing of the aortic valve |
| `C_ASPIRIN` | CHEMICAL | aspirin | acetylsalicylic acid | drug used for pain fever inflammation and antiplatelet therapy |

The table is displayed with pandas. The two `AS` entries are the heart of the ambiguity example used throughout both notebooks.

### Mention is not concept

The notebook states the two directions of the many-to-many relation explicitly:

```text
breast cancer
breast carcinoma
cancer of the breast
          ↓
C_BREAST_CANCER
```

and

```text
AS → ankylosing spondylitis
AS → aortic stenosis
```

The second case requires context.

## 2. Normalize strings

`normalize_string(text)` standardizes a surface form before dictionary lookup:

```python
def normalize_string(text):
    text = unicodedata.normalize("NFKC", text)
    text = text.lower().strip()
    text = re.sub(r"[^a-z0-9]+", " ", text)
    text = re.sub(r"\s+", " ", text)
    return text.strip()
```

Steps in order: NFKC Unicode normalization, lowercasing and stripping, replacement of every run of non-alphanumeric characters with a single space, then collapsing internal whitespace. So `"Breast Carcinoma"`, `"breast-carcinoma"`, and `"  Breast   carcinoma  "` all become `"breast carcinoma"`.

The notebook notes this is an engineering choice, not a universal preprocessing ritual.

## 3. Build an exact synonym dictionary

The lookup table maps each normalized surface form to a list of concept identifiers:

```python
surface_to_concepts = {}

for concept in TERMINOLOGY:
    names = [concept["preferred"]] + concept["synonyms"]
    for name in names:
        key = normalize_string(name)
        surface_to_concepts.setdefault(key, [])
        surface_to_concepts[key].append(concept["concept_id"])
```

Values are lists, not single identifiers, because an ambiguous abbreviation such as `AS` occurs as a synonym of multiple concepts. The cell prints the whole table so you can inspect it.

## 4. Exact dictionary normalization

The simplest possible normalizer:

```python
def exact_normalize(mention):
    key = normalize_string(mention)
    return surface_to_concepts.get(key, [])
```

Test mentions: `"breast carcinoma"`, `"cancer of the breast"`, `"aspirin"`, `"AS"`, `"breast tumour"`.

The dictionary baseline already produces three distinct outcomes:

```text
1 candidate  → easy exact normalization
>1 candidate → ambiguity
0 candidates → unseen surface form
```

Even a rule-based system therefore exposes the two central normalization problems: candidate generation (which concepts could match?) and candidate ranking / disambiguation (which one is correct?).

## 5. Optional spaCy baseline: mention detection + concept ID

This section shows how spaCy's `EntityRuler` can attach a concept ID directly to a detected span:

```python
import spacy

nlp = spacy.blank("en")
ruler = nlp.add_pipe("entity_ruler")

patterns = []
for concept in TERMINOLOGY:
    for surface in [concept["preferred"]] + concept["synonyms"]:
        if surface == "AS":
            continue
        patterns.append({
            "label": concept["type"],
            "pattern": surface,
            "id": concept["concept_id"],
        })

ruler.add_patterns(patterns)
```

The ambiguous abbreviation `AS` is deliberately excluded: one deterministic rule cannot decide which concept it means. On the example sentence *"A patient with breast carcinoma was treated with aspirin."* the pipeline prints mention, entity type, and concept ID in one pass.

What this baseline can and cannot do:

```text
It can produce:   mention → entity type → concept ID   (for known patterns)
It cannot handle: unseen synonyms
                  spelling variation not encoded in the rules
                  ambiguous abbreviations
                  semantically similar but lexically different mentions
```

So the notebook moves from exact lookup to retrieval and ranking.

## 6. Build one searchable text representation per concept

For lexical retrieval each concept is flattened into a single string:

```python
def concept_document(concept):
    parts = [
        concept["preferred"],
        *concept["synonyms"],
        concept["description"],
    ]
    return " ".join(parts)
```

Preferred label, synonyms, and description are concatenated with spaces. The concept stays structured knowledge; this string is only the textual representation fed to the retrieval model. A pandas table shows `concept_id`, `type`, `preferred`, and `document` for all six concepts.

## 7. TF-IDF candidate retrieval

The notebook reuses the idea from Lecture 07: character n-gram TF-IDF is a useful lexical baseline for terminology matching because it is relatively robust to small spelling and morphological differences.

```python
vectorizer = TfidfVectorizer(
    analyzer="char_wb",
    ngram_range=(3, 5),
    lowercase=True,
)

concept_matrix = vectorizer.fit_transform(concept_documents)
```

`char_wb` builds character n-grams (length 3 to 5) inside word boundaries. Each row of `concept_matrix` is one concept in TF-IDF feature space.

Given a mention the flow is:

```text
mention
  ↓
TF-IDF vector
  ↓
cosine similarity with every concept
  ↓
rank concepts
```

The retrieval function:

```python
def retrieve_concepts(query, top_k=3, entity_type=None):
    query_vector = vectorizer.transform([query])
    scores = cosine_similarity(query_vector, concept_matrix).ravel()

    rows = []
    for i, score in enumerate(scores):
        concept = TERMINOLOGY[i]
        if entity_type is not None and concept["type"] != entity_type:
            continue
        rows.append({
            "concept_id": concept["concept_id"],
            "type": concept["type"],
            "preferred": concept["preferred"],
            "score": float(score),
        })

    result = pd.DataFrame(rows)
    result = result.sort_values("score", ascending=False).head(top_k)
    return result.reset_index(drop=True)
```

Optional `entity_type` filtering restricts candidates to a type (for example only `DISEASE`), which mimics the constraint that a normalizer often knows the entity type from the NER stage.

## 8. Retrieve concepts for straightforward mentions

Four queries are run through `retrieve_concepts`: `"breast carcinoma"`, `"pulmonary carcinoma"`, `"acetylsalicylic acid"`, `"diabetes"`. All are exact synonyms stored in the terminology, so the correct concept ranks first.

## 9. An unseen surface form

`"breast tumour"` was never stored as a synonym. Exact lookup returns an empty list. TF-IDF retrieval can still rank `C_BREAST_CANCER` highly because the concept document contains overlapping character sequences ("breast", "tumour" vs "tumor"-like fragments, "cancer", "malignant").

The notebook's caveat: lexical similarity alone is not enough for genuine semantic ambiguity.

## 10. Ambiguity: the abbreviation `AS`

`exact_normalize("AS")` returns two candidates:

```text
C_ANKYLOSING_SPONDYLITIS
C_AORTIC_STENOSIS
```

The abbreviation by itself carries almost no distinguishing information. The fix is to add surrounding context to the query:

```text
CONTEXT A (spine):
"The patient with AS reported chronic inflammatory back pain affecting the sacroiliac joints."

CONTEXT B (heart):
"Severe AS caused narrowing of the aortic valve and required cardiac evaluation."
```

Running `retrieve_concepts` on each full context sentence changes the ranking: context A favors `C_ANKYLOSING_SPONDYLITIS`, context B favors `C_AORTIC_STENOSIS`.

### Important interpretation

The mention is identical (`AS`) but the surrounding context changes the ranking. This demonstrates the conceptual point:

```text
surface form alone ≠ concept
```

A practical normalizer therefore uses `mention + sentence context + concept description`, not only the mention string.

## 11. A two-stage normalizer

`normalize_entity` combines the ideas:

```python
def normalize_entity(mention, context=None, entity_type=None, top_k=3):
    exact_candidates = exact_normalize(mention)

    if len(exact_candidates) == 1:
        return {
            "method": "exact",
            "concept_id": exact_candidates[0],
            "candidates": exact_candidates,
        }

    query = mention if context is None else f"{mention} {context}"

    ranked = retrieve_concepts(query, top_k=top_k, entity_type=entity_type)

    best_id = ranked.iloc[0]["concept_id"] if len(ranked) > 0 else None

    return {
        "method": "retrieval",
        "concept_id": best_id,
        "candidates": ranked,
    }
```

Logic: if exact lookup yields exactly one concept, return it immediately (method `"exact"`). Otherwise build a query from mention plus context and fall back to TF-IDF retrieval (method `"retrieval"`). The result dict records which path was taken, which is useful for debugging.

Four examples are run: `"breast carcinoma"`, `"AS"` with context A, `"AS"` with context B, and `"breast tumour"`.

## 12. Candidate generation and candidate ranking are different problems

If the correct concept is not among the retrieved candidates, no reranker can recover it. In a two-stage architecture you must therefore measure:

```text
candidate Recall@k
```

before evaluating the final top-1 prediction. This idea reappears in the optional notebook with the bi-encoder and cross-encoder.

## 13. Build a tiny evaluation set

`EVAL_DATA` holds six items, each with `mention`, `context`, `type`, and `gold` (the gold concept ID):

| mention | gold | notes |
|---|---|---|
| breast carcinoma | `C_BREAST_CANCER` | exact synonym |
| pulmonary carcinoma | `C_LUNG_CANCER` | exact synonym |
| diabetes | `C_DIABETES` | short form |
| AS + context A | `C_ANKYLOSING_SPONDYLITIS` | ambiguity, spine context |
| AS + context B | `C_AORTIC_STENOSIS` | ambiguity, cardiac context |
| acetylsalicylic acid | `C_ASPIRIN` | CHEMICAL type, drug synonym |

## 14. Evaluate Accuracy@1 and Recall@k

```python
def evaluate_retrieval(data, k=3):
    records = []
    for item in data:
        query = f"{item['mention']} {item['context']}"
        ranked = retrieve_concepts(query, top_k=k, entity_type=item["type"])
        predicted_ids = ranked["concept_id"].tolist()
        top1 = predicted_ids[0] if predicted_ids else None
        records.append({
            "mention": item["mention"],
            "gold": item["gold"],
            "top1": top1,
            "correct_at_1": top1 == item["gold"],
            f"gold_in_top_{k}": item["gold"] in predicted_ids,
        })
    return pd.DataFrame(records)
```

Two aggregate metrics are printed:

- `Accuracy@1`: is the first-ranked concept correct?
- `Recall@3`: is the correct concept present among the first three candidates?

Recall@3 is the diagnostic metric for the candidate generation stage; Accuracy@1 measures the final ranking.

## 15. Inspect ranking errors

A loop prints every evaluation item where the top-1 prediction is wrong or the ranking is empty, together with the full ranked table. Possible error categories listed by the notebook:

- candidate missing
- wrong top-ranked concept
- abbreviation ambiguity
- insufficient context
- terminology missing the required synonym
- NIL (concept absent from the terminology)

## 16. End-to-end example: NER output to normalization

An NER component is assumed to have already produced spans with types. In the demo the detections are hardcoded:

```python
detected_entities = [
    {"text": "breast carcinoma", "type": "DISEASE"},
    {"text": "aspirin", "type": "CHEMICAL"},
    {"text": "AS", "type": "DISEASE"},
]
```

Each entity is passed to `normalize_entity` with the full sentence as context. The result is a pandas table with mention, type, concept ID, and method (`exact` or `retrieval`).

The standard pipeline illustrated:

```text
text
  ↓
NER
  ↓
mention + type
  ↓
normalization
  ↓
concept ID
```

Key point: if NER misses a mention, the downstream normalizer never sees it. This is why end-to-end evaluation is stricter than evaluating normalization on gold mentions.

## 17. From TF-IDF to neural concept retrieval

The TF-IDF retriever is deliberately transparent. A neural bi-encoder uses the same overall formulation:

```text
mention + context
      ↓ encoder
mention vector

concept name + synonyms + description
      ↓ encoder
concept vector

cosine similarity
      ↓
rank concepts
```

The architectural advantage: concept vectors can be precomputed once and reused for every query.

An optional pseudocode block (not executed, to keep the notebook free of model downloads) shows the pattern with `SentenceTransformer`:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("some-domain-appropriate-model")

concept_vectors = model.encode(concept_documents)
mention_vector = model.encode(["breast tumour in the left breast"])
scores = cosine_similarity(mention_vector, concept_vectors)
```

The notebook warns that for a real biomedical system the embedding model must be selected and evaluated for biomedical terminology normalization, not chosen only because it is convenient.

## 18. Cross-encoder reranking

The common two-stage architecture:

```text
bi-encoder / lexical retriever
        ↓
top-k candidates
        ↓
cross-encoder
        ↓
best concept
```

A cross-encoder jointly processes `mention + context + candidate concept` and learns a richer compatibility function. The trade-off is computational cost. This section is conceptual only; the optional notebook implements it.

## 19. Final comparison

| Method | Strength | Main limitation |
|---|---|---|
| Exact dictionary | fast, transparent | known surface forms only |
| EntityRuler + ID | integrates detection and IDs | rule coverage |
| TF-IDF retrieval | simple lexical generalization | weak semantic understanding |
| Bi-encoder | scalable semantic retrieval | independent encoding |
| Cross-encoder | richer mention-concept interaction | expensive |
| Hybrid system | combines strengths | more engineering complexity |

## 20. Take-home messages

1. NER and normalization are different tasks.
2. A biomedical mention may have many synonymous surface forms.
3. The same surface form may denote multiple concepts.
4. Terminologies provide the canonical concept space.
5. Normalization is naturally formulated as candidate retrieval plus ranking.
6. Rules, lexical retrieval, contextual models, and domain knowledge are complementary.
7. Evaluation should separate mention detection, candidate recall, concept ranking, and end-to-end performance.

## References and resources

- NCBI Disease Corpus: https://www.ncbi.nlm.nih.gov/research/bionlp/Data/disease/
- NCBI BioNLP tools: https://www.ncbi.nlm.nih.gov/research/bionlp/Tools/
- UMLS: https://www.nlm.nih.gov/research/umls/
- BioCreative V CDR: https://biocreative.bioinformatics.udel.edu/tasks/biocreative-v/track-3-cdr/
- Leaman et al. (2013), *DNorm: disease name normalization with pairwise learning to rank*
- Leaman & Lu (2016), *TaggerOne: joint named entity recognition and normalization with semi-Markov models*

---

# Notebook 2: optional neural normalization

## Purpose

This notebook extends the TF-IDF normalizer from the main notebook into a neural two-stage system:

```text
NER output
   ↓
mention + context
   ↓
BI-ENCODER
   ↓
top-k candidate concepts
   ↓
CROSS-ENCODER
   ↓
best normalized concept
```

The stated goal is architectural understanding, not state-of-the-art biomedical normalization.

## 0. Installation

A commented-out cell: `# %pip install -U sentence-transformers`. Run it once if the package is missing.

## 1. Imports

`numpy`, `pandas`, and from `sentence_transformers` the classes `SentenceTransformer` and `CrossEncoder` plus the helper `cos_sim`.

## 2. Tiny biomedical terminology

The same six concepts as the main notebook, with one small difference: `C_DIABETES` lists only `"diabetes"` as synonym (the main notebook also repeats `"diabetes mellitus"`). The table is shown again for completeness.

## 3. Convert each concept into text

The concept document format is more structured than in the main notebook:

```python
def concept_document(concept):
    synonyms = "; ".join(concept["synonyms"])
    return (
        f"Preferred name: {concept['preferred']}. "
        f"Synonyms: {synonyms}. "
        f"Description: {concept['description']}."
    )
```

Labeled fields (`Preferred name:`, `Synonyms:`, `Description:`) give the encoder explicit signals about the role of each part. All six documents are printed.

## 4. Load a bi-encoder

```python
bi_encoder = SentenceTransformer(
    "sentence-transformers/all-MiniLM-L6-v2"
)
```

A bi-encoder encodes query and concept independently:

```text
mention + context        concept description
       ↓                         ↓
    encoder                   encoder
       ↓                         ↓
      e_m                       e_c

             cosine(e_m, e_c)
```

Because concept vectors do not depend on the query, they can be precomputed. The notebook states the model is deliberately small and general-purpose: it shows the architecture, it is not a biomedical state-of-the-art model.

## 5. Precompute concept embeddings

```python
concept_embeddings = bi_encoder.encode(
    concept_documents,
    convert_to_tensor=True,
    normalize_embeddings=True,
)
```

`normalize_embeddings=True` produces unit vectors, so cosine similarity becomes a plain dot product. The matrix shape is printed.

## 6. Encode one mention + context

Query format:

```python
mention = "breast carcinoma"
context = "The patient was diagnosed with breast carcinoma after imaging and biopsy."
query = f"Mention: {mention}. Context: {context}"
```

The query embedding is computed with the same encoder and normalization settings.

## 7. Compare with every concept vector

```python
scores = cos_sim(query_embedding, concept_embeddings).cpu().numpy().ravel()
```

A pandas DataFrame lists every concept with its `bi_encoder_score`, sorted descending. The bi-encoder's job is candidate retrieval: place plausible concepts near the top.

## 8. Retrieval function

```python
def bi_encoder_retrieve(mention, context, top_k=3, entity_type=None):
    query = f"Mention: {mention}. Context: {context}"
    query_embedding = bi_encoder.encode(
        query, convert_to_tensor=True, normalize_embeddings=True,
    )
    similarities = cos_sim(query_embedding, concept_embeddings).cpu().numpy().ravel()
    # filter by entity_type if given, sort by score, keep top_k
    ...
```

Same shape as `retrieve_concepts` in the main notebook: optional type filter, descending sort, `top_k` cut. The DataFrame keeps an `index` column pointing back to `TERMINOLOGY`, which the reranking stage uses to rebuild pairs.

## 9. Test bi-encoder retrieval

Three tests: `"breast carcinoma"`, `"acetylsalicylic acid"`, and the unseen form `"breast tumour"`. The neural retriever handles the unseen form through semantic similarity rather than exact string overlap.

## 10. Ambiguous abbreviation: `AS`

The same two contexts as the main notebook (spine vs cardiac) are queried with the bi-encoder. The notebook notes the limitation: query and concept are still encoded independently, they do not directly attend to each other.

## 11. Load a cross-encoder

```python
cross_encoder = CrossEncoder(
    "cross-encoder/ms-marco-MiniLM-L-6-v2"
)
```

A cross-encoder receives query and one candidate jointly:

```text
[mention + context] + [candidate concept]
                ↓
          Transformer
                ↓
        compatibility score
```

One forward pass per query-candidate pair, which is more expensive than the bi-encoder. Again a small general-purpose ranking model used only to illustrate the architecture.

## 12. Retrieve candidates first

`"AS"` with the spine context is passed to `bi_encoder_retrieve` with `top_k=3` and `entity_type="DISEASE"`. This produces the candidate list that the cross-encoder will rerank.

## 13. Build query-candidate pairs

```python
query_text = f"Mention: {mention}. Context: {context}"

pairs = []
for _, row in candidates.iterrows():
    concept = TERMINOLOGY[int(row["index"])]
    pairs.append((query_text, concept_document(concept)))
```

Each pair is a tuple `(query_text, concept_document)` ready for `CrossEncoder.predict`.

## 14. Cross-encoder reranking

```python
cross_scores = cross_encoder.predict(pairs)

reranked = candidates.copy()
reranked["cross_encoder_score"] = cross_scores
reranked = reranked.sort_values("cross_encoder_score", ascending=False)
```

Architecture summary given in the notebook:

```text
BI-ENCODER
query → vector
concept → vector
compare vectors

CROSS-ENCODER
query + concept → joint Transformer → score
```

## 15. Complete retrieve + rerank function

```python
def retrieve_and_rerank(mention, context, entity_type=None, top_k=3):
    candidates = bi_encoder_retrieve(mention, context, top_k=top_k, entity_type=entity_type)
    if len(candidates) == 0:
        return candidates
    query_text = f"Mention: {mention}. Context: {context}"
    pairs = [...]
    cross_scores = cross_encoder.predict(pairs)
    candidates = candidates.copy()
    candidates["cross_encoder_score"] = cross_scores
    return candidates.sort_values("cross_encoder_score", ascending=False).reset_index(drop=True)
```

Retrieve with the bi-encoder, rerank with the cross-encoder, return the re-sorted table. The empty-candidate case returns early.

## 16. Compare retrieval and reranking

Three examples are run through both `bi_encoder_retrieve` and `retrieve_and_rerank`: `"breast tumour"`, `"AS"` with spine context, `"AS"` with cardiac context. For each example both tables are displayed so you can see whether the cross-encoder changed the order.

## 17. Why not cross-encode every concept?

If a terminology has 500,000 concepts, a cross-encoder would need roughly 500,000 forward passes for one mention. The bi-encoder precomputes all concept vectors and retrieves only the top-k candidates; the cross-encoder then reranks perhaps 10, 50, or 100 of them. This is the standard retrieve-then-rerank trade-off.

## 18. Candidate recall matters

If the gold concept is not in the bi-encoder top-k, the cross-encoder cannot recover it. Candidate Recall@k is the diagnostic metric for the retrieval stage, exactly as in the main notebook.

## 19. Tiny evaluation set

The same `EVAL_DATA` as the main notebook: six items with mention, context, type, and gold concept.

## 20. Evaluate Recall@3 and final Accuracy@1

For each item the loop:

1. runs `bi_encoder_retrieve` (top-3) and records `bi_encoder_top1` and `gold_in_top3`
2. runs `retrieve_and_rerank` (top-3) and records `cross_encoder_top1` and `final_correct`

Two aggregates are printed:

- `Bi-encoder candidate Recall@3`
- `Final cross-encoder Accuracy@1`

The pair of numbers separates retrieval quality from reranking quality.

## 21. Do not compare the raw scores directly

The bi-encoder score is cosine similarity; the cross-encoder score is the output of a different ranking model. Both are useful for ranking within their own stage, but the numeric values are not on the same scale.

## 22. What would training look like?

The notebook performs inference only. A trained bi-encoder would learn to make `similarity(query, gold concept)` larger than similarity to negative concepts. A trained cross-encoder would learn to score `(query, correct concept)` higher than `(query, incorrect concept)`. This connects normalization to ranking losses and contrastive learning.

## 23. Pedagogical limitation

Models used:

```text
sentence-transformers/all-MiniLM-L6-v2
cross-encoder/ms-marco-MiniLM-L-6-v2
```

They are small and easy to run, not biomedical normalization models. For a serious biomedical system, model choice and fine-tuning should be validated on biomedical normalization corpora.

## 24. Final comparison

| Component | Input | Output | Main role |
|---|---|---|---|
| TF-IDF retriever | mention / concept text | lexical similarity | transparent baseline |
| Bi-encoder | query and concept separately | vector similarity | fast candidate retrieval |
| Cross-encoder | query + candidate jointly | compatibility score | accurate reranking |

```text
mention + context
      ↓
bi-encoder
      ↓
top-k candidates
      ↓
cross-encoder
      ↓
best concept
```

## Take-home messages

1. A bi-encoder independently embeds the query and concepts.
2. Concept embeddings can be precomputed.
3. A cross-encoder jointly processes a query-candidate pair.
4. Cross-encoders are more expensive but can model richer interactions.
5. They are naturally combined as: bi-encoder to retrieve, cross-encoder to rerank.
6. Candidate Recall@k matters because reranking cannot recover an unretrieved concept.

---

# How the two notebooks fit together

```text
Main notebook                          Optional notebook
-----------                            -----------------
Terminology + exact lookup      →      same terminology
TF-IDF candidate retrieval      →      bi-encoder retrieval
(not implemented)               →      cross-encoder reranking
Accuracy@1 / Recall@3           →      Recall@3 + Accuracy@1 after rerank
```

Read the main notebook first: it defines the terminology, the evaluation protocol, and the two-stage mindset with a transparent lexical retriever. The optional notebook keeps the same data and metrics and swaps only the retrieval and reranking models for neural ones.

# Evaluation summary

Both notebooks use the same six-item `EVAL_DATA` and the same two metric families:

- **Accuracy@1**: fraction of items whose top-ranked concept equals the gold concept.
- **Recall@k** (k=3): fraction of items whose gold concept appears in the top-k candidates.

Recall@k diagnoses candidate generation; Accuracy@1 diagnoses final ranking. The main notebook reports both for the TF-IDF retriever. The optional notebook reports bi-encoder Recall@3 (retrieval) and final cross-encoder Accuracy@1 (after reranking), so the two stages can be scored separately.

# Suggested reading order

1. Run the main notebook top to bottom and inspect every printed table.
2. Check the exact dictionary outcomes for `AS` and `breast tumour`.
3. Confirm that context A and context B flip the ranking for `AS`.
4. Read the optional notebook sections 0 to 10 (bi-encoder only).
5. Then sections 11 to 16 (cross-encoder reranking).
6. Finish with the evaluation cells and compare Recall@3 against final Accuracy@1.
