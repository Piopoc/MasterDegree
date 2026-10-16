---
type: course-note
date: 2026-10-09
tags:
  - course
  - graph-databases
  - llm
course: "[[Graph Databases]]"
sources:
  - "[[LOTUS]]"
ai-first: true
---

## For Future Agent

Study note for the [[Graph Databases]] course on the LOTUS system and its **semantic operators** - declarative LLM-powered relational transformations with statistical accuracy guarantees. Created 2026-10-09 by translating and restructuring [[LOTUS]] (Italian) and verifying the paper citation online. Key points: langex-parameterized operators, the reference-algorithm semantics, the operator toolbox, and cost optimization via proxy scores and thresholds $\tau^-, \tau^+$.

## Source / Citation

Liana Patel, Siddharth Jha, Melissa Pan, Harshit Gupta, Parth Asawa, Carlos Guestrin, Matei Zaharia. **"Semantic Operators and Their Optimization: Enabling LLM-Based Data Processing with Accuracy Guarantees in LOTUS."** *Proc. VLDB Endow.* 18(11): 4171-4184, 2025.

- Paper (PDF): https://www.vldb.org/pvldb/vol18/p4171-patel.pdf
- DOI: https://doi.org/10.14778/3749646.3749685
- Code: https://github.com/lotus-data/lotus

(all links verified 2026-10-09; lecture summary - see Sources)

## 1. Motivation: Why Existing Approaches Fail

- **Traditional SQL search** over explicit tables requires exhaustive pre-processing of all possible categories; keyword matching fails on synonyms and gives no guarantee of contextual semantic relevance.
- **IR and embeddings** produce a continuous ranking, but no crisp boundary for what to keep or discard.
- **Procedural LLM pipelines** force manual handling of row iteration, prompt construction, error handling, and cost/latency optimization.

## 2. The Semantic Operator Model

- **Definition:** a **semantic operator** is a *declarative* transformation over one or more datasets, parameterized by a natural-language expression (**langex**).
- **Separation of concerns:** decouples application logic (*what* you want) from AI execution logic (*how* models are invoked).
- **langex syntax:** attribute names are embedded in curly braces inside the predicate, e.g. `{plot}`.
- **Reference algorithm:** because many execution strategies exist, correct operator behavior is *formally defined* with respect to a high-quality reference algorithm - optimizations must approximate it, not invent semantics.

## 3. The Operator Toolbox

Extends classic relational transformations with semantic reasoning:

| Operator | Semantics | Complexity (reference) |
|---|---|---|
| `sem_filter` | Keeps rows satisfying the langex predicate. | $O(N)$ |
| `sem_join` | Joins two tables evaluating a langex predicate over row pairs. | $O(\lvert T_1 \rvert \cdot \lvert T_2 \rvert)$ |
| `sem_topk` | Top $k$ rows best satisfying a semantic ordering criterion. | - |
| `sem_agg` | Aggregates/combines many rows into one syntagmatic result. | - |
| `sem_group_by` | Groups rows into semantic categories described by the langex. | - |
| `sem_map` | Transforms/projects each row independently per instructions. | - |

## 4. Optimization with Statistical Accuracy Guarantees

Calling a large LLM on every row ($O(N)$ for filters) or every pair ($O(N \cdot M)$ for joins) is prohibitively expensive. The optimizer reduces cost while providing **statistical accuracy guarantees** - target precision/recall $\gamma$ at confidence level $1 - \delta$ - relative to the reference algorithm.

### Cascade for `sem_filter`

1. A smaller, cheaper **helper model** computes a *proxy score* from token log-probabilities.
2. Proxy scores are recalibrated into **quantiles** over the score range.
3. **Importance + uniform sampling** selects a subset to evaluate with the main model (the *oracle*).
4. Statistically estimate a rejection threshold $\tau^-$ and an acceptance threshold $\tau^+$, accounting for statistical uncertainty.
5. Tuples outside $[\tau^-, \tau^+]$ are auto-rejected / auto-accepted; the expensive oracle is invoked **only for uncertain tuples** between $\tau^-$ and $\tau^+$.

### Approximation for `sem_join`

Avoids quadratic cost by estimating proxy scores via **embedding similarity**, with two strategies:

- **Sim-Filter** - direct embedding similarity.
- **Project-Sim-Filter** - intermediate LLM projection of rows, then embedding similarity.

## 5. The LOTUS Framework

- **LOTUS** = **L**LMs **O**ver **T**ables of **U**nstructured and **S**tructured data - the system implementing semantic operators with a **Pandas DataFrame-style API** in Python.
- Operators compose into declarative pipelines:

```python
movies.sem_filter(...).sem_topk(...).sem_agg(...)
```

- The infrastructure handles execution and optimization of the underlying models transparently.

## 6. Verified Results from the Paper (additions beyond the lecture summary)

From the PVLDB abstract:

- Optimizations accelerate semantic filtering, joining, group-by and top-k by **up to 1,000x** vs. reference algorithms.
- LOTUS matches or exceeds state-of-the-art AI pipeline quality while running **up to 3.6x faster** than the highest-quality baselines.
- vs. AI analytics systems (UQE, DocETL): similar or **up to 170% higher accuracy** with statistical guarantees the others do not provide.

## 7. Exam Hooks

- The core exam distinction: *declarative operator + langex* (what) vs. *execution strategy* (how), with correctness anchored to the **reference algorithm**.
- Guarantees are stated as $(\gamma, 1-\delta)$ pairs - precision/recall target and confidence - not as absolute correctness.
- The cascade is the template answer for "how do you avoid calling the big model on everything?" - proxy scores -> sampling -> thresholds -> oracle only on the uncertain band.

## Related

- [[NLP Linguistic Structures and Annotation]] - the linguistic layer under any semantic predicate (cross-course)
- [[Linked Open Data]] - query/publish layer over the same graph stack
- [[Graph Databases]] - course hub

## Sources

- [[LOTUS]] (AY 2026/2027)
- [PVLDB paper](https://www.vldb.org/pvldb/vol18/p4171-patel.pdf) (2025)
