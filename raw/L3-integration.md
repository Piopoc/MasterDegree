# Microdata Privacy Frameworks: A Comprehensive Technical Breakdown of k-Anonymity, ℓ-Diversity, and t-Closeness

## 1. Foundations of Microdata Privacy & Identity Attacks

Data privacy in microdata releases—datasets containing granular records of individual respondents—requires protecting subjects from re-identification while preserving the statistical utility of the published dataset. A foundational premise in privacy-preserving information access is that basic **sanitizing** or **de-identification** (the simple removal of explicit personal identifiers such as Name, Social Security Number, or Address) is insufficient to grant true anonymity. 

Modern data ecosystems expose microdata to sophisticated linkage risks due to the vast availability of external public records, social media data, and high-performance computing capabilities that enable automated cross-referencing.

> **Definition — Identity Linking Attack:** An attack wherein an adversary fuses multiple data sources to aggregate information about the same individual, thereby de-anonymizing data that was previously subjected to basic sanitization.

To analyze data linking vulnerabilities, privacy frameworks distinguish between two fundamental forms of linkage:

*   **Record Linkage:** The process of identifying and linking multiple records across disparate databases that correspond to the same underlying individual, even if explicit personal identifiers are absent.
*   **Identity Linking:** A specific, critical instance of record linkage where at least one of the linked records contains explicit direct identifiers (Personally Identifiable Information, or PII), thereby fully revealing the real-world identity of the subject associated with the remaining private attributes.

| Linkage Paradigm | Scope & Mechanism | Outcome |
| :--- | :--- | :--- |
| **Record Linkage** | Matches records across tables based on shared non-explicit attributes | Associates records belonging to the same entity without necessarily knowing PII |
| **Identity Linking** | Connects sanitized target microdata with an external database containing PII | Fully de-anonymizes the target subject and exposes sensitive private attributes |

### Quasi-Identifiers (QIs)

The feasibility of an identity linking attack depends on attributes that are non-unique individually but become uniquely identifying when combined.

> **Definition — Quasi-Identifiers (QI):** A set of attributes $QI = \{A_1, A_2, \dots, A_k\}$ contained within a private table that, while not directly revealing an individual's identity on their own, are externally available or ascertainable by an adversary and can be exploited in combination to link private records to specific individuals.

In typical medical or demographic microdata, attributes such as *Race*, *Date of Birth (DoB)*, *Sex*, *ZIP Code*, and *Marital Status* act as quasi-identifiers. An adversary possessing external knowledge of an individual's birth date, gender, and postal code can isolate that individual's record in a sanitized medical table and uncover sensitive diagnostic or financial information.

---

## 2. The k-Anonymity Framework

To prevent identity linking attacks, privacy-preserving systems enforce statistical indistinguishability over the set of quasi-identifiers. The primary model for achieving this is **k-anonymity**.

> **Definition — k-Anonymity Requirement:** A dataset release satisfies $k$-anonymity if and only if every combination of quasi-identifier values present in the released table matches at least $k$ distinct respondents within the dataset.

Formally, let $T(A_1, A_2, \dots, A_m)$ be a relational table and $QI \subseteq \{A_1, \dots, A_m\}$ be the associated set of quasi-identifiers. $T$ satisfies $k$-anonymity with respect to $QI$ if and only if each unique sequence of values in the projection $T[QI]$ appears with a frequency of at least $k$ in $T[QI]$.

> **Definition — Equivalence Class (q-block):** A subset of records in a table $T$ that share identical values across all designated quasi-identifiers $QI$.

### Probability of Re-Identification & Parameter Choice

Enforcing $k$-anonymity guarantees that an adversary who knows the quasi-identifier values of a target individual cannot isolate that individual's record to a group smaller than $k$. Consequently, the maximum probability of successfully de-anonymizing an individual within a $q$-block is bounded by:

$$P(\text{Re-identification}) = \frac{1}{k}$$

*   **Bound $k = 1$:** Represents an un-anonymized dataset where records can be unique, leading directly to record isolation violations.
*   **Bound $k = N$ (where $N = |T|$):** Represents total generalization where all records in the dataset are rendered completely identical across quasi-identifiers. While privacy protection is absolute, data utility is entirely destroyed.
*   **Operational Setting ($k = 2$ vs $k = 5$):** Setting $k = 2$ leaves a $50\%$ probability of re-identification under an identity attack. In practice, $k = 5$ is frequently adopted as a balanced threshold, though the optimal choice depends strictly on dataset granularity and domain risk tolerances.

---

## 3. Mechanisms for Achieving k-Anonymity: Generalization & Suppression

To transform an un-anonymized table into a table that satisfies $k$-anonymity while preserving data truthfulness (avoiding false data generation), two primary non-perturbative operations are used: **Generalization** and **Suppression**.

### Generalization Framework

Generalization replaces specific quasi-identifier values with broader, less specific categories derived from an underlying domain hierarchy.

> **Definition — Domain Generalization Hierarchy ($DGH_D$):** A domain generalization relationship $\le_D$ defines a partial ordering between attribute domains such that for domains $D_i, D_j, D_z \in \mathbf{Dom}$:
> 1. $\forall D_i, D_j, D_z \in \mathbf{Dom}: D_i \le_D D_j \land D_i \le_D D_z \implies D_j \le_D D_z \lor D_z \le_D D_j$
> 2. All maximal elements of $\mathbf{Dom}$ are singletons (representing the single most generalized state where all domain values map to a single universal representation).
>
> Conditions 1 and 2 establish a totally ordered hierarchy of domains for each attribute, termed the Domain Generalization Hierarchy ($DGH_D$).

A corresponding **Value Generalization Hierarchy ($VGH_D$)** is induced by a value generalization relationship $\le_V$, which maps each specific value in domain $D_i$ to a unique generalized parent value in domain $D_j$.

Generalization can be applied at two structural granularities:
*   **Attribute Generalization (AG):** All values within a given column/attribute are generalized to the exact same level in the domain hierarchy across the entire table.
*   **Cell Generalization (CG):** Generalization is applied independently at the cell level, allowing a column to contain entries at varying levels of generalization.

While generalizing quasi-identifiers is necessary, generalizing sensitive attributes is generally avoided unless the sensitive attribute also functions as a quasi-identifier, as doing so rapidly destroys domain utility.

### Suppression Framework

Outliers in microdata can force severe over-generalization across an entire table to satisfy $k$-anonymity for a few rare tuples. To prevent this utility loss, outlier tuples are removed via **suppression**.

Suppression operates across three structural granularities:
*   **Tuple Suppression (TS):** Complete removal of an entire record/row from the published table.
*   **Attribute Suppression (AS):** Complete removal of an entire column/attribute from the published table.
*   **Cell Suppression (CS):** Slighting specific cell values by replacing them with a missing or null value indicator.

Combinations of generalization and suppression yield a design space represented by a $3 \times 4$ operational matrix:

| Generalization / Suppression | Tuple Suppression (TS) | Attribute Suppression (AS) | Cell Suppression (CS) | No Suppression (_) |
| :--- | :--- | :--- | :--- | :--- |
| **Attribute Generalization (AG)** | `AG_TS` | `AG_AS` | `AG_CS` | `AG_` |
| **Cell Generalization (CG)** | `CG_TS` | `CG_AS` | `CG_CS` | `CG_` |
| **No Generalization (_)** | `_TS` | `_AS` | `_CS` | — |

### Formalization of k-Minimal Generalization

Let $T_i$ and $T_j$ be two tables defined over the same attribute set. $T_j$ is a generalization with tuple suppression of $T_i$ (denoted $T_i \preceq T_j$) if:
1.  $|T_j| \le |T_i|$ (due to tuple suppression).
2.  The domain $\text{dom}(A, T_j)$ of each attribute $A$ in $T_j$ is equal to or a generalization of $\text{dom}(A, T_i)$ in $T_i$.
3.  There exists an injective mapping associating each tuple $t_j \in T_j$ with a tuple $t_i \in T_i$ such that every attribute value in $t_j$ is equal to or a generalization of the corresponding value in $t_i$.

> **Definition — Distance Vector ($DV_{i,j}$):** Given $T_i(A_1, \dots, A_n) \preceq T_j(A_1, \dots, A_n)$, the distance vector $DV_{i,j} = [d_1, d_2, \dots, d_n]$ captures the generalization depth, where each $d_z$ ($z = 1, \dots, n$) represents the path length between $\text{dom}(A_z, T_i)$ and $\text{dom}(A_z, T_j)$ within the domain generalization hierarchy $DGH_{D_z}$.

> **Definition — k-Minimal Generalization with Suppression:** Given a table $T_i$ and a maximum suppression threshold $MaxSup$, a table $T_j$ is a $k$-minimal generalization of $T_i$ iff:
> 1.  $T_j$ satisfies $k$-anonymity while enforcing minimal required suppression: $T_j$ satisfies $k$-anonymity, and $\forall T_z : T_i \preceq T_z \land DV_{i,z} = DV_{i,j}$, if $T_z$ satisfies $k$-anonymity then $|T_j| \ge |T_z|$.
> 2.  Total suppressed tuples do not exceed the threshold: $|T_i| - |T_j| \le MaxSup$.
> 3.  $T_j$ is minimal in generalization depth: $\forall T_z : T_i \preceq T_z$ meeting conditions 1 and 2, $\neg(DV_{i,z} < DV_{i,j})$.

When selecting among valid $k$-minimal generalizations, four optimization criteria are evaluated:
*   **Minimum Absolute Distance:** Prefers generalizations that minimize the sum total of generalization steps across all attributes.
*   **Minimum Relative Distance:** Normalizes generalization steps by dividing each attribute's step count by the total height of its domain hierarchy, minimizing the sum of relative heights.
*   **Maximum Distribution:** Prefers generalizations that maximize the number of distinct tuples retained in the output table.
*   **Minimum Suppression:** Prefers generalizations that suppress the fewest total records.

---

## 4. Algorithmic Enforcement: The Mondrian Multidimensional Partitioning Algorithm

Enforcing $k$-anonymity optimally is an NP-hard problem. Top-down greedy multidimensional partitioning, implemented via the **Mondrian** algorithm, provides an efficient mechanism for constructing $k$-anonymous equivalences.

Mondrian partitions the data space into multidimensional regions (equivalence classes) by recursively selecting an attribute dimension and splitting the data at an optimal cut value.

### Monodimensional vs. Multidimensional Cuts

*   **Monodimensional Cut:** Applies a split across a single dimension uniformly across the entire dataset, ignoring prior splits in other dimensions.
*   **Multidimensional Cut:** Applies a split along a chosen dimension conditionally within a localized region, taking into account cuts previously executed on other dimensions.

```
Anonymize(partition)
    if (no allowable multidimensional cut for partition)
        return φ : partition → summary
    else
        dim ← choose_dimension()
        fs ← frequency_set(partition, dim)
        splitVal ← find_median(fs)
        lhs ← {t ∈ partition : t.dim ≤ splitVal}
        rhs ← {t ∈ partition : t.dim > splitVal}
        return Anonymize(rhs) ∪ Anonymize(lhs)
```

An execution cut is deemed **allowable** if and only if both resulting sub-partitions ($lhs$ and $rhs$) contain at least $k$ records. If no allowable cut exists for a partition, the algorithm terminates recursion on that partition and executes the **Summary Phase**, replacing raw attribute values with generalized bounding intervals or set summaries.

### Dimension Selection Strategies

To determine which attribute dimension $j$ to split during recursion, two mathematical selection criteria are used:

1.  **Widest Normalized Span:** Selects the dimension with the largest relative range within partition $P$ compared to the entire original dataset $D$:

$$S_j(P) = \frac{\max(P_j) - \min(P_j)}{\max(D_j) - \min(D_j)}$$

2.  **Entropy-Based Selection:** Evaluates the empirical entropy of attribute $X_j$ within partition $P$ and selects the attribute exhibiting the highest entropy (greatest disorder):

$$H(X_j \mid P) = -\sum_{v} p(v) \log_2 p(v)$$

where $v$ ranges over all distinct values of attribute $X_j$ present in $P$, and $p(v)$ is the proportion of records in $P$ taking value $v$.

### Optimal Splitting Value Selection

While splitting at the median value ($find\_median$) is the standard heuristic, an alternative strategy maximizes the reduction in information loss using the **Normalized Certainty Penalty (NCP)**. The penalty of a partition $P$ across $d$ dimensions is given by:

$$L(P) = |P| \sum_{j=1}^{d} w_j S_j(P)$$

where $w_j$ represents the importance weight assigned to dimension $j$. The optimal split point is selected to maximize the information gain:

$$\text{Gain} = L(P) - L(P_L) - L(P_R)$$

---

## 5. Attacks on k-Anonymity & The ℓ-Diversity Framework

While $k$-anonymity prevents identity linking by ensuring that records are indistinguishable across quasi-identifiers, it does not restrict the distribution of **sensitive attributes** within an equivalence class. This leaves $k$-anonymous tables vulnerable to two major disclosure attacks.

### Vulnerabilities of k-Anonymity

| Attack Type | Mechanism & Threat | Vulnerability Condition |
| :--- | :--- | :--- |
| **Homogeneity Attack** | All $k$ records within an equivalence class share the exact same sensitive attribute value. The adversary infers the sensitive value with $100\%$ certainty without needing to resolve individual identity. | Zero variance / zero diversity of sensitive attributes inside a $q$-block. |
| **Background Knowledge Attack** | Sensitive values in an equivalence class are distinct, but an adversary leverages external domain or demographic facts to eliminate options until a single choice remains. | Insufficient statistical uncertainty relative to external adversary knowledge. |

To defend against these vulnerabilities, privacy models adhere to the **Uninformative Disclosure Principle**.

> **Definition — Uninformative Disclosure Principle:** A published table should provide an adversary with minimal additional information relative to their background knowledge. That is, the difference between an adversary's prior belief (before observing the published table) and posterior belief (after observing the table) regarding a subject's sensitive attribute must be bounded and small.

### The ℓ-Diversity Framework

To enforce diversity within sensitive attributes, the **$\ell$-diversity** framework requires that every equivalence class contain well-represented sensitive values.

> **Definition — Distinct ℓ-Diversity:** An equivalence class satisfies distinct $\ell$-diversity if there are at least $\ell$ distinct values for the sensitive attribute within that equivalence class. A table satisfies distinct $\ell$-diversity if every equivalence class within it satisfies distinct $\ell$-diversity.

*   **Vulnerability of Distinct $\ell$-Diversity:** Distinct $\ell$-diversity only counts unique categories, making it vulnerable to unbalanced class distributions. For example, an equivalence class with 100 records containing 96 instances of *Flu*, 2 of *Cold*, and 2 of *Bronchitis* satisfies distinct 3-diversity, yet an adversary can still predict *Flu* with $96\%$ probability.

To address class imbalance, **Entropy $\ell$-Diversity** incorporates entropy measurements into the diversity requirement.

> **Definition — Entropy ℓ-Diversity:** Let $S$ be the domain of sensitive attribute values, and $p(s, E)$ be the probability of observing value $s \in S$ within equivalence class $E$. The entropy of $E$ is defined as:
>
> $$\text{Entropy}(E) = -\sum_{s \in S} p(s, E) \log p(s, E)$$
>
> An equivalence class $E$ satisfies Entropy $\ell$-Diversity if and only if:
>
> $$\text{Entropy}(E) \ge \log(\ell)$$
>
> A table satisfies Entropy $\ell$-Diversity if all of its equivalence classes satisfy Entropy $\ell$-Diversity.

While Entropy $\ell$-diversity guarantees high diversity and effectively eliminates frequency imbalance, it is highly conservative and can severely reduce data utility in practice.

### Step-by-Step Computation of ℓ-Diversity

Computing and enforcing $\ell$-diversity follows a three-step procedure:

1.  **Step 1 — Identify Equivalence Classes ($q$-blocks):** Group table records based on matching quasi-identifier tuples and count total records per $q$-block.
2.  **Step 2 — Evaluate Distinct $\ell$-Diversity:** Count the number of distinct sensitive attribute values present in each $q$-block. If any $q$-block contains fewer than $\ell$ distinct sensitive values, distinct $\ell$-diversity is violated.
3.  **Step 3 — Evaluate Entropy $\ell$-Diversity:**
    *   *Step 3.1:* Convert raw frequency counts for each sensitive value into empirical probabilities $p(s, E) = \frac{\text{count}(s)}{|E|}$ and compute $\log_2 p(s, E)$.
    *   *Step 3.2:* Calculate products $p(s, E) \cdot \log_2 p(s, E)$, sum across all sensitive values $s \in S$, and negate to obtain $\text{Entropy}(E)$.
    *   *Step 3.3:* Compare $\text{Entropy}(E)$ against $\log_2(\ell)$. If $\text{Entropy}(E) < \log_2(\ell)$ for any $q$-block, the table fails entropy $\ell$-diversity and requires further generalization or suppression.

---

## 6. Limitations of ℓ-Diversity & The t-Closeness Framework

Despite preventing homogeneity and background knowledge attacks, $\ell$-diversity remains vulnerable to attacks that exploit semantic relationships among sensitive values: the **Similarity Attack**.

### The Similarity Attack

A similarity attack occurs when the sensitive values within an equivalence class are distinct ($\ell$-diverse) but share a single semantic meaning or narrow numerical range.

*   *Example:* An equivalence class contains sensitive income values of $\$3\text{k}$, $\$4\text{k}$, and $\$5\text{k}$. The class satisfies distinct 3-diversity and entropy 3-diversity. However, an adversary who identifies a target subject within this class learns with $100\%$ certainty that the individual has a extremely low income, even if the exact value is not pin-pointed.

### The t-Closeness Framework

To prevent similarity attacks, the **$t$-closeness** model constrains the distributional distance between local equivalence classes and the global dataset.

> **Definition — t-Closeness Principle:** An equivalence class $E$ satisfies $t$-closeness if the distance between the probability distribution of a sensitive attribute within $E$ and the probability distribution of the sensitive attribute in the whole table does not exceed threshold $t$. A table satisfies $t$-closeness if all of its equivalence classes satisfy $t$-closeness.

$t$-Closeness satisfies two major structural properties:
*   **Generalization Property:** Let $T$ be a table, and $A$ and $B$ be two generalizations on $T$ such that $A$ is more general than $B$. If $T$ satisfies $t$-closeness under generalization $B$, $T$ automatically satisfies $t$-closeness under $A$.
*   **Subset Property:** Let $C$ be a set of attributes in table $T$. If $T$ satisfies $t$-closeness with respect to $C$, then $T$ satisfies $t$-closeness with respect to any attribute subset $D \subset C$.

### Quantifying Distribution Distance: Earth Mover's Distance (EMD)

To evaluate $t$-closeness, the distance between the local sensitive attribute distribution $P = (p_1, p_2, \dots, p_m)$ in an equivalence class and the global distribution $Q = (q_1, q_2, \dots, q_m)$ across the entire table is measured using the **Earth Mover's Distance (EMD)**.

EMD quantifies the minimum work required to transform distribution $P$ into distribution $Q$, conceptually measuring how much probability mass ("dirt") must be moved and over what distance.

| Attribute Domain Type | Mathematical Formulation of Earth Mover's Distance (EMD) | Operational Meaning |
| :--- | :--- | :--- |
| **Categorical Attributes** | $$D[P, Q] = \frac{1}{2} \sum_{i=1}^{|S|} \|p_i - q_i\|$$ | Evaluates $L_1$ distance across unordered categories, assuming equal inter-category distance ($1$). |
| **Numerical Discrete Attributes** | $$D[P, Q] = \frac{1}{\|S\| - 1} \sum_{i=1}^{\|S\|} \left\| \sum_{j=1}^{i} r_j \right\|$$ | Measures accumulated probability mass differences $r_i = p_i - q_i$ across an ordered sequence. |
| **Numerical Continuous Attributes** | $$D[P, Q] = \int_{S} \|F_P(x) - F_Q(x)\| \, dx$$ | Integrates the absolute area difference between Cumulative Distribution Functions (CDFs) $F_P(x)$ and $F_Q(x)$. |

---

## 7. Evaluation Metrics for Anonymization Quality & System Performance

Evaluating microdata protection requires assessing three competing operational dimensions: **Computational Efficiency**, **Re-identification Risk**, and **Information Loss (Data Quality)**.

| Operational Dimension | Primary Evaluation Metric | System Trade-Off Dynamics |
| :--- | :--- | :--- |
| **Computational Efficiency** | Processing time vs. Quasi-Identifier count ($|QI|$) | NP-hard complexity; execution time scales non-linearly as $t$-closeness constraints tighten. |
| **Re-identification Risk** | Average Equivalence Class Size & Population Uniqueness ($Pr(PU)$) | Larger $q$-blocks reduce de-anonymization probability ($1/k$), but increase information loss. |
| **Information Loss / Utility** | Discernibility Metric ($DM$), MSE/MAE, or Machine Learning Degradation | Minimal generalization maximizes utility, but increases disclosure risk for rare tuples. |

### Computational Complexity & Efficiency

Achieving optimal $k$-anonymity, $\ell$-diversity, or $t$-closeness is NP-hard. Algorithms scale non-linearly with respect to the number of quasi-identifiers. Experimental benchmarks show that execution time increases exponentially as the number of quasi-identifiers grows beyond $k$-anonymity to enforce tighter $t$-closeness thresholds (e.g., $t=0.2$ vs $t=0.15$).

### Re-identification Risk Metrics

1.  **Average Equivalence Class Size:** Measures the average cardinal size of all $q$-blocks generated in anonymized table $T^*$:

$$\text{Avg } |q\text{-block}| = \frac{|T^*|}{|EQ(T^*)|}$$

where $|EQ(T^*)|$ is the total number of equivalence classes. Larger average equivalence class sizes indicate lower re-identification risk, but correspond to greater information loss.

2.  **Population Uniqueness ($Pr(PU)$):** Calculates the probability that a combination of quasi-identifier attributes in the dataset is unique within the broader overall population $N$:

$$\text{Pr}(PU) = \sum_{j} \frac{I(F_j = 1)}{N}$$

where $F_j$ is the frequency of combination $j$ in the population, and $I(\cdot)$ is an indicator function returning $1$ if $F_j = 1$ and $0$ otherwise.

### Information Loss Metrics

Information loss quantifies the utility penalty introduced by anonymization:

1.  **Continuous Data:** Evaluated by comparing pre- and post-anonymization statistical summaries using Mean Square Error (MSE), Mean Absolute Error (MAE), or Mean Variation.
2.  **Categorical Data:** Evaluated via cell change counts, category distance tracking, contingency table comparisons, or Shannon Entropy reduction.
3.  **Machine Learning Utility:** Evaluated by measuring the degradation in predictive performance (accuracy, precision, recall, F1-score) of models trained on anonymized microdata versus models trained on raw microdata.

### The Discernibility Metric (DM)

The **Discernibility Metric** assigns a penalty to each record based on how indistinguishable it is from other records in the anonymized table $T$.

> **Definition — Discernibility Penalty:** Let $t$ be a record in table $T$, and $E_t$ be the equivalence class containing $t$. The penalty assigned to $t$ is:
>
> $$\text{penalty}(t) = \begin{cases} |T| & \text{if } t \text{ is suppressed} \\ |E_t| & \text{otherwise} \end{cases}$$
>
> The total discernibility score of table $T$ is the sum of penalties across all records:
>
> $$\text{discernibility}(T) = \sum_{t \in T} \text{penalty}(t)$$

Suppressed records incur the maximum penalty ($|T|$) because they are completely lost to analysis. Unsuppressed records incur a penalty equal to the size of their equivalence class ($|E_t|$), reflecting the degree of ambiguity introduced. Lower discernibility scores indicate higher data utility and lower overall information loss.
