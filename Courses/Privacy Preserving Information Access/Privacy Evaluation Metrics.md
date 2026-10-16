---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - evaluation
course: "[[Privacy Preserving Information Access]]"
sources:
  - "(see -> maps/NotebookLM Mind Map.png)"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering **evaluation metrics**: how confidentiality and information loss of a protected release are measured. Created 2026-10-09 from the *Evaluation Metrics* branch of the (see -> maps/NotebookLM Mind Map.png) - this branch exists **only** in the mind map, not in the markdown sources, so detail here is mind-map-level only. Key points: population uniqueness, record-linkage approaches, information-loss metrics per data type, and the R-U confidentiality map.

> **Confidence notice:** everything below is `stated` at the *name/category* level by the mind map. Formal definitions, formulas, and worked examples are **TBD** - the accompanying markdown sources ([[Macrodata and Microdata]], [[microdati]]) do not cover this topic. Verify against the full lecture material before exams.

## 1. Why Evaluation Metrics Exist

Every technique in [[Microdata Masking and Synthetic Data]] trades **disclosure risk** against **statistical utility** (the desiderata in [[Microdata Disclosure Risks and Sanitization]]). Evaluation metrics quantify *both sides* of that trade-off so a release can be compared objectively.

Two questions:

1. **Confidentiality** - how likely is re-identification/disclosure now?
2. **Information loss** - how much analytic utility did the protection destroy?

## 2. Confidentiality Measurement

| Metric | What it captures (per mind map) |
|---|---|
| **Population Uniqueness** | The probability that a given combination of values is *unique in the population* - uniqueness is exactly what enables identity disclosure. |
| **Record Linkage** | Estimates how well an attacker could link released records back to individuals; approaches: **deterministic**, **probabilistic**, and **distance-based**. |

## 3. Information Loss

Metrics grouped by data type:

| Data type | Metrics |
|---|---|
| **Continuous data** | MSE (Mean Squared Error), MAE (Mean Absolute Error), Mean Variation. |
| **Categorical data** | Direct comparison, contingency tables, Shannon entropy. |
| **Machine learning** | Decrease in model performance metrics after protection (utility measured downstream). |

## 4. The R-U Confidentiality Map

The **Risk-Utility (R-U) confidentiality map** plots the trade-off between disclosure risk and utility: each protection configuration of a dataset becomes a point, letting you visualize how much risk is reduced per unit of information lost.

Conceptually it operationalizes the two desiderata from Microdata Disclosure Risks and Sanitization into a single plane - the standard way to pick *how much* protection a release needs.

## 5. Open Items (TBD)

- Formal definitions of population uniqueness and the three record-linkage approaches.
- Formulas for MSE/MAE/mean variation and the entropy-based categorical measures as used in this course.
- How the R-U map is constructed concretely for a given technique.

If you find these in later lectures or slides, extend this note and drop this section (source: future lecture material - TBD).

## Related

- [[Microdata Masking and Synthetic Data]] - the techniques whose cost this note measures
- [[Microdata Disclosure Risks and Sanitization]] - the disclosure side of the R-U trade-off
- [[Macrodata Protection]] - protection configurations plotted on the R-U map

## Sources

- (see -> maps/NotebookLM Mind Map.png) - evaluation-metrics branch (not covered by the markdown sources; see the confidence notice above)
