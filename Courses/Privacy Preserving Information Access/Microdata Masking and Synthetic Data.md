---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - microdata
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[microdati]]"
  - "[[Macrodata and Microdata]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course: the complete catalog of **microdata protection techniques** - non-perturbative masking, perturbative masking, and synthetic data generation. Created 2026-10-09 by merging the Italian source [[microdati]] with §4-5 of [[Macrodata and Microdata]] (the two overlap; this note is the canonical synthesis). Key points: 13 techniques with applicability (categorical vs. continuous), LaTeX for rounding intervals and Cholesky-based synthesis.

## 0. Overview

Preliminary step: **sanitization** - removal of direct identifiers (names, addresses, tax codes, phone numbers). To protect against quasi-identifier and sensitive-attribute re-identification, two macro-categories follow:

```
                     ┌───────────────────────────┐
                     │ Microdata Protection      │
                     └─────────────┬─────────────┘
        ┌──────────────────────────┴──────────────────────────┐
        ▼                                                     ▼
   Masking                                       Synthetic Data Generation
   ├── Non-Perturbative (suppress/eliminate)      ├── Fully Synthetic
   └── Perturbative (alter/add noise)             └── Partial / Perturbative Synthetic
```

## 1. Non-Perturbative Masking

Reduces detail or removes observations **without** altering existing values with false data.

| Technique | Mechanism | Categorical | Continuous |
|---|---|---|---|
| **Sampling** | Removes observations under a *"just a few"* policy, lowering the chance a specific individual is in the released sample. | YES | YES |
| **Local Suppression** | Replaces risky cells/attributes with a *missing value* (null). | YES | YES |
| **Global Recoding** | Partitions an attribute's domain into disjoint intervals; values replaced by interval label. | ONLY ORDERED | YES |
| **Top-Coding** | Variant: values above a threshold get a cap label (heights > 1.9m -> `">1.9m"`). | ONLY ORDERED | YES |
| **Bottom-Coding** | Mirror variant: values below a threshold get a floor label (heights < 1.7m -> `"<1.7m"`). | ONLY ORDERED | YES |
| **Generalization** | Uses a generalization hierarchy: full DoB -> birth year/month; zip code truncated to a prefix. | YES | VIA RECODING |

## 2. Perturbative Masking

Alters the original data by introducing noise, combinations, or variations.

| Technique                        | Mechanism                                                                                                                                                                                          | Categorical | Continuous |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | ---------- |
| **Rounding**                     | Replaces values with fixed interval points (see LaTeX below).                                                                                                                                      | NO          | YES        |
| **Resampling**                   | Resamples the column with replacement $k$ times, sorts each sample, averages row-wise, maps back.                                                                                                  | NO          | YES        |
| **Lossy Compression**            | Treats the data table as an image; applies lossy compression (e.g. JPEG); compression rate = obfuscation parameter.                                                                                | NO          | YES        |
| **PRAM**                         | Probabilistic label switching of categorical attributes via a transition matrix.                                                                                                                   | YES         | NO         |
| **Random Noise**                 | Adds noise sampled from a distribution; *decorrelated* (independent) or *correlated* (respects the covariance matrix). Typically combined with linear/non-linear transforms for adequate security. | NO          | YES        |
| **Swapping**                     | Randomly swaps sensitive attribute values between tuples. Strong identity protection, but fails to preserve statistical relationships across subdomains.                                           | NO          | YES        |
| **Rank Swapping**                | Swaps values only within a maximum rank distance $p$ (the obfuscation parameter).                                                                                                                  | NO          | YES        |
| **Micro-aggregation / Blurring** | Groups tuples into clusters of maximum similarity, computes the cluster mean of the sensitive attribute, replaces every tuple's value with that mean.                                              | YES         | YES        |

### Rounding - mathematical formulation

Given a step size $b$ and points $p_1, \dots, p_k$, each interval $I_i$ is defined as:

$$I_i = \begin{cases} [0,\ p_1 + \tfrac{b}{2}], & i = 0 \\ [p_i + \tfrac{b}{2},\ p_{i+1} + \tfrac{b}{2}), & 0 < i < k \\ [p_k,\ \max], & i = k - 1 \end{cases}$$

where $p_i$ is the label point assigned to interval $I_i$. Difference from global recoding: rounding substitutes the value with a *point inside* the interval, recoding with the *interval label*.

### PRAM - transition matrix intuition

Baseline probability of keeping the original label; semantically close categories are mapped together more often - e.g. `meat` -> `fish` with probability 0.25 (both animal products) but `meat` -> `vegetables` with only 0.05.

## 3. Synthetic Data Generation

Instead of masking real records, constructs artificial datasets.

### Fully synthetic

Generates an entirely new dataset from the estimated probabilistic distribution of the original, mirroring its structure.

- **Cholesky decomposition** (continuous data): compute the decomposition of the covariance matrix

  $$C = U^\top U$$

  then multiply $U$ by a matrix $R$ of random values whose covariance is the identity $I$. The synthetic set $U^\top R$ **preserves the original covariance** while containing only generated data.

### Partial / perturbative (partially synthetic)

Synthetic values for *some* attributes or tuples, combined with real data:

- **Random Response** - used in the **Differential Privacy** framework: the subject answers one question chosen from a set (many innocuous, one sensitive) without revealing which was chosen; the overall distribution remains estimable.
- **Blank and Impute** - erase sensitive attributes for a random subset of tuples, then refill the blanks by imputation (e.g. mean, or the most frequent category of the most similar tuples).

## 4. Comparative Reference Matrix

| Technique         | Class            | Categorical  | Continuous   | Key mechanism                               |
| ----------------- | ---------------- | ------------ | ------------ | ------------------------------------------- |
| Sampling          | Non-Perturbative | YES          | YES          | Removes a subset of records ("just a few"). |
| Local Suppression | Non-Perturbative | YES          | YES          | Replaces risky cells with null.             |
| Global Recoding   | Non-Perturbative | ONLY ORDERED | YES          | Maps values into discrete range intervals.  |
| Top/Bottom Coding | Non-Perturbative | ONLY ORDERED | YES          | Caps/floors extreme distribution tails.     |
| Generalization    | Non-Perturbative | YES          | VIA RECODING | Hierarchical abstraction of attributes.     |
| Rounding          | Perturbative     | NO           | YES          | Values -> interval midpoint/label point.    |
| Resampling        | Perturbative     | NO           | YES          | Resample, sort, average, re-map.            |
| Lossy Compression | Perturbative     | NO           | YES          | Table-as-image, JPEG-like compression.      |
| PRAM              | Perturbative     | YES          | NO           | Probabilistic label switching via matrix.   |
| Random Noise      | Perturbative     | NO           | YES          | Uncorrelated/correlated additive noise.     |
| Swapping          | Perturbative     | NO           | YES          | Random value swaps across tuples.           |
| Rank Swapping     | Perturbative     | NO           | YES          | Swaps within bounded rank distance $p$.     |
| Micro-aggregation | Perturbative     | YES          | YES          | Replace values with cluster means.          |

## 5. Exam Hooks

- Classify-before-apply: check whether the attribute is categorical or continuous first - the matrix above is the decision table.
- Every technique is a move on the desiderata trade-off from [[Microdata Disclosure Risks and Sanitization]]; the measurement of that move is in [[Privacy Evaluation Metrics]].
- Know *one* concrete example per technique (heights for top/bottom coding, meat/fish for PRAM, Cholesky for synthetic).

## Related

- [[Macrodata Protection]] - aggregate-level techniques
- [[Data Security and Privacy Fundamentals]] - what quasi-identifiers are
- [[Microdata Disclosure Risks and Sanitization]] - the threats these techniques defend against
- [[Privacy Evaluation Metrics]] - measuring what masking costs

## Sources

- [[microdati]] (AY 2026/2027)
- [[Macrodata and Microdata]] (AY 2026/2027, §5 for the comparative matrix)
