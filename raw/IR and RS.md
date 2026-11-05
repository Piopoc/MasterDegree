# Foundations of Information Retrieval and Recommender Systems

## 1. Document Overview and Scope

This technical report summarizes the lecture content from **L4.pdf**, entitled _"Foundations of Information Retrieval and Recommender Systems"_. The lecture is part of the course _"Privacy Preserving Information Access"_ taught by **Guglielmo Faggioli** at the **Intelligent Interactive Information Access (IIIA) Hub**, Department of Information Engineering, **University of Padua** (Academic Year 2026/2027). The course targets Master Degree programs in Cybersecurity, Computer Engineering, and International Cybersecurity and Cyberintelligence.

The lecture material contains machine-printed slide text, tabular models, mathematical equations, structural diagrams, and explicit handwritten annotations. This report integrates all content elements without external additions.

**Key Foundational Definitions**

- **Database (DB)**: A structured collection of data.
- **Information Retrieval (IR)**: A field more related to unstructured information and user relevance.

## 2. Comparison Between Database Systems and Information Retrieval

Database systems and Information Retrieval systems process data differently. Relational databases manage structured records made of well-defined attributes (such as account numbers, balances, names, addresses, and social security numbers). Field comparison in databases uses well-defined semantics. In contrast, Information Retrieval handles text and multimedia data, where exact word matching is insufficient because natural languages contain multiple expressions for identical concepts.

|   |   |   |
|---|---|---|
|Feature / Dimension|Database Systems (DB)|Information Retrieval Systems (IR)|
|**Data Unit**|Structured data with clear semantics based on a formal model.|Mostly unstructured data (for example, free text with some organization).|
|**Query Character**|Formally defined, unambiguous queries.|Vague and imprecise information needs.|
|**Result Semantics**|Correct in a formal sense under a closed-world assumption.|Sometimes relevant, often not, under an open-world assumption.|
|**Matching Rule**|Exact match without document ranking.|Best match with ranked document output.|
|**Interaction Pattern**|One-shot queries with no user interaction.|Continuous interaction is fundamental.|

## 3. Information Retrieval Architecture and System Mechanics

An Information Retrieval system operates across two primary execution environments: an offline processing pipeline and an online search pipeline.

### System Architecture Pipeline ("The Y" Model)

1. **Offline Pipeline**:
    - The input **Corpus** (a collection of raw documents) undergoes **Indexing**.
    - Indexing converts documents into an **Index** that stores a **Bag of Words (BoW)** document representation.
2. **Online Pipeline**:
    - A user formulates a statement of an **Information Need** as a **Query**.
    - The system applies **Indexing** to create a **Query Representation**.
    - The **Matching** engine executes a **Retrieval Model**, comparing the query representation with the document index.
    - Matching computes a numerical **Retrieval Score** for candidate documents and generates **Retrieval Results**.

```
[ Offline Pipeline ]                            [ Online Pipeline ]
Documents (Corpus)                             Information Need
        │                                              │
        ▼                                              ▼
     Indexing                                        Query
        │                                              │
        ▼                                              ▼
 Index (Bag of Words)                               Indexing
        │                                              │
        │                                              ▼
        │                                     Query Representation
        │                                              │
        └──────────────────────┬───────────────────────┘
                               │
                               ▼
                   Matching (Retrieval Model) ───► Calculates Retrieval Score
                               │
                               ▼
                       Retrieval Results
```

### Document Preprocessing Workflow

Before index construction, raw text documents pass through three preprocessing stages:

1. **Lexical Analysis / Tokenization**: Converting continuous text streams into distinct term tokens.
2. **Elimination of Stopwords**: Filtering out high-frequency words using a predefined stoplist.
3. **Stemming**: Reducing words to their root grammatical stem.

## 4. Inverted Index Mechanics and Search Operations

An inverted index converts terms into pointers. This structure fulfills two core system functions:

- **Computational Complexity Reduction**: It restricts search operations strictly to documents containing query terms.
- **Spatial Complexity Reduction**: It reduces corpus size by replacing repeated words with structural pointers.

### Inverted Index Structural Types

The lecture defines two inverted index formats:

1. **Document-Frequency Inverted Index**:
    - Stores term-to-document postings along with term frequencies (for example, posting entry `1:2` indicates Document 1 contains the term 2 times).
    - **Capabilities**: Enables document ranking by term relevance.
    - **Limitations**: Does not support phrase matching or term proximity matching.
2. **Document-Position Inverted Index**:
    - Stores term-to-document postings along with specific word token positions (for example, posting entry `1:10,17` indicates the term appears at word positions 10 and 17 in Document 1).
    - **Capabilities**: Enables relevance ranking, exact phrase matching (for example, `"small marsupial"`), and proximity matching (for example, finding `"small"` within 5 words of `"marsupials"`).

### Boolean Search Operations

Given a boolean query composed of terms and boolean operators, the search engine applies set operations to posting lists:

- **AND Operator**: Computes the **intersection** of posting lists (retrieves documents containing all specified terms).
- **OR Operator**: Computes the **union** of posting lists (retrieves documents containing at least one specified term).

## 5. Lexical Retrieval Models and Mathematical Formulations

Retrieval models define a logical framework to represent documents and queries, applying a ranking function to score document relevance.

### Term Frequency - Inverse Document Frequency (TF-IDF)

TF-IDF measures term importance within a document relative to the entire collection:

- **Term Frequency (TF)**: Measures how often a token appears in a document normalized by document length: \text{TF} = \frac{\text{number of times a token appears in a document}}{\text{length of the document}}
- **Inverse Document Frequency (IDF)**: Measures term rarity across the corpus: \text{IDF} = \log\left(\frac{\text{total number of documents}}{\text{number of documents containing the token}}\right)
- **TF-IDF Calculation**: High TF indicates strong document topicality, while high IDF indicates term specificity. \text{TF-IDF} = \text{TF} \times \text{IDF}

### Vector Space Model (VSM)

The Vector Space Model represents queries and documents as vectors in a V-dimensional space, where each index token forms a distinct dimension. The model measures similarity using the cosine of the angle between query vector \vec{q} and document vector \vec{d}:

R(q, d) = \cos(\theta(\vec{q}, \vec{d})) = \frac{\vec{q} \cdot \vec{d}}{|\vec{q}| |\vec{d}|} = \frac{\sum_{j=1}^V q_j d_{j,k}}{\left(\sqrt{\sum_{j=1}^V q_j^2}\right)\left(\sqrt{\sum_{j=1}^V d_{j,k}^2}\right)}

The denominator performs vector (document) length normalization, placing higher-ranked documents closer to the query vector.

### Other Classical Lexical Models

The lecture provides mathematical formulations for two additional classical lexical models:

1. **Language Model**: \text{score}(D, Q) \propto \left( \prod_{w \in Q} \lambda (\text{TF}_{w,D}) \cdot (1 - \lambda)\text{IDF}_w + 1 \right)
2. **BM25 Model**: \text{score}(D, Q) = \sum_{w \in Q} \text{IDF}_w \frac{\text{TF}_{w,D} \cdot (k_1 + 1)}{\text{TF}_{w,D} + k_1 \cdot \left( 1 - b + b \cdot \frac{|D|}{\text{avgdl}} \right)}

**Handwritten Note on Exam Scope**

An explicit handwritten note on the slide states: _"non le chiede, basta sapere che ci sono"_ _(English translation: "he/she does not ask them, it is enough to know that they exist")._

## 6. Evaluation Paradigms in Information Retrieval

### The Cranfield Paradigm

Established by Cyril W. Cleverdon in the mid-1960s, the Cranfield paradigm provides a standardized experimental setup for system evaluation. It relies on test collections consisting of:

1. A **Document Corpus**.
2. **Topics** (formal representations of information needs).
3. **Relevance Judgments** (also called ground-truth, relevance assessments, or **qrels**), using binary or multi-graded scales.

This paradigm ensures experiment comparability and repeatability.

### Taxonomy of IR Evaluation Measures

|   |   |   |
|---|---|---|
|Relevance Scale|Set-Based Retrieval|Rank-Based Retrieval|
|**Binary Relevance**|Precision (P), Recall (R), F-measure (F).|Precision at k (P@k), Recall at k (R@k), R-Precision (Rprec), Average Precision (AP), Rank-Biased Precision (RBP).|
|**Multi-graded Relevance**|No widely agreed generalizations.|Discounted Cumulated Gain (DCG), Normalized Discounted Cumulated Gain (nDCG).|

### Set-Based Evaluation Measures

Given a set of relevant documents A and a set of retrieved documents B:

- **Precision (****P****)**: The proportion of retrieved documents that are relevant: P = \frac{|A \cap B|}{|B|}
- **Recall (****R****)**: The proportion of relevant documents that are retrieved: R = \frac{|A \cap B|}{|A|}
- **F-measure (****F****)**: The harmonic mean of Precision and Recall: F = \frac{2}{\frac{1}{P} + \frac{1}{R}} = 2 \frac{P \cdot R}{P + R}

Maximizing Precision and Recall aligns with the Probability Ranking Principle (ordering documents by decreasing probability of relevance).

### Rank-Based Evaluation Measures (DCG and nDCG)

Discounted Cumulated Gain (DCG) penalizes relevant documents that appear lower in ranked output. For an example with total relevant documents RB = 8 and an impatient user, DCG accumulates discounted relevance weights:

DCG = 3 + \frac{1}{\log_2(3)} + \frac{2}{\log_2(4)} + \frac{2}{\log_2(8)} = 5.2976

To normalize scores between 0 and 1, Normalized Discounted Cumulated Gain (nDCG) divides the run DCG by the ideal run iDCG (the pool sorted in descending order of relevance):

nDCG(k) = \frac{DCG(k)}{iDCG(k)}

For the lecture example (DCG = 5.2976, iDCG = 10.1996):

nDCG = \frac{5.2976}{10.1996} = 0.5194

## 7. Recommender Systems Principles and Cognitive Dynamics

### The Paradox of Choice and Decision Fatigue

Recommender systems address cognitive limits in human decision making. Users prefer having options up to a specific limit. Above this threshold, choosing consumes excess cognitive resources, leading users to avoid selection entirely or experience anxiety over choice correctness.

**Handwritten Note on Cognitive Load**

A handwritten annotation explicitly labels this psychological phenomenon as `**decision fatigue**`.

### Recommender System Definition and Architecture

A Recommender System takes a user u \in U and an item set I, computing a numerical score s for each item i \in I. A higher score indicates greater predicted user interest.

```
                  [ Recommender System Architecture ]
                                   │
      ┌────────────────────────────┴────────────────────────────┐
      ▼                                                         ▼
User (u ∈ U)                                               Item Set (I)
  │ (User acts as the Query)                                 │
  └────────────────────────────┬────────────────────────────────┘
                               │
                               ▼
                   Computes Score s for Item i
                               │
                               ▼
                     Ranked Recommendations
```

**Handwritten Annotations on System Dynamics**

- **Query Generation**: A Recommender System acts as an _indirect query to Search Engines_, where _the user itself serves as the query_.
- **Personalization Need**: Personalized Recommender Systems are necessary to process _user feedback, fresh information, and rich information_.
- **Implicit Feedback**: When user feedback is uncertain, the system uses _implicit feedback_.
- **Trust Factor**: User trust is essential. System designers must protect user privacy not only for legal compliance, but to establish system trustworthiness.

### The Rating Matrix

The system structures user-item interactions in a **Rating Matrix** (Users \times Items). Entries do not require explicit star ratings; they can store binary signals or continuous values (such as video watch duration).

## 8. Recommender Systems Taxonomy and Algorithmic Approaches

```
                          [ Recommender Systems ]
                                     │
         ┌───────────────────────────┴───────────────────────────┐
         ▼                                                       ▼
Content-Based Systems                               Collaborative Filtering Systems
(Item & User Attribute Matching)                                 │
                                         ┌───────────────────────┴───────────────────────┐
                                         ▼                                               ▼
                              Non-Personalized                                      Personalized
                            (Popularity Filtering)                                       │
                                                                       ┌─────────────────┴─────────────────┐
                                                                       ▼                                   ▼
                                                              Memory-Based (KNN)                  Model-Based (Matrix Factorization)
```

### 1. Content-Based Recommender Systems

- Characterizes items by explicit attributes (color, cost, genre, actors, material).
- Profiles users based on preferred attributes derived from historical activity.
- Computes recommendations by matching item attributes with user profiles.

### 2. Collaborative Filtering (CF) Systems

Exploits data from multiple users. Requires user willingness to share data (explicitly or implicitly).

#### A. Popularity Filtering (Non-Personalized)

- Recommends items with the highest overall user interaction.
- **PROS**: Simple computation; presents no privacy issues (top ranks can be computed using **Differential Privacy**); achieves high practical performance.
- **CONS**: Provides no personalization.
- **Handwritten Note**: Popularity filtering targets _most interacted items, which are not necessarily the best ones_.

#### B. Memory-Based Collaborative Filtering (Personalized)

- Extends the k-Nearest Neighbors (k-NN) algorithm, maintaining the full rating matrix in memory.
- Computes similarity between users or items to infer missing ratings.
- **Prediction Formula**: s_{u,i} = \frac{\sum_{v \in \mathcal{U}_i} r_{v,i} \cdot \cos(u, v)}{r_{\max} \cdot |v \in \mathcal{U}_i|} _Where_ _v \in \mathcal{U}_i_ _represents users who rated item_ _i__, weighted by user similarity_ _\cos(u, v)__, normalized by maximum rating_ _r_{\max}_ _and user count._
- **PROS**: Simple temporal computation; effective output.
- **CONS**: High spatial memory cost; unscalable to large datasets.

#### C. Model-Based Collaborative Filtering (Matrix Factorization)

- Factorizes the Rating Matrix into smaller matrices (Users \times Latent Features and Latent Features \times Items).
- **Handwritten Note**: Matrix factorization is applied specifically _"(to reduce memory consumption)"_.
- **Mechanism**: Compresses raw data into latent feature vectors, removing noise and representing intrinsic user preferences and item characteristics.

## 9. Evaluation Methodologies for Recommender Systems

Evaluation techniques for Recommender Systems adapt based on whether the task is framed as classification, ranking, or rating prediction.

```
                     [ Recommender System Evaluation ]
                                     │
   ┌──────────────────────┬──────────┴───────────┬──────────────────────┐
   ▼                      ▼                      ▼                      ▼
Classification         Ranking                 ROC / AUC           Rating Prediction
(P, R, F1-Score)     (nDCG, AP)       (Area Under ROC Curve)        (RMSE Metric)
```

### 1. Classification Metrics

By converting ratings into binary relevance classes, evaluation uses standard classification measures:

- **Precision**: Proportion of recommended items that are relevant.
- **Recall**: Proportion of relevant items that are recommended.
- **F1-score**: Harmonic mean of Precision and Recall.

### 2. Ranking Metrics

By framing recommendations as a ranked sorting task, evaluation applies IR ranking metrics:

- **nDCG (Normalized Discounted Cumulated Gain)**.
- **AP (Average Precision)**.

### 3. Area Under the ROC Curve (AUC)

- Measures the Area Under the Receiver Operating Characteristic (ROC) curve, plotting True Positive Rate against False Positive Rate.
- Rarely used in traditional IR, but widely applied in Recommender Systems evaluation.

### 4. Rating Prediction Metric (RMSE)

When predicting exact numerical ratings, evaluation uses **Root Mean Squared Error (RMSE)**:

\text{RMSE} = \sqrt{\frac{\sum_{u \in \mathcal{U}_T} \sum_{i \in \mathcal{I}_T} (\hat{r}_{u,i} - r_{u,i})^2}{|\mathcal{U}_T| \cdot |\mathcal{I}_T|}}

_Where_ _\hat{r}_{u,i}_ _is the predicted rating,_ _r_{u,i}_ _is the true rating,_ _\mathcal{U}_T_ _is the test user set, and_ _\mathcal{I}_T_ _is the test item set._