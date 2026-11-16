# Biomedical Named Entity Recognition and Concept Normalization: Architectural Principles, Terminologies, and Technical Pipelines

## 1. System Framing: Mention Detection vs. Concept Normalization

Biomedical text mining relies on two connected operations to extract structured medical facts from raw text. As documented in the lecture materials, the complete information extraction task splits into Named Entity Recognition (NER) and Entity Normalization (also named Entity Linking).

Named Entity Recognition identifies text spans and assigns broad semantic types. For example, NER detects the word sequence _"breast carcinoma"_ in a clinical document and marks this sequence as a `DISEASE` entity. However, NER does not provide a unique database code for the detected mention.

Entity Normalization converts extracted mentions into canonical concept identifiers from standard biomedical terminologies. While NER determines which text span is present, normalization determines the exact biomedical concept that the span denotes.

```
+-----------------------------------------------------------------------+
|  Text Input: "Patient shows signs of chronic breast carcinoma."       |
+-----------------------------------------------------------------------+
                                  |
                                  v
+-----------------------------------------------------------------------+
|  1. Mention Detection (NER): [breast carcinoma] -> Type: DISEASE     |
+-----------------------------------------------------------------------+
                                  |
                                  v
+-----------------------------------------------------------------------+
|  2. Concept Normalization: [breast carcinoma] -> Concept ID: C0006142 |
+-----------------------------------------------------------------------+
```

The table below contrasts the main characteristics of these two processing stages.

|   |   |   |   |
|---|---|---|---|
|Processing Stage|Primary Input|Operational Goal|Output Artifact|
|**Named Entity Recognition (NER)**|Unstructured text tokens|Detect entity span boundaries and classify broad entity types|Labeled character offsets and category labels (for example, `DISEASE`)|
|**Entity Normalization**|Extracted text mentions and surrounding context|Map text mentions to standardized knowledge base entries|Unique concept identifiers (for example, UMLS CUI or MeSH ID)|

Separating mention recognition from normalization allows modular system design. Engineers can evaluate boundary detection errors independently from concept mapping errors.

## 2. Structured Domain Knowledge and Benchmark Datasets

Concept normalization depends on external terminologies and ontologies. These resources supply the target concepts, preferred names, unique identifiers, and text synonyms necessary for mapping.

### Standard Biomedical Terminologies

Biomedical terminologies provide structured domain knowledge. They act as reference dictionaries rather than training datasets. The primary terminologies categorized by domain include:

- **Disease Ontologies**: Medical Subject Headings (MeSH), Online Mendelian Inheritance in Man (OMIM), SNOMED CT, and the Unified Medical Language System (UMLS).
- **Chemical and Drug Databases**: RxNorm, MeSH, and DrugBank.
- **Gene and Protein Repositories**: NCBI Gene and UniProtKB.
- **Phenotype Vocabularies**: Human Phenotype Ontology (HPO).

The Unified Medical Language System (UMLS) serves as a meta-thesaurus. It integrates multiple source vocabularies into a single framework using Concept Unique Identifiers (CUIs).

### Standard Benchmark Datasets

System performance is measured on standardized human-annotated corpora. These datasets contain both mention boundary annotations and concept identifier tags:

1. **NCBI Disease Corpus**: Contains 793 PubMed abstracts fully annotated for disease mentions and mapped to MeSH or OMIM identifiers. It provides a standard benchmark for disease NER and normalization.
2. **BC5CDR Corpus**: Developed for the BioCreative V Chemical-Disease Relation task. It contains 1,500 PubMed abstracts annotated for chemical mentions, disease mentions, and chemical-induced disease relations.

## 3. The Four-Stage Normalization Pipeline

A complete normalization pipeline converts a text mention into a canonical identifier through four sequential processing steps.

```
[Mention Detection] ---> [Candidate Generation] ---> [Concept Ranking] ---> [Identifier Assignment]
```

As detailed in the lecture walkthrough, each pipeline step performs a specific role:

1. **Mention Detection**: The system identifies the target entity span and type from the source text.
2. **Candidate Generation**: The system retrieves a subset of top-k plausible candidate concepts from the terminology. This step reduces the search space from hundreds of thousands of terminology entries down to a manageable list (for example, k = 10 to k = 100).
3. **Concept Ranking**: The system computes similarity scores between the mention and candidate concepts. Advanced models incorporate surrounding sentence context to re-rank candidates.
4. **Identifier Assignment**: The system selects the highest-scoring candidate and outputs the final canonical identifier.

## 4. Methodological Progression: Rules, Lexical Search, and Neural Encoders

Biomedical normalization techniques evolve from rigid dictionary lookups to context-aware neural architectures.

### Rule-Based and Dictionary Normalization

Dictionary lookup methods use string matching tables that map preferred names and known synonyms directly to concept identifiers.

Before matching, surface strings undergo text preprocessing:

- Lowercasing text
- Stripping punctuation characters
- Normalizing whitespace characters

A common rule-based implementation uses the spaCy `EntityRuler` component. This component matches exact surface patterns and attaches pre-assigned concept identifiers directly during tokenization.

- **Advantages**: Very fast execution, complete transparency, and high precision on exact matches.
- **Limitations**: Fails completely on unseen morphological variants, spelling errors, or ambiguous abbreviations.

### Lexical Retrieval via TF-IDF Character N-Grams

To handle spelling variations and partial matches, lexical retrieval models build vector space representations using Term Frequency-Inverse Document Frequency (TF-IDF) over character n-grams.

For each terminology concept, the system concatenates its preferred name and synonyms into a single concept text document:

\text{concept\_text} = \text{preferred\_name} + \text{" "} + \text{join}(\text{synonyms})

The system vectorizes concept texts and input mentions using character n-gram TF-IDF (typically n=3 to n=5). It then calculates similarity scores using cosine similarity:

\text{score}(q, C) = \cos(\mathbf{v}_q, \mathbf{v}_C) = \frac{\mathbf{v}_q \cdot \mathbf{v}_C}{\|\mathbf{v}_q\| \|\mathbf{v}_C\|}

Where \mathbf{v}_q represents the TF-IDF vector of the mention query and \mathbf{v}_C represents the TF-IDF vector of candidate concept C. Character n-grams enable fuzzy matching, capturing minor typos and morphological changes.

### Neural Architectures: Bi-Encoders and Cross-Encoders

Neural models project mentions and terminology descriptions into dense vector embeddings.

- **Bi-Encoder Models**: Bi-encoders (such as SapBERT and BioSyn) use pretrained transformer backbones (like PubMedBERT) to compute independent dense embeddings for input mentions and concept descriptions. Candidate retrieval uses fast approximate nearest neighbor search in vector space.
- **Cross-Encoder Models**: Cross-encoders concatenate the mention with surrounding sentence context and full candidate descriptions. The transformer processes full pairwise interactions between mention context and candidate text.

Cross-encoders resolve ambiguous mentions where the same string denotes different concepts based on context. For example, the abbreviation _"AS"_ denotes _Ankylosing Spondylitis_ when surrounded by terms like _"back pain"_, but denotes _Aortic Stenosis_ when surrounded by terms like _"aortic valve"_.

The table below summarizes the trade-offs across these normalization methodologies.

|   |   |   |   |   |
|---|---|---|---|---|
|Method Class|Technical Approach|Computational Cost|Handling of Unseen Variants|Context Awareness|
|**Exact Dictionary**|Table lookup / spaCy `EntityRuler`|Very Low (O(1))|None (Exact string match only)|No context resolution|
|**Lexical Retrieval**|Character n-gram TF-IDF + Cosine Similarity|Low to Medium|High (Handles typos and morphology)|Limited to lexical context|
|**Dense Bi-Encoder**|Pretrained Transformer (SapBERT) Embeddings|Medium (Pre-indexed)|High (Captures semantic similarity)|Moderate|
|**Cross-Encoder Re-ranker**|Full Joint Transformer Attention|High|Very High|Full context resolution|

### Hybrid Normalization Pipelines

Practical industrial systems combine these methods into hybrid pipelines. A rule-based lookup processes high-confidence exact matches first. If no exact match exists, a fast bi-encoder or TF-IDF retriever generates candidate entries, and a cross-encoder re-ranks the top candidates. Tools like DNorm, TaggerOne, and scispacy follow these hybrid design patterns.

## 5. Quantitative Evaluation and Diagnostic Metrics

Evaluating biomedical entity processing requires separate metrics for Named Entity Recognition, Concept Normalization, and End-to-End system performance.

### Named Entity Recognition Metrics

Standard span-based precision (P), recall (R), and F1-score (F1) evaluate mention boundary detection:

P_{\text{NER}} = \frac{|\text{Correct Entity Spans}|}{|\text{Predicted Entity Spans}|}

R_{\text{NER}} = \frac{|\text{Correct Entity Spans}|}{|\text{Gold Entity Spans}|}

F1_{\text{NER}} = \frac{2 \cdot P_{\text{NER}} \cdot R_{\text{NER}}}{P_{\text{NER}} + R_{\text{NER}}}

### Concept Normalization Metrics (Gold Mentions)

When evaluating normalization independently, models receive gold-standard human-annotated mention spans as input. Evaluation uses three diagnostic metrics:

1. **Accuracy@1**: The percentage of mentions where the top-ranked concept identifier matches the gold concept identifier.
2. **Candidate Recall@k**: The proportion of mentions where the correct concept identifier appears within the top-k retrieved candidates. Candidate recall measures the quality of candidate generation.
3. **Mean Reciprocal Rank (MRR)**: Evaluates ranking quality across candidate lists:

\text{MRR} = \frac{1}{|M|} \sum_{i=1}^{|M|} \frac{1}{\text{rank}_i}

Where |M| is the total number of evaluation mentions, and \text{rank}_i is the position of the ground-truth concept in the candidate list for mention i.

### End-to-End Evaluation Metrics

End-to-end evaluation requires a system to correctly detect the mention span, assign the correct entity category, and map the span to the correct canonical concept identifier. An output prediction counts as correct only if all three conditions are satisfied simultaneously.

```
End-to-End Correctness = (Correct Span Boundary) AND (Correct Entity Type) AND (Correct Concept ID)
```

Diagnostic error analysis separates pipeline failures into boundary errors (e.g., extracting _"breast"_ instead of _"breast carcinoma"_), type assignment errors, candidate generation omissions, and context disambiguation errors.