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
  - "[[L3-integration]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering **evaluation metrics**: how confidentiality, information loss, and computational cost of a protected release are measured. Created 2026-10-09 from the *Evaluation Metrics* branch of the (see -> maps/NotebookLM Mind Map.png) at name/category level; extended 2026-10-28 with the formal anonymization metrics of the microdata-privacy integration briefing. Key points: the three operational dimensions, average q-block size, population uniqueness, information-loss metrics per data type, the Discernibility Metric, and the R-U confidentiality map.

> **Confidence notice:** the formal metrics below (average equivalence-class size, population-uniqueness formula, complexity/scaling, Discernibility Metric) were added 2026-10-28 from the extended integration briefing. Everything else - the record-linkage approaches, the explicit MSE/MAE/entropy formulas, the R-U map construction - remains `stated` at the *name/category* level by the mind map only, since the accompanying markdown sources ([[Macrodata and Microdata]], [[microdati]]) do not cover this topic. Verify against the full lecture material before exams.

## 1. Why Evaluation Metrics Exist

Every technique in [[Microdata Masking and Synthetic Data]] trades **disclosure risk** against **statistical utility** (the desiderata in [[Microdata Disclosure Risks and Sanitization]]). Evaluation metrics quantify *both sides* of that trade-off so a release can be compared objectively.

Two questions:

1. **Confidentiality** - how likely is re-identification/disclosure now?
2. **Information loss** - how much analytic utility did the protection destroy?

The extended briefing frames every evaluation along three competing operational dimensions:

| Dimension | Primary metric | Trade-off dynamics |
|---|---|---|
| **Computational efficiency** | processing time vs $\lvert \mathrm{QI} \rvert$ | enforcement is NP-hard; runtime scales non-linearly as constraints tighten (e.g. $t$-closeness from $0.2$ to $0.15$) |
| **Re-identification risk** | average q-block size, population uniqueness | larger equivalence classes reduce the $1/k$ re-identification probability but increase information loss |
| **Information loss / utility** | DM, MSE/MAE, ML degradation | minimal generalization maximizes utility but raises disclosure risk for rare tuples |

## 2. Confidentiality Measurement

| Metric | What it captures |
|---|---|
| **Population Uniqueness** | The probability that a given combination of values is *unique in the population* - uniqueness is exactly what enables identity disclosure. |
| **Record Linkage** | Estimates how well an attacker could link released records back to individuals; approaches: **deterministic**, **probabilistic**, and **distance-based** (mind-map level only). |
| **Average equivalence-class size** | The average cardinality of the q-blocks in the anonymized table $T^*$ - larger blocks mean lower re-identification risk, at the cost of more information loss. |

Formal versions (added 2026-10-28 from the extended briefing):

$$\mathrm{Avg}\,\lvert q\text{-block}\rvert = \frac{\lvert T^* \rvert}{\lvert EQ(T^*) \rvert}$$

where $\lvert EQ(T^*)\rvert$ is the total number of equivalence classes in $T^*$, and

$$\mathrm{Pr}(PU) = \sum_{j} \frac{I(F_j = 1)}{N}$$

where $F_j$ is the frequency of QI combination $j$ in the population of size $N$, and $I(\cdot)$ is the indicator function ($1$ iff $F_j = 1$).

## 3. Information Loss

Metrics grouped by data type:

| Data type | Metrics |
|---|---|
| **Continuous data** | MSE (Mean Squared Error), MAE (Mean Absolute Error), Mean Variation - pre- vs post-anonymization statistical summaries. |
| **Categorical data** | Cell-change counts, category distance tracking, contingency-table comparison, Shannon entropy reduction. |
| **Machine learning** | Decrease in model performance metrics after protection (accuracy, precision, recall, F1) - utility measured downstream. |

## 4. The Discernibility Metric (DM)

The **Discernibility Metric** penalizes each record according to how indistinguishable it is in the anonymized table $T$. For a record $t$ whose equivalence class is $E_t$:

$$\mathrm{penalty}(t) = \begin{cases} \lvert T \rvert & \text{if } t \text{ is suppressed} \\ \lvert E_t \rvert & \text{otherwise} \end{cases}$$

$$\mathrm{discernibility}(T) = \sum_{t \in T} \mathrm{penalty}(t)$$

Suppressed records incur the maximum penalty ($\lvert T\rvert$) because they are completely lost to analysis; unsuppressed records incur the size of their own equivalence class - the degree of ambiguity introduced. **Lower scores indicate higher data utility** (confidence: high - the definition is self-contained in the source).

## 5. The R-U Confidentiality Map

The **Risk-Utility (R-U) confidentiality map** plots the trade-off between disclosure risk and utility: each protection configuration of a dataset becomes a point, letting you visualize how much risk is reduced per unit of information lost.

Conceptually it operationalizes the two desiderata from Microdata Disclosure Risks and Sanitization into a single plane - the standard way to pick *how much* protection a release needs.

## 6. Open Items (TBD)

- Formalization of the three record-linkage approaches (deterministic, probabilistic, distance-based).
- Formulas for MSE/MAE/mean variation and the entropy-based categorical measures as used in this course.
- How the R-U map is constructed concretely for a given technique.

If you find these in later lectures or slides, extend this note and drop this section (source: future lecture material - TBD).

## Related

- [[k-Anonymity]] - whose privacy-utility trade-off these metrics quantify
- [[Microdata Masking and Synthetic Data]] - the techniques whose cost this note measures
- [[Microdata Disclosure Risks and Sanitization]] - the disclosure side of the R-U trade-off
- [[Macrodata Protection]] - protection configurations plotted on the R-U map

## Sources

- (see -> maps/NotebookLM Mind Map.png) - evaluation-metrics branch (mind-map-level items; see the confidence notice above)
- [[L3-integration]] (AY 2026/2027)
