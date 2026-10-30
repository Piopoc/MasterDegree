---
type: course-note
date: 2026-10-15
tags:
  - course
  - ppia
  - k-anonymity
  - l-diversity
  - t-closeness
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[k-anonymity]]"
  - "[[L3-integration]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering the **statistical disclosure-control frameworks for microdata releases**: k-anonymity (identity-linking threat via quasi-identifiers, formal definition and equivalence classes, generalization/suppression machinery, minimality metrics, Mondrian partitioning) plus its two extensions **ℓ-diversity** and **t-closeness**, which protect the sensitive attribute itself. Created 2026-10-15 by translating the briefing [[k-anonymity]] into exam-oriented structure; extended 2026-10-28 from the microdata-privacy integration briefing (Mondrian cut/selection refinements, ℓ-diversity, t-closeness). Key points: re-identification probability $1/k$, DGH/VGH hierarchies, AG/CG x TS/AS/CS taxonomy, k-minimal generalization, median-cut recursion, homogeneity/background-knowledge attacks, entropy ℓ-diversity, EMD-based t-closeness.

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

Optimal k-anonymity enforcement is **NP-hard**; Mondrian is the standard greedy heuristic: **top-down multidimensional partitioning** by median cuts - it builds the q-blocks directly instead of searching the generalization lattice.

### Monodimensional vs multidimensional cuts

- **Monodimensional cut** - splits a single dimension uniformly across the whole dataset, ignoring cuts already applied on other dimensions.
- **Multidimensional cut** - splits a chosen dimension *conditionally within the current region*, honoring all previous cuts; Mondrian uses these, so every partition is a proper sub-rectangle of the QI space.

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

A cut is **allowable** iff both resulting children contain at least $k$ records. When no allowable cut exists, recursion stops on that partition and the **summary phase** replaces raw values with generalized bounding intervals or set summaries.

Execution logic:

1. **Termination check** - if no allowable cut exists, convert the partition to a summary region ($\varphi$).
2. **Dimension selection** - pick a QI dimension `dim` to split on.
3. **Frequency + median** - build the value distribution `fs` on `dim`, take `splitVal`.
4. **Binary partition** - `lhs` = values $\le$ split, `rhs` = values $>$ split.
5. **Recurse** on both children; return their union.

Each leaf is an equivalence class of size $\ge k$, so the output satisfies k-anonymity by construction (confidence: high - follows directly from the termination check).

### Dimension selection strategies

1. **Widest normalized span** - pick the dimension $j$ with the largest relative range inside partition $P$ compared to the full dataset $D$:

$$S_j(P) = \frac{\max(P_j) - \min(P_j)}{\max(D_j) - \min(D_j)}$$

2. **Entropy-based** - pick the dimension with the highest empirical entropy inside $P$:

$$H(X_j \mid P) = -\sum_{v} p(v) \log_2 p(v)$$

where $v$ ranges over the distinct values of $X_j$ present in $P$ and $p(v)$ is their proportion among records of $P$.

### Choosing the split value

Splitting at the median ($\mathrm{find\_median}$) is the standard heuristic. An information-loss-aware alternative maximizes the reduction of the **Normalized Certainty Penalty (NCP)**: the penalty of a partition $P$ over $d$ dimensions is

$$L(P) = |P| \sum_{j=1}^{d} w_j S_j(P)$$

with $w_j$ the importance weight of dimension $j$; the optimal split maximizes the gain

$$\mathrm{Gain} = L(P) - L(P_{L}) - L(P_{R})$$

## 7. Attacks on k-Anonymity: the ℓ-Diversity Framework

k-anonymity forces indistinguishability on the **quasi-identifiers** but places no constraint on the distribution of the **sensitive attribute** inside a q-block, so two disclosure attacks remain:

| Attack | Mechanism | Vulnerability condition |
|---|---|---|
| **Homogeneity attack** | all $k$ records in the class share the same sensitive value; the adversary learns it with 100% certainty without resolving identity | zero variance of the sensitive attribute inside the q-block |
| **Background-knowledge attack** | sensitive values are distinct, but external facts let the adversary eliminate options until a single one remains | insufficient statistical uncertainty relative to the adversary's knowledge |

> **Uninformative Disclosure Principle** - a published table should give the adversary minimal information *beyond* their background knowledge: the difference between the adversary's prior belief (before seeing the table) and posterior belief (after) about a subject's sensitive attribute must be bounded and small.

### Distinct ℓ-diversity

> A q-block satisfies **distinct ℓ-diversity** iff it contains at least $\ell$ distinct values of the sensitive attribute; a table satisfies it iff every one of its q-blocks does.

**Weakness:** it only counts *unique* categories, so skewed distributions slip through - a class of 100 records with 96 *Flu*, 2 *Cold*, 2 *Bronchitis* satisfies distinct 3-diversity, yet an adversary predicts *Flu* with 96% probability.

### Entropy ℓ-diversity

Let $S$ be the domain of the sensitive attribute and $p(s,E)$ the probability of observing value $s$ inside class $E$. The class satisfies **entropy ℓ-diversity** iff

$$\mathrm{Entropy}(E) = -\sum_{s \in S} p(s,E)\log p(s,E) \ge \log(\ell)$$

Entropy ℓ-diversity eliminates frequency imbalance and enforces high diversity, but it is highly conservative and can severely reduce data utility in practice (confidence: stated).

### Checking ℓ-diversity in three steps

1. **Group** records into q-blocks by QI tuple and count block sizes.
2. **Distinct check** - count distinct sensitive values per block; any block with fewer than $\ell$ distinct values fails distinct ℓ-diversity.
3. **Entropy check** - convert counts to $p(s,E) = \mathrm{count}(s)/|E|$, compute $\mathrm{Entropy}(E)$ and compare against $\log_2 \ell$; any failing block requires further generalization or suppression.

## 8. Limitations of ℓ-Diversity: the t-Closeness Framework

Even ℓ-diverse classes remain open to the **similarity attack**: sensitive values are distinct but semantically close (or numerically adjacent), so identifying someone inside the class still reveals a *narrow range* of the attribute.

*Example:* income values \$3k, \$4k, \$5k satisfy both distinct and entropy 3-diversity, yet an adversary who isolates the target learns with 100% certainty that the individual has an extremely low income, even without pinning the exact value.

### t-Closeness

> An equivalence class $E$ satisfies **$t$-closeness** iff the distance between the distribution of the sensitive attribute *inside $E$* and its distribution *in the whole table* does not exceed threshold $t$; a table satisfies $t$-closeness iff every one of its classes does.

Two structural properties:

- **Generalization property** - if a table satisfies $t$-closeness under a generalization $B$, it automatically satisfies it under any more general transformation $A$.
- **Subset property** - $t$-closeness with respect to an attribute set $C$ implies $t$-closeness with respect to any subset $D \subset C$.

### Earth Mover's Distance (EMD)

The local-vs-global distance is measured with the **Earth Mover's Distance** - the minimum "work" required to reshape the local distribution $P = (p_1,\dots,p_m)$ of a class into the global distribution $Q = (q_1,\dots,q_m)$ of the table:

| Domain type | EMD formulation | Operational meaning |
|---|---|---|
| Categorical | $$D[P,Q] = \frac{1}{2}\sum_{i=1}^{\lvert S\rvert}\lvert p_i - q_i\rvert$$ | $L_1$ distance over unordered categories (unit inter-category distance) |
| Numerical discrete | $$D[P,Q] = \frac{1}{\lvert S\rvert - 1}\sum_{i=1}^{\lvert S\rvert}\left\lvert\sum_{j=1}^{i} r_j\right\rvert,\quad r_i = p_i - q_i$$ | accumulated probability-mass difference over the ordered domain |
| Numerical continuous | $$D[P,Q] = \int_{S}\lvert F_P(x) - F_Q(x)\rvert\,dx$$ | area between the two cumulative distribution functions |

A tighter $t$ means stronger protection but exponentially harder computation - the efficiency/utility side of the trade-off quantified in the evaluation note listed under Related.

## 9. Exam Hooks

- "Sanitization is not enough" is the standard opener: **quasi-identifiers + external data = identity linking**, regardless of PII removal (see Microdata Disclosure Risks and Sanitization).
- k-anonymity limits **identity disclosure only**; attribute disclosure requires the ladder **k-anonymity → ℓ-diversity → t-closeness**, each model closing the previous one's attack: homogeneity and background-knowledge attacks → similarity attack.
- Remember the two mechanisms and their granularities: generalization AG/CG, suppression TS/AS/CS.
- Mondrian = greedy top-down multidimensional median cuts; allowable cut = both children $\ge k$; dimension chosen by widest normalized span or entropy; split at the median, or at the NCP-maximizing gain.
- The $1/k$ probability is the number to quote for "how much privacy does k provide".
- Distinct vs entropy ℓ-diversity: the 96/2/2 *Flu* example is the standard counterexample to distinct ℓ-diversity.
- t-closeness in one phrase: "local sensitive distribution kept close to the global one, distance measured with EMD".

## Related

- [[Data Security and Privacy Fundamentals]] - QI taxonomy
- [[Microdata Disclosure Risks and Sanitization]] - threat side
- [[Microdata Masking and Synthetic Data]] - perturbative alternative
- [[Macrodata Protection]] - aggregate-level protection
- [[Privacy Evaluation Metrics]] - measuring the utility cost

## Sources

- [[k-anonymity]] (AY 2026/2027)
- [[L3-integration]] (AY 2026/2027)
