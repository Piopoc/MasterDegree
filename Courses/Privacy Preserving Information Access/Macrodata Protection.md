---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - macrodata
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Macrodata and Microdata]]"
  - "(see -> maps/NotebookLM Mind Map.png)"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering protection of **macrodata** (aggregated tabular releases): sensitive-cell identification, suppression, rounding, CTA, and the magnitude-table rules. Created 2026-10-09 from §2 of [[Macrodata and Microdata]] plus the (see -> maps/NotebookLM Mind Map.png). Key points: three release paradigms, count/frequency protections, the $(n,k)$-rule and $p$-percentage rule in LaTeX.

## 1. Data Release Paradigms

Data is released across three structural formats depending on operational needs (mandatory statistical reporting, marketing, research):

```
              ┌─────────────────────────┐
              │    Data Release Form    │
              └────────────┬────────────┘
                           │
    ┌──────────────────────┼──────────────────────┐
    ▼                      ▼                      ▼
Macrodata          Statistical Databases      Microdata
(Aggregated)       (API / Distributional)     (Individual records)
```

- **Macrodata** -> this note.
- **Microdata** -> [[Microdata Disclosure Risks and Sanitization]].
- **Statistical databases** -> section 4 below.

## 2. Macrodata: Characteristics and Weaknesses

Macrodata consists of aggregated data presented in tabular format:

- **Table categories:** counts/frequencies (statistics about quantity/totals) and magnitudes.
- **Weaknesses:** information about *rare subjects* can still be inferred from aggregate numbers; low flexibility and limited information availability.
- **Protection approach:** selective obfuscation of sensitive cells.

### Sensitive cells in count & frequency tables

A **sensitive cell** exposes too much information by isolating or identifying a small number of subjects. Threshold rules set explicit cutoffs - e.g. a count threshold of 2: any cell below it is deemed sensitive.

## 3. Protection Techniques for Counts and Frequencies

| Technique | Mechanism |
|---|---|
| **Primary Suppression** | Direct removal of cells below the threshold. |
| **Secondary Suppression** | Removal of *additional* non-sensitive cells, chosen via **linear programming** so attackers cannot recompute the primary values from marginal totals. |
| **Rounding** | Round values to the closest multiple of a chosen base; particularly effective against record isolation. |
| **Category Roll-up** | Combine fine-grained categories into broader macro-categories, raising the subject count per cell. |
| **Sampling** | Move from full census to sample surveys. |
| **Controlled Tabular Adjustment (CTA)** | Replace lower values with the target threshold, then adjust remaining cells via linear programming to preserve table sums. Sacrifices exact cell utility to enforce threshold constraints. |

## 4. Sensitive Cells in Magnitude Tables

For tables of continuous magnitudes (economic output, total income), two formal rules apply:

### The $(n, k)$-rule

A cell is **sensitive if fewer than $n$ subjects contribute at least $k\%$ of the cell's total value**.

- Under a $(2, 50)$-rule: sensitive if a single user contributes over 50% of the total.
- Under a $(3, 30)$-rule: if fewer than 3 individuals contribute over 30% each (e.g. 2 individuals at 50% each), the cell is sensitive.

### The $p$-percentage rule

Let $t$ be the cell's total value, $v_1$ the largest individual contribution and $v_2$ the second largest. With security parameter $p < 100$, the cell is **sensitive** if:

$$t - v_1 - v_2 < \frac{p}{100} \cdot v_1$$

If the inequality holds, an adversary can easily estimate $v_1$ from the published total, so the cell must be protected.

## 5. Statistical Databases

Databases that restrict query capability to **distributional queries** via an API:

- **Weaknesses:** reconstruction attacks and differencing attacks; low flexibility.
- **Protection techniques:** query rate limiting, result perturbation, and **Differential Privacy**.

## 6. Exam Hooks

- Why is secondary suppression needed at all? Because marginals let an attacker solve for the suppressed cell - suppression must be *consistent* across the whole table (linear programming).
- CTA vs. suppression: CTA keeps every cell filled but perturbs values; suppression removes cells but keeps the rest exact.
- Know both magnitude rules by heart and be able to apply them to a numeric example - see [[Privacy Evaluation Metrics]] for how the resulting protection/utility trade-off is measured.

## Related

- [[Microdata Masking and Synthetic Data]] - record-level techniques
- [[Privacy Evaluation Metrics]] - measuring confidentiality and information loss
- [[Microdata Disclosure Risks and Sanitization]] - why release control is needed at all

## Sources

- [[Macrodata and Microdata]] (AY 2026/2027)
- (see -> maps/NotebookLM Mind Map.png)
