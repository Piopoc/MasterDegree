---
type: course
date: 2026-10-09
tags:
  - course
  - ppia
ai-first: true
---

## For Future Agent

Course index for **Privacy Preserving Information Access (PPIA)** - the hub note linking all study notes for this course. Created 2026-10-09 when the vault's `Courses/` structure was bootstrapped from `raw/` material. The course identity (University of Padua, AY 2026/2027, taught by Guglielmo Faggioli) is stated in [[Macrodata and Microdata]]. Related: [[Advanced Text Analytics]], [[Game Theory]], [[Graph Databases]].

## Course Identity

| Field | Value |
|---|---|
| Course | Privacy Preserving Information Access |
| Institution | University of Padua |
| Academic Year | 2026/2027 |
| Instructor | Guglielmo Faggioli |
| Status | active |

## Notes

1. [[Data Security and Privacy Fundamentals]] - data/attribute taxonomy, security vs. privacy, GDPR and legal framework, privacy as evolutionary need.
2. [[Database Foundations and Architecture]] - DB/DBMS/UoD, ANSI/SPARC, design phases, relational model, platform case studies.
3. [[SQL Injection and Defenses]] - SQLi anatomy, OWASP history, PreparedStatement, tautology vs. DDL attacks.
4. [[Macrodata Protection]] - release paradigms, sensitive cells, suppression/rounding/CTA, $(n,k)$-rule and $p$-percentage rule.
5. [[Microdata Disclosure Risks and Sanitization]] - identity/attribute/inferential disclosure, outliers, natural defenses, desiderata.
6. [[Microdata Masking and Synthetic Data]] - full technique catalog (non-perturbative, perturbative, synthetic) with applicability matrix.
7. [[Privacy Evaluation Metrics]] - confidentiality measurement (avg q-block size, population uniqueness), information-loss metrics, Discernibility Metric, R-U map.
8. [[k-Anonymity]] - identity-linking threat, formal definition and q-blocks, generalization/suppression, k-minimality metrics, Mondrian algorithm, plus the ℓ-diversity and t-closeness extensions.
9. [[Information Retrieval Foundations]] - DB vs IR semantics, the "Y" architecture, preprocessing and inverted indexes, TF-IDF/VSM/BM25, Cranfield evaluation with P/R/F and DCG/nDCG.
10. [[Recommender Systems]] - paradox of choice and decision fatigue, user-as-query architecture, rating matrix, content-based vs collaborative filtering taxonomy, evaluation framings (P/R/F1, nDCG/AP, AUC, RMSE).
11. [[Differential Privacy Foundations]] - Part I: Fundamental Law of Information Recovery, differencing attack and failure of query auditing, randomized response with $p = 2Y - 1/2$, simplex and randomized algorithms, histograms and neighbouring databases, $(\varepsilon,\delta)$-DP definition, choosing $\varepsilon$ and $\delta$, privacy loss, composition, group privacy, post-processing.
12. [[Differential Privacy Mechanisms]] - Part II: $\ell_1$-sensitivity, Laplace mechanism (proof + accuracy bound), count/histogram queries and adjacency, 2010-census first-names example, Report Noisy Max, Gaussian mechanism and $\ell_2$-sensitivity, exponential mechanism via the auction example, sparse vector technique (AboveThreshold, Sparse, NumericSparse).
