---
type: course-note
date: 2026-11-04
tags:
  - course
  - ppia
  - information-retrieval
  - evaluation
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[IR and RS]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course (Lecture 4, part 1) covering **information retrieval**: how IR differs from database querying, the two-pipeline ("Y") system architecture, preprocessing and inverted indexes, the lexical retrieval models (TF-IDF, vector space model, language model, BM25), and the Cranfield evaluation paradigm with set-based and rank-based measures. Created 2026-11-04 from the L4 technical report [[IR and RS]]; companion note for the same lecture is [[Recommender Systems]]. Key points: closed-world exact matching vs open-world best-match ranking, posting lists and boolean set operations, cosine as length-normalized similarity, P/R/F, DCG/nDCG with the lecture's worked numbers.

## 1. Databases vs Information Retrieval

A **database** is a structured collection of data queried by exact semantics. **Information retrieval (IR)** deals instead with unstructured information (text, multimedia) and with *user relevance*, where exact word matching is insufficient because natural language expresses the same concept in many ways.

| Dimension | Database systems (DB) | Information retrieval (IR) |
|---|---|---|
| **Data unit** | Structured data with clear semantics, based on a formal model | Mostly unstructured data (free text with some organization) |
| **Query character** | Formally defined, unambiguous | Vague, imprecise information needs |
| **Result semantics** | Correct in a formal sense, **closed-world** assumption | Sometimes relevant, often not, **open-world** assumption |
| **Matching rule** | Exact match, no ranking | Best match, **ranked** output |
| **Interaction** | One-shot query, no user interaction | Continuous interaction is fundamental |

Two consequences worth remembering:

- The DB answer is a **set** (either a tuple satisfies the predicate or it does not); the IR answer is a **ranking** (every document gets a score, and relevance is a matter of degree).
- The IR notion of correctness is **empirical**: it cannot be proved from the schema, it must be measured against user judgments - which is exactly why the evaluation machinery in section 6 exists.

## 2. System Architecture: the "Y" Model

An IR system has two execution environments that meet in the matching stage:

1. **Offline pipeline** - the **corpus** (collection of raw documents) is processed by **indexing** into an **index** holding a **bag-of-words** representation of each document.
2. **Online pipeline** - the user formulates an **information need** as a **query**; the same **indexing** procedure produces a **query representation**; the **matching** engine applies a **retrieval model** to compare query and document representations, computes a numerical **retrieval score** per candidate, and returns **retrieval results**.

```text
[ Offline Pipeline ]                        [ Online Pipeline ]
Documents (Corpus)                          Information Need
        |                                           |
        v                                           v
     Indexing                                     Query
        |                                           |
        v                                           v
Index (Bag of Words)                            Indexing
        |                                           |
        |                                           v
        |                                  Query Representation
        |                                           |
        +---------------------+---------------------+
                              |
                              v
                 Matching (Retrieval Model)
                              |
                              v
                    Retrieval Results (ranked)
```

The shape gives the model its name: two branches converge into one matching stage. The architectural point is that **documents and queries must be represented in the same space by the same procedure** - otherwise the comparison is meaningless.

## 3. Document Preprocessing

Before index construction, raw text passes through three stages:

| Stage | Operation | Effect |
|---|---|---|
| **Lexical analysis / tokenization** | Split the continuous character stream into distinct **term tokens** | Defines the indexing units |
| **Stopword elimination** | Remove high-frequency words using a predefined **stoplist** | Shrinks the index, removes low-information terms |
| **Stemming** | Reduce each word to its grammatical **stem** | Merges morphological variants (`retrieval`, `retrieve` -> same stem) |

These steps are lossy by design: they buy recall and index compactness at the price of information (word order is discarded by the bag-of-words assumption, and stems are not words). Preprocessing is part of the retrieval model's inductive bias, not a neutral cleanup.

## 4. Inverted Index and Search Operations

An inverted index maps each **term** to **pointers** into the collection. It serves two purposes:

- **Computational complexity reduction** - search is restricted to documents that actually contain the query terms, instead of scanning the whole corpus.
- **Spatial complexity reduction** - repeated words are replaced by structural pointers, so the index is smaller than the raw text it represents.

### Two structural variants

| Variant | Posting content | Capabilities | Limits |
|---|---|---|---|
| **Document-frequency index** | Document id + term frequency (e.g. `1:2` = document 1 contains the term twice) | Ranking by term relevance | No phrase matching, no proximity matching |
| **Document-position index** | Document id + word positions (e.g. `1:10,17` = term at positions 10 and 17 of document 1) | Ranking **and** exact phrase matching (`"small marsupial"`) **and** proximity matching (`small` within 5 words of `marsupials`) | Larger index |

The trade-off is index size against query expressiveness: positions are what make phrase and proximity operators possible.

### Boolean operations on posting lists

A boolean query combines terms with operators evaluated as **set operations** over posting lists:

- **AND** -> **intersection** of the posting lists (documents containing *all* terms).
- **OR** -> **union** of the posting lists (documents containing *at least one* term).

Boolean retrieval returns an unranked set; turning it into a ranking requires a retrieval model.

## 5. Lexical Retrieval Models

A retrieval model is a logical framework that represents documents and queries and defines a **ranking function** scoring document relevance.

### TF-IDF

TF-IDF weights a term by how concentrated it is in one document against how rare it is in the collection:

$$\text{TF} = \frac{\text{occurrences of the token in the document}}{\text{length of the document}} \qquad \text{IDF} = \log\left(\frac{\text{total number of documents}}{\text{number of documents containing the token}}\right)$$

$$\text{TF-IDF}(t,d) = \text{TF}(t,d)\times \text{IDF}(t)$$

High TF means the document is *about* that token; high IDF means the token is *specific* (it discriminates). A term that is frequent everywhere carries little signal. The full treatment of TF-IDF, its smoothing variants and the scikit-learn implementation lives in the canonical note [[TF-IDF and Static Embeddings]] of the text analytics course - here only the exam-level definition is needed.

### Vector space model (VSM)

Documents and queries become vectors in a $V$-dimensional space, where $V$ is the number of index terms:

$$R(q,d) = \cos\left(\theta(\vec{q},\vec{d})\right) = \frac{\vec{q}\cdot\vec{d}}{\lvert\vec{q}\rvert\,\lvert\vec{d}\rvert} = \frac{\sum_{j=1}^{V} q_j\, d_j}{\sqrt{\sum_{j=1}^{V} q_j^2}\;\sqrt{\sum_{j=1}^{V} d_j^2}}$$

The denominator is **length normalization**: it removes the advantage of long documents and puts higher-ranked documents closer to the query vector. Cosine compares direction, not magnitude.

### Language model and BM25

Two further classical models are stated on the slides:

$$\text{score}(D,Q) \propto \prod_{w \in Q} \Big[\lambda\,P(w \mid D) + (1-\lambda)\,P(w \mid C)\Big] \qquad \text{(language model, Jelinek-Mercer mixture)}$$

$$\text{score}(D,Q) = \sum_{w \in Q} \text{IDF}_w \cdot \frac{\text{TF}_{w,D}\cdot(k_1+1)}{\text{TF}_{w,D} + k_1\left(1-b+b\cdot\frac{\lvert D\rvert}{\text{avgdl}}\right)} \qquad \text{(BM25)}$$

In BM25, $k_1$ controls term-frequency saturation (repeated occurrences stop adding score) and $b$ controls length normalization against the average document length $\text{avgdl}$. In the language model, $\lambda$ interpolates between the document and the collection distribution to avoid zero probabilities for unseen terms.

**Exam scope (handwritten note on the slide):** *"non le chiede, basta sapere che ci sono"* - the instructor does not ask for these two formulas; it is enough to know the models exist and what family they belong to. The TF-IDF/VSM machinery, by contrast, is fair game.

## 6. Evaluation: the Cranfield Paradigm

IR evaluation is standardized by the **Cranfield paradigm** (Cyril W. Cleverdon, mid-1960s). A **test collection** has three components:

1. A **document corpus**.
2. **Topics** - formal representations of information needs (what becomes the query).
3. **Relevance judgments** - ground truth, also called relevance assessments or **qrels**, on a binary or multi-graded scale.

The paradigm's value is **comparability and repeatability**: the same corpus, topics and qrels let different systems (or different versions of one system) be compared on identical ground.

### Taxonomy of measures

| Relevance scale | Set-based retrieval | Rank-based retrieval |
|---|---|---|
| **Binary** | Precision (P), Recall (R), F-measure (F) | P@k, R@k, R-Precision (Rprec), Average Precision (AP), Rank-Biased Precision (RBP) |
| **Multi-graded** | No widely agreed generalization | DCG, nDCG |

### Set-based measures

With $A$ = set of relevant documents and $B$ = set of retrieved documents:

$$P = \frac{\lvert A \cap B\rvert}{\lvert B\rvert} \qquad R = \frac{\lvert A\cap B\rvert}{\lvert A\rvert} \qquad F = \frac{2}{\frac{1}{P}+\frac{1}{R}} = 2\,\frac{P\cdot R}{P+R}$$

Precision answers *how clean is the result set*, recall *how complete it is*; they trade off against each other (retrieve more -> recall up, precision down). The F-measure is their harmonic mean, so it is dominated by the weaker of the two. Maximizing both is the operational reading of the **Probability Ranking Principle**: order documents by decreasing probability of relevance.

### Rank-based measures: DCG and nDCG

**Discounted Cumulated Gain** penalizes relevant documents placed low in the ranking, because an impatient user stops reading. Gains are discounted logarithmically by position:

$$\text{DCG} = \sum_{i=1}^{k} \frac{g_i}{\log_2(i+1)}$$

The lecture example (a pool with $RB = 8$ relevant documents, gains $3, 1, 2, 2$ in the first four positions):

$$\text{DCG} = 3 + \frac{1}{\log_2 3} + \frac{2}{\log_2 4} + \frac{2}{\log_2 8} = 3 + 0.6309 + 1 + 0.6667 = 5.2976$$

**nDCG** normalizes into $[0,1]$ by dividing the run's DCG by the **ideal** DCG ($iDCG$), i.e. the same pool sorted in descending order of relevance:

$$\text{nDCG}(k) = \frac{\text{DCG}(k)}{iDCG(k)} \qquad \text{nDCG} = \frac{5.2976}{10.1996} = 0.5194$$

Normalization is what makes runs with different pools or different numbers of judged documents comparable; a raw DCG is only meaningful against its own ideal.

## 7. Exam Hooks

- The DB/IR table is a classic short-answer question: reproduce the five dimensions (data unit, query, result semantics, matching, interaction).
- Be able to draw the "Y": corpus -> indexing -> index on the left, need -> query -> representation on the right, matching and scores at the junction.
- Know the two inverted-index variants and *which query operators each one enables* (positions buy phrases and proximity).
- AND = intersection, OR = union - and the result is unranked.
- Cosine formula with the normalization denominator spelled out; explain why normalization matters.
- Cranfield = corpus + topics + qrels; its purpose is comparability/repeatability.
- Compute a DCG/nDCG by hand: discount by $\log_2(i+1)$, then divide by the ideal ordering.
- BM25 and the language model: existence and family only, not the formulas.

## Related

- [[Recommender Systems]] - part 2 of the same lecture; it reuses P/R, AP and nDCG as recommendation metrics
- [[TF-IDF and Static Embeddings]] - canonical treatment of TF-IDF, weighting variants and cosine similarity
- [[Privacy Evaluation Metrics]] - the privacy-side counterpart: how protection is measured, where this note measures retrieval quality

## Sources

- [[IR and RS]] (AY 2026/2027)
