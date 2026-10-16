# k-Anonymity Frameworks and Mechanisms

## Executive Summary

Simple de-identification and sanitization—such as removing direct Personally Identifiable Information (PII) like names or Social Security Numbers—do not guarantee anonymity in published datasets. With the proliferation of publicly available data sources, social networks, and high-performance computing, attackers can aggregate disparate datasets to conduct **identity linking attacks**. This briefing outlines the foundational privacy paradigm of **k-anonymity**, detailing its mathematical definitions, vulnerability models, implementation mechanisms (generalization and suppression), metrics for minimal distortion, and algorithmic enforcement via multidimensional partitioning (Mondrian).

Key takeaways include:

- **The Identity Linking Threat:** Attackers exploit non-direct identifiers—termed **quasi-identifiers** (e.g., Date of Birth, ZIP code, Sex, Marital Status)—by linking sanitized private tables with external public sources to uniquely re-identify individuals and uncover sensitive attributes.
- **Core Principle of k-Anonymity:** A dataset satisfies k-anonymity if every unique combination of quasi-identifier values appears at least k times, forming equivalence classes (q-blocks) of size \ge k. The probability of identifying a specific individual within a q-block is 1/k.
- **Data Transformation Mechanisms:** k-anonymity is enforced primarily through **Generalization** (replacing specific values with broader categorical domains via hierarchical structures) and **Suppression** (removing outlier tuples, attributes, or cells to reduce necessary generalization).
- **Optimization and Minimality:** To preserve data utility, anonymization must be k-minimal. Selection criteria such as minimum absolute distance, minimum relative distance, maximum distribution, and minimum suppression guide the balance between privacy protection and data utility under a maximum allowable suppression threshold (\text{MaxSup}).
- **Algorithmic Enforcement:** Greedy top-down partitioning algorithms, such as **Mondrian**, recursively split data along chosen dimensions using median values to generate compliant k-anonymous summary partitions.

## 1. Threat Model: Identity Linking Attacks & Quasi-Identifiers

### De-Identification Limitations

Removing direct identifiers (sanitization) is insufficient to protect subject privacy. Vulnerabilities stem from two primary drivers:

1. **Large Volumes of Information:** Massive quantities of public data sources and social network records are readily accessible.
2. **High Computational Power:** Advanced computing infrastructure allows large-scale data fusion and record matching.

### Record Linkage vs. Identity Linking

Understanding re-identification requires distinguishing between record linkage and identity linking:

- **Record Linkage:** The process of linking multiple records across datasets that correspond to the same individual, without necessarily revealing the real-world identity.
- **Identity Linking:** A specific instance of record linkage where at least one linked record contains one or more direct identifiers (PII), directly de-anonymizing the subject.

```
+-------------------------------------------------------+
| Record Linkage                                        |
| (Linking multiple records of the same individual)     |
|                                                       |
|   +-----------------------------------------------+   |
|   | Identity Linking                              |   |
|   | (Includes direct identifiers / PII,           |   |
|   |  resulting in complete de-anonymization)      |   |
|   +-----------------------------------------------+   |
+-------------------------------------------------------+
```

### Quasi-Identifiers (QIs)

Quasi-identifiers are sets of attributes present in a private table that are also externally available and can be exploited for linking attacks.

- **Common Examples:** Race, Date of Birth (DoB), Sex, ZIP Code, Marital Status.
- **Mechanism:** While a single quasi-identifier value may not uniquely identify a person, combining multiple quasi-identifiers creates a high-dimensional signature that uniquely pinpoints individual records.
- _Exemplar Vulnerability:_ In a table stripped of SSNs and Names, knowing that a target (e.g., "Louise Wayland") was born on `64/09/27`, resides in a specific ZIP code, and is a `widow` allows an attacker to uniquely isolate her row and reveal her sensitive `Income` ($56,000).

## 2. Formalization of k-Anonymity

### Definition and Requirements

The k-anonymity requirement mandates that every release of data must ensure that every combination of quasi-identifier values can be indistinctly matched to at least k respondents.

- **Formal Definition:** Let T(A_1, \dots, A_m) be a relational table, and QI be the set of quasi-identifiers associated with it. T satisfies k-anonymity with respect to QI if and only if each sequence of values in the projection T[QI] appears with at least k occurrences in T[QI].
- **Equivalence Class (q-block):** A set of tuples in T that share the exact same quasi-identifier attribute values. Under k-anonymity, every q-block must contain at least k tuples.

### Evaluating the Parameter k

The selection of k dictates the privacy-utility tradeoff:

- **k = 1****:** Offers zero privacy protection, leading directly to record isolation violations.
- **k = |T|** **(Database Size** **n****):** Provides complete privacy but destroys data utility by collapsing all records into a single uniform group.
- **Re-identification Probability:** The risk of an individual being identified within a compliant q-block is given by \frac{1}{k}.
    - For k = 2, there is a 50\% probability of being correctly identified in an attack.
    - While k = 5 is often considered a practical starting threshold, the optimal value depends heavily on the specific dataset setup and operational requirements.

## 3. Anonymization Techniques: Generalization & Suppression

To satisfy k-anonymity without introducing false data, transformations must preserve data truthfulness.

### Generalization Mechanics

Generalization replaces specific quasi-identifier values with broader, less specific categories.

- **Domains (****Dom****):** The set of all possible values for a given attribute. A generalized domain encompasses multiple primitive domains.
- **Domain Generalization Hierarchy (****\text{DGH}_D****):** Defined by a partial ordering relationship (\le_D) over a set of domains Dom, satisfying two conditions:
    1. **Condition 1:** \forall D_i, D_j, D_z \in Dom : (D_i \le_D D_j \land D_i \le_D D_z) \Rightarrow (D_j \le_D D_z \lor D_z \le_D D_j).
    2. **Condition 2:** All maximal elements of Dom are singletons (represented structurally as a tree where all domain paths converge to a single root value).
- **Value Generalization Hierarchy (****\text{VGH}_D****):** A value generalization relationship (\le_V) that maps each specific value in domain D_i to a unique direct generalization in domain D_j.

#### Granularity of Generalization

- **Attribute Generalization (AG):** Generalization is applied uniformly across an entire column/attribute.
- **Cell Generalization (CG):** Generalization is applied at the individual cell level; a table may contain entries at differing levels of generalization within the same attribute.
- **Sensitive Attribute Handling:** Sensitive attributes (e.g., medical diagnoses, income) should only be generalized if they also act as quasi-identifiers. Generalizing sensitive attributes drastically reduces overall dataset utility.

### Suppression Mechanics

Rare attribute values or statistical outliers can force extreme generalization across an entire dataset to satisfy k-anonymity. To prevent excessive data distortion, outlier records can be suppressed (removed or blanked out).

#### Granularity of Suppression

- **Tuple Suppression (TS):** Entire rows/records are removed from the dataset.
- **Attribute Suppression (AS):** Entire columns/attributes are deleted.
- **Cell Suppression (CS):** Specific individual cells are removed/replaced with null values.

### Taxonomy of Anonymization Approaches

Combining generalization and suppression techniques yields a spectrum of operational strategies:

|   |   |   |   |   |
|---|---|---|---|---|
|Generalization Level \ Suppression Level|Tuple Suppression (TS)|Attribute Suppression (AS)|Cell Suppression (CS)|No Suppression (_)|
|**Attribute Generalization (AG)**|`AG_TS`|`AG_AS`|`AG_CS`|`AG_`|
|**Cell Generalization (CG)**|`CG_TS`|`CG_AS`|`CG_CS`|`CG_`|
|**No Generalization (_)**|`_TS`|`_AS`|`_CS`|N/A|

## 4. Mathematical Models for Minimality & Distance

### Generalized Table with Suppression

Let T_i and T_j be two tables defined over the same attribute set. Table T_j is a generalization with tuple suppression of T_i (denoted T_i \preceq T_j) if:

1. |T_j| \le |T_i|
2. For each attribute A, \text{dom}(A, T_j) is equal to, or a generalization of, \text{dom}(A, T_i).
3. An injective function exists mapping each tuple t_j \in T_j to a tuple t_i \in T_i such that every attribute value in t_j is equal to, or a generalization of, the corresponding attribute value in t_i.

### Distance Vector (DV_{i,j})

The structural distance between an original table T_i(A_1, \dots, A_n) and a generalized table T_j(A_1, \dots, A_n) where T_i \preceq T_j is represented as a vector: \text{DV}_{i,j} = [d_1, d_2, \dots, d_n] Where each component d_z (z = 1, \dots, n) represents the length of the unique path between \text{dom}(A_z, T_i) and \text{dom}(A_z, T_j) within the domain generalization hierarchy \text{DGH}_{D_z}.

### k-Minimal Generalization Requirements

Given a maximum allowable suppression threshold (\text{MaxSup}), a table T_j is considered a **k-minimal generalization** of T_i if and only if:

1. **Minimal Required Suppression:** T_j satisfies k-anonymity, and for any T_z such that T_i \preceq T_z and \text{DV}_{i,z} = \text{DV}_{i,j}, if T_z satisfies k-anonymity, then |T_j| \ge |T_z|.
2. **Suppression Threshold Bound:** |T_i| - |T_j| \le \text{MaxSup}.
3. **Distance Minimality:** For all T_z such that T_i \preceq T_z satisfying conditions 1 and 2, \neg(\text{DV}_{i,z} < \text{DV}_{i,j}).

### Preferred Generalization-Suppression Criteria

When selecting optimal transformations among valid minimal generalizations, systems utilize four main selection criteria:

- **Minimum Absolute Distance:** Prefers generalizations that minimize the total sum of generalization steps across attributes.
- **Minimum Relative Distance:** Prefers generalizations that minimize relative steps (where each attribute step is normalized by dividing by the total height of its corresponding domain hierarchy).
- **Maximum Distribution:** Prefers generalized tables that retain the highest number of distinct tuples.
- **Minimum Suppression:** Prefers transformations that suppress the fewest number of tuples.

## 5. Algorithmic Enforcement: The Mondrian Algorithm

Mondrian is a multidimensional partitioning algorithm designed to achieve k-anonymity by recursively splitting the data space into sub-regions using median cuts.

### Pseudocode Implementation

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

### Execution Logic

1. **Termination Check:** The algorithm checks if any valid, allowable multidimensional cuts remain that would keep child partitions above size k. If no cuts are possible, it converts the partition into a summary region (\varphi).
2. **Dimension Selection:** It selects a quasi-identifier dimension (`dim`) along which to split the dataset.
3. **Frequency & Median Computation:** It constructs the frequency distribution (`fs`) of values along the selected dimension and locates the median value (`splitVal`).
4. **Binary Partitioning:** It splits the current partition into left-hand side (`lhs`) and right-hand side (`rhs`) sub-partitions relative to `splitVal`.
5. **Recursion:** It recursively calls `Anonymize` on both child partitions and returns their union.