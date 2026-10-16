---
type: course-note
date: 2026-10-15
tags:
  - course
  - ppia
  - k-anonymity
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[k-anonymity]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering the **k-anonymity** privacy model: the identity-linking threat via quasi-identifiers, the formal definition and equivalence classes, the generalization/suppression machinery, minimality metrics (distance vectors, MaxSup, selection criteria), and the Mondrian partitioning algorithm. Created 2026-10-15 by translating the briefing [[k-anonymity]] into exam-oriented structure. Key points: re-identification probability $1/k$, DGH/VGH hierarchies, AG/CG x TS/AS/CS taxonomy, k-minimal generalization, median-cut recursion.

## 1. Threat Model: Why De-Identification Fails

Removing direct PII (names, SSNs) is **not enough**. Two drivers make re-identification practical:

1. **Large volumes of public information** - open data sources and social networks.
2. **High computational power** - large-scale data fusion and record matching.

### Record Linkage vs Identity Linking

- **Record Linkage** - matching records across datasets that refer to the same individual, without revealing the real-world identity.
- **Identity Linking** - the dangerous subset where at least one linked record carries a direct identifier (PII), fully de-anonymizing the subject.

```
+-------------------------------------------------------+
| Record Linkage                                        |
| (Linking multiple records of the same individual)     |
|   +-----------------------------------------------+   |
|   | Identity Linking (direct identifiers ->       |   |
|   | complete de-anonymization)                    |   |
|   +-----------------------------------------------+   |
+-------------------------------------------------------+
```

### Quasi-Identifiers (QIs)

Attributes that exist in the private table **and** in external public sources, enabling linking attacks. Common examples: Race, Date of Birth, Sex, ZIP Code, Marital Status (as defined in [[Data Security and Privacy Fundamentals]]).

- A single QI value rarely identifies anyone; **combining** several QIs creates a high-dimensional signature that isolates one row.
- Exemplar: knowing that "Louise Wayland" was born on `64/09/27`, lives in a specific ZIP, and is a `widow` isolates her row and exposes her `Income` ($56,000) - even with name and SSN removed.

This is exactly the **Identity Disclosure** channel described in [[Microdata Disclosure Risks and Sanitization]].

## 2. Formal Definition

> A table $T(A_1,\dots,A_m)$ with quasi-identifier set $\mathrm{QI}$ satisfies **k-anonymity** iff every value combination in the projection $T[\mathrm{QI}]$ appears at least $k$ times in $T[\mathrm{QI}]$.

- **Equivalence class (q-block):** the set of tuples sharing identical QI values. Under k-anonymity every q-block has size $\ge k$.
- **Re-identification probability** for an individual inside a compliant q-block: $\frac{1}{k}$.

### Choosing k (privacy-utility dial)

| k | Effect |
|---|---|
| $k = 1$ | zero protection - direct record isolation |
| $k = 2$ | 50% chance of correct identification in an attack |
| $k = 5$ | practical starting threshold |
| $k = \lvert T \rvert = n$ | complete privacy, zero utility (all records collapse into one group) |

The optimum depends on the dataset and operational requirements (confidence: stated).

## 3. Enforcement Mechanism 1: Generalization

Generalization replaces specific QI values with broader categories, preserving truthfulness (no invented data).

- **Domain ($\mathrm{Dom}$):** all possible values of an attribute; a generalized domain unions primitive domains.
- **Domain Generalization Hierarchy ($\mathrm{DGH}_D$):** partial order $\le_D$ over domains satisfying:
  1. *Condition 1 (comparability):* $\forall D_i,D_j,D_z \in \mathrm{Dom}: (D_i \le_D D_j \land D_i \le_D D_z) \Rightarrow (D_j \le_D D_z \lor D_z \le_D D_j)$
  2. *Condition 2 (convergence):* all maximal elements are singletons - structurally a tree where every path converges to one root value.
- **Value Generalization Hierarchy ($\mathrm{VGH}_D$):** maps each value in $D_i$ to a unique direct generalization in $D_j$.

### Granularity

- **Attribute Generalization (AG):** whole column generalized uniformly.
- **Cell Generalization (CG):** per-cell generalization; one attribute can hold mixed levels.
- **Sensitive attributes:** generalize *only* if they also act as QIs - otherwise utility collapses for nothing.

## 4. Enforcement Mechanism 2: Suppression

Rare values/outliers can force extreme whole-table generalization; suppression removes the offenders instead.

- **Tuple Suppression (TS):** remove entire rows.
- **Attribute Suppression (AS):** remove entire columns.
- **Cell Suppression (CS):** blank individual cells (null).

### Combined Taxonomy (generalization x suppression)

| Generalization \ Suppression | TS | AS | CS | none |
|---|---|---|---|---|
| **Attribute Generalization (AG)** | `AG_TS` | `AG_AS` | `AG_CS` | `AG_` |
| **Cell Generalization (CG)** | `CG_TS` | `CG_AS` | `CG_CS` | `CG_` |
| **No Generalization (_)** | `_TS` | `_AS` | `_CS` | N/A |

(The `_` column means no suppression; the `_` row means no generalization.)

## 5. Minimality: Keeping Utility

### Order on tables

$T_i \preceq T_j$ ("$T_j$ is a generalization with tuple suppression of $T_i$") iff:

1. $|T_j| \le |T_i|$,
2. for each attribute $A$, $\mathrm{dom}(A,T_j)$ equals or generalizes $\mathrm{dom}(A,T_i)$,
3. an injective map sends each $t_j \in T_j$ to some $t_i \in T_i$ whose attribute values equal or generalize $t_j$'s.

### Distance vector

$$\mathrm{DV}_{i,j} = [d_1, d_2, \dots, d_n]$$

where $d_z$ = length of the path between $\mathrm{dom}(A_z,T_i)$ and $\mathrm{dom}(A_z,T_j)$ in the corresponding $\mathrm{DGH}_{D_z}$ - one component per attribute, measuring how far each column was generalized.

### k-minimal generalization (given MaxSup)

$T_j$ is a **k-minimal generalization** of $T_i$ iff:

1. **Minimal suppression:** $T_j$ satisfies k-anonymity, and among tables at the *same* distance $\mathrm{DV}_{i,z} = \mathrm{DV}_{i,j}$ that satisfy k-anonymity, none keeps more tuples: $|T_j| \ge |T_z|$ for all such $T_z$,
2. **Threshold bound:** $|T_i| - |T_j| \le \mathrm{MaxSup}$,
3. **Distance minimality:** no admissible $T_z$ (satisfying 1-2) has $\mathrm{DV}_{i,z} < \mathrm{DV}_{i,j}$.

### Four selection criteria among valid minimal candidates

| Criterion | Prefers |
|---|---|
| Minimum absolute distance | smallest total number of generalization steps across attributes |
| Minimum relative distance | steps normalized by each hierarchy's total height |
| Maximum distribution | table retaining the most distinct tuples |
| Minimum suppression | fewest suppressed tuples |

## 6. Algorithmic Enforcement: Mondrian

Greedy **top-down multidimensional partitioning** by median cuts - it builds the q-blocks directly instead of searching the generalization lattice.

```text
Anonymize(partition)
    if (no allowable multidimensional cut for partition)
        return φ : partition -> summary
    else
        dim <- choose_dimension()
        fs <- frequency set(partition, dim)
        splitVal <- find_median(fs)
        lhs <- {t ∈ partition : t.dim ≤ splitVal}
        rhs <- {t ∈ partition : t.dim > splitVal}
        return Anonymize(rhs) U Anonymize(lhs)
```

Execution logic:

1. **Termination check** - if no cut keeps every child partition at size $\ge k$, stop and convert the partition to a summary region ($\varphi$).
2. **Dimension selection** - pick a QI dimension `dim` to split on.
3. **Frequency + median** - build the value distribution `fs` on `dim`, take `splitVal`.
4. **Binary partition** - `lhs` = values $\le$ split, `rhs` = values $>$ split.
5. **Recurse** on both children; return their union.

Each leaf is an equivalence class of size $\ge k$, so the output satisfies k-anonymity by construction (confidence: high - follows directly from the termination check).

## 7. Exam Hooks

- "Sanitization is not enough" is the standard opener: **quasi-identifiers + external data = identity linking**, regardless of PII removal (see Microdata Disclosure Risks and Sanitization).
- k-anonymity limits **identity disclosure only** - it says nothing about attribute/inferential disclosure (homogeneous q-blocks still leak sensitive values). That gap is the motivation for l-diversity/t-closeness (not covered by the source briefing - TBD if in course).
- Remember the two mechanisms and their granularities: generalization AG/CG, suppression TS/AS/CS.
- Mondrian = greedy median cuts, top-down, O(n log n)-style partitioning vs exhaustive lattice search; the answer to "how is k-anonymity enforced algorithmically".
- The $1/k$ probability is the number to quote for "how much privacy does k provide".

## Related

- [[Data Security and Privacy Fundamentals]] - QI taxonomy
- [[Microdata Disclosure Risks and Sanitization]] - threat side
- [[Microdata Masking and Synthetic Data]] - perturbative alternative
- [[Macrodata Protection]] - aggregate-level protection
- [[Privacy Evaluation Metrics]] - measuring the utility cost

## Sources

- [[k-anonymity]] (AY 2026/2027)
