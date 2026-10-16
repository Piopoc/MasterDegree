# Macrodata and Microdata Protection in Privacy-Preserving Information Access

## Executive Summary

Data sharing is essential for statistical analysis, marketing, and scientific research. However, releasing raw or aggregated data presents severe privacy risks, including the re-identification of individuals and the unauthorized disclosure of sensitive attributes.

This briefing document provides an exhaustive technical analysis of the strategies, methodologies, and mathematical rules governing **Macrodata** and **Microdata Protection**, based on the _Privacy Preserving Information Access_ curriculum at the University of Padua (Academic Year 2026/2027, taught by Guglielmo Faggioli).

### Key Takeaways

1. **Core Distinction Between Security and Privacy:** Security focuses on restricting access to data to prevent theft or unauthorized entry. Privacy protection focuses on preventing the disclosure or inference of sensitive information once data is accessed, published, or evaluated for value.
2. **Data Classification Framework:** Effective privacy control relies on categorizing attributes into **Identifiers**, **Quasi-identifiers**, **Confidential attributes**, and **Non-confidential attributes**.
3. **Macrodata Protection Strategies:** Tabular data releases require protection against inference risks on rare subjects. Techniques include **Primary/Secondary Cell Suppression**, **Controlled Tabular Adjustment (CTA)**, **Rounding**, and **Category Roll-up**, alongside formal detection methods like the **(n, k)****-rule** and the **p****-percentage rule**.
4. **Microdata Protection Paradigms:** Individual-level records face **Identity**, **Attribute**, and **Inferential disclosures**. Mitigation strategies balance privacy protection against statistical utility using **Non-Perturbative Masking**, **Perturbative Masking**, and **Synthetic Data Generation**.

## 1. Fundamentals of Data, Security, and Privacy

### Data Definition & Interdependence

Data is defined as information—specifically facts or numbers—collected to be examined, considered, and used to support decision-making. Individual data points gain meaningful context only when evaluated collectively.

Anonymization strategies evaluate data through three key analytical vectors:

- **Record Isolation:** Preventing single records from being distinctly isolated.
- **Inference Control:** Managing authorized versus unauthorized inferences.
- **Linkage:** Preventing unauthorized linkage between disparate datasets.

### Attribute Taxonomy

Attributes within a dataset are classified into four structural types:

|   |   |   |
|---|---|---|
|Attribute Type|Definition|Examples|
|**Identifiers**|Attributes that allow for the direct or (almost) univocal identification of a person or entity.|Name, SSN/tax code, student ID, fingerprints, email address, phone number.|
|**Quasi-identifiers**|Attributes that do not uniquely identify an individual in isolation, but can re-identify a person when combined.|Postal/zip code, gender, Date of Birth (DoB), workplace name.|
|**Confidential Attributes**|Attributes that pertain to the private or personal sphere of the respondent.|Medical diagnoses/diseases, income, financial balance.|
|**Non-confidential Attributes**|Information that currently cannot be used to disclose a user's identity or sensitive information.|General non-sensitive observations (subject to change over time).|

### Security vs. Privacy Protection

- **Data Security:** Protecting physical or digital data from unauthorized access, theft, or corruption. Security manages data _access_ and serves as a prerequisite for privacy.
- **Privacy Protection:** Protecting the underlying information contained within or inferable from the data. Privacy concerns arise when data (or a subset thereof) is released, shared, or processed to extract value.

## 2. Data Release Paradigms & Macrodata Protection

Data is released across three distinct structural formats based on operational requirements (e.g., mandatory statistical reporting, marketing, or research):

```
                      ┌─────────────────────────┐
                      │    Data Release Form    │
                      └────────────┬────────────┘
                                   │
        ┌──────────────────────────┼──────────────────────────┐
        ▼                          ▼                          ▼
   Macrodata              Statistical Databases            Microdata
(Aggregated Tables)      (API / Distributional)        (Individual Records)
```

### 1. Macrodata

Macrodata consists of aggregated data presented in tabular format.

- **Categories of Tables:** Counts/frequencies (statistics about quantity/totals) and magnitudes.
- **Weaknesses:** Information regarding rare subjects can still be inferred from aggregate numbers; provides low flexibility and limited information availability.
- **Protection Approach:** Selective obfuscation of sensitive cells.

#### Sensitive Cell Identification in Count & Frequency Tables

A **sensitive cell** is a table cell that exposes too much information by isolating or identifying a small number of subjects. Threshold rules establish explicit cutoffs (e.g., setting a count threshold of 2, below which a cell is deemed sensitive).

#### Protection Techniques for Counts and Frequencies

- **Cell Suppression:**
    - _Primary Suppression:_ Direct removal of cells containing values below the established threshold.
    - _Secondary Suppression:_ Removal of additional non-sensitive cells using linear programming to prevent attackers from recalculating or estimating the suppressed primary values from marginal totals.
- **Rounding:** Rounding values to the closest multiple of a chosen base number. This technique is particularly effective for mitigating record isolation.
- **Category Roll-up:** Reducing table dimensions by combining fine-grained categories into broader macro-categories, thereby increasing the count of subjects per cell.
- **Sampling:** Transitioning from full census data collection to sample surveys.
- **Controlled Tabular Adjustment (CTA):** Lower values are replaced with the target threshold value, and linear programming is used to adjust the remaining cells to preserve table sums. This sacrifices exact cell utility to enforce threshold constraints.

#### Sensitive Cell Identification in Magnitude Tables

For tables reporting continuous magnitudes (e.g., economic output, total income), specialized rules identify sensitive cells:

- **(n, k)****-Rule:** A cell is marked sensitive if fewer than n subjects contribute at least k\% of the cell's total value.
    - _Example:_ Under a (2, 50)-rule, a cell is sensitive if a single user contributes over 50\% of the total value.
    - _Example:_ Under a (3, 30)-rule, if fewer than 3 individuals contribute over 30\% each (e.g., 2 individuals contributing 50\% each), the cell is sensitive.
- **p****-Percentage Rule:** A cell t with total value t, contributed to by individual values v_1 (largest contribution) and v_2 (second largest contribution), is sensitive if the following inequality holds (where p < 100 is the security parameter): t - v_1 - v_2 < \frac{p}{100} \cdot v_1 If the inequality holds, t is sensitive because an adversary can easily estimate v_1.

### 2. Statistical Databases

Databases that restrict query capabilities to distributional queries via an API.

- **Weaknesses:** Subject to reconstruction attacks and differencing attacks; low flexibility.
- **Protection Techniques:** Query rate limiting, result perturbation, and **Differential Privacy**.

## 3. Microdata Protection Framework

Microdata consists of individual-level records representing unique entities (e.g., patients, citizens, or corporations) requiring privacy protection.

```
                              ┌───────────────────┐
                              │ Microdata Risks   │
                              └─────────┬─────────┘
                                        │
        ┌───────────────────────────────┼───────────────────────────────┐
        ▼                               ▼                               ▼
Identity Disclosure            Attribute Disclosure            Inferential Disclosure
(Direct re-identification)     (Inferring sensitive values)    (Statistical property inference)
```

### Risk Profile & Vulnerabilities

- **Outliers / Highly Visible Tuples:** Individuals with rare attributes or peculiarities (e.g., rare medical conditions) represent the most vulnerable and exposed subjects.
- **Re-identification Attacks:** Joining sanitized microdata with external datasets using shared quasi-identifiers.

### Natural Defenses of Microdata

1. **Sampling Subsets:** Datasets rarely cover 100\% of a population, creating uncertainty about whether a target subject is present.
2. **Temporal Lag:** Data collection and public release are separated by years, making data outdated.
3. **Natural Noise:** Inherent collection noise hinders accurate cross-referencing with external databases.
4. **Format Heterogeneity:** Inconsistent structural formats across disparate datasets complicate linkages.

### Types of Disclosure

1. **Identity Disclosure:** Re-identification of a specific individual through direct or combined unique attributes (e.g., Name, SSN, Address).
2. **Attribute Disclosure:** Re-identifying an individual's sensitive attribute value via combinations of indirect identifiers (quasi-identifiers).
3. **Inferential Disclosure:** High-probability deduction of personal information based on the statistical properties of the published dataset.

### Data Sanitization & The Core Desiderata

Sanitization begins by removing all explicit identifiers (names, exact street addresses, SSNs, phone numbers). However, sanitization alone is insufficient due to quasi-identifier linkage risks.

Data custodians must manage a fundamental trade-off (**Microdata Desiderata**):

1. **Protect User Privacy:** Modify, hide, or transform data so individual re-identification becomes impossible.
2. **Maintain Key Statistical Properties:** Preserve analytical utility; over-scrambling transforms data into useless white noise.

## 4. Technical Microdata Protection Techniques

Microdata protection strategies are divided into **Masking** (Non-perturbative vs. Perturbative) and **Synthetic Data Generation**.

```
                           ┌───────────────────────────┐
                           │ Microdata Protection      │
                           └─────────────┬─────────────┘
                                         │
        ┌────────────────────────────────┴────────────────────────────────┐
        ▼                                                                 ▼
   Masking                                                   Synthetic Data Generation
   ├── Non-Perturbative (Suppress/Eliminate)                 ├── Fully Synthetic
   └── Perturbative (Alter/Add Noise)                        └── Partial/Perturbative Synthetic
```

### Non-Perturbative Masking Methods

Non-perturbative methods remove or reduce detail without injecting false or altered values into the dataset.

- **Sampling:** Suppresses observations using a "just a few" policy, lowering the probability that a specific individual exists in the dataset.
    - _Applicability:_ Categorical: **YES** | Continuous: **YES**
- **Local Suppression:** Replaces specific attributes or cell values that drive disclosure risk with a "missing value" indicator (null).
    - _Applicability:_ Categorical: **YES** | Continuous: **YES**
- **Global Recoding:** Partitions the domain of an attribute into disjoint, equal-sized intervals and replaces raw values with their interval label.
    - _Applicability:_ Categorical: **ONLY ORDERED** | Continuous: **YES**
- **Top-Coding & Bottom-Coding:** Specific instances of global recoding.
    - _Top-Coding:_ Replaces all values above a threshold with a capped label (e.g., heights > 1.9\text{m} mapped to `">1.9m"`).
    - _Bottom-Coding:_ Replaces all values below a threshold with a floor label (e.g., heights < 1.7\text{m} mapped to `"<1.7m"`).
    - _Applicability:_ Categorical: **ONLY ORDERED** | Continuous: **YES**
- **Generalization:** Uses a formal generalization hierarchy to replace attributes with higher-level abstractions (e.g., converting full DoB to birth year/month, or truncating zip codes).
    - _Applicability:_ Categorical: **YES** | Continuous: **VIA GLOBAL RECODING**

### Perturbative Masking Methods

Perturbative methods alter the data directly, introducing noise, swaps, or mathematical transformations.

- **Rounding:** Replaces continuous values with fixed interval points instead of range labels.
    - _Mathematical Formulation:_ Given a step size b and k points p_1, \dots, p_k, each interval I_i is defined as: I_i = \begin{cases} [0, p_1 + \frac{b}{2}], & \text{if } i = 0 \\ [p_i + \frac{b}{2}, p_{i+1} + \frac{b}{2}), & \text{if } 0 < i < k \\ [p_k, \text{max}], & \text{if } i = k - 1 \end{cases} Where p_i is the label point assigned to interval I_i.
    - _Applicability:_ Categorical: **NO** | Continuous: **YES**
- **Resampling:** Resamples the target column with replacement multiple times (S_1, S_2, S_3, \dots), sorts the values within each resampled column, calculates row-wise averages across sorted samples, and maps these perturbed average values back to the original dataset elements.
    - _Applicability:_ Categorical: **NO** | Continuous: **YES**
- **Lossy Compression:** Formats tabular data as an image matrix, converts scalar values into pixel "colors", and applies lossy compression algorithms (e.g., JPEG). The compression rate serves as the obfuscation parameter.
    - _Applicability:_ Categorical: **NO** | Continuous: **YES**
- **Post RAndomized Method (PRAM):** Randomly alters categorical labels according to a predefined probability transition matrix.
    - _Example Matrix:_ Preserves original labels with baseline probabilities while mapping related concepts together based on semantic proximity (e.g., mapping `meat` to `fish` with 0.25 probability since both are animal products, but to `vegetables` with only 0.05 probability).
    - _Applicability:_ Categorical: **YES** | Continuous: **NO**
- **Random Noise (Additive Noise):** Adds noise sampled from a statistical distribution. Uncorrelated noise samples independently, whereas correlated noise accounts for attribute covariance. Additive noise typically requires combination with linear or non-linear transformations to ensure adequate security.
    - _Applicability:_ Categorical: **NO** | Continuous: **YES**
- **Swapping:** Swaps sensitive attribute values randomly between records.
    - _Trade-off:_ Provides strong identity protection, but fails to preserve statistical relationships across subdomains (non-confidential attribute groupings).
    - _Applicability:_ Categorical: **YES** | Continuous: **YES**
- **Rank Swapping:** Swaps values between records sorted by rank, restricting swaps to tuples within a maximum distance p (where p is the obfuscation parameter).
    - _Applicability:_ Categorical: **YES** | Continuous: **YES**

### Synthetic Data Generation

Rather than masking existing records, synthetic generation constructs artificial datasets:

- **Fully Synthetic Data:** Generates an entirely new artificial dataset based on the estimated probabilistic distribution of the original dataset.
- **Partial / Perturbative Synthetic Data:** Generates synthetic values for specific sensitive attributes or vulnerable tuples while retaining real data for remaining fields.

## 5. Comparative Reference Matrix for Microdata Protection

|   |   |   |   |   |
|---|---|---|---|---|
|Technique|Method Class|Categorical Data|Continuous Data|Key Operational Mechanism / Notes|
|**Sampling**|Non-Perturbative|**YES**|**YES**|Removes a subset of records ("just a few" policy).|
|**Local Suppression**|Non-Perturbative|**YES**|**YES**|Replaces risky cells with missing values (null).|
|**Global Recoding**|Non-Perturbative|**ONLY ORDERED**|**YES**|Maps values into discrete range intervals.|
|**Top/Bottom Coding**|Non-Perturbative|**ONLY ORDERED**|**YES**|Caps/floors extreme tails of distributions.|
|**Generalization**|Non-Perturbative|**YES**|**VIA RECODING**|Substitutes attributes using hierarchical structures.|
|**Rounding**|Perturbative|**NO**|**YES**|Replaces values with interval midpoints/labels.|
|**Resampling**|Perturbative|**NO**|**YES**|Resamples, sorts, averages, and re-maps columns.|
|**Lossy Compression**|Perturbative|**NO**|**YES**|Treats table as image; applies JPEG-like lossy compression.|
|**PRAM**|Perturbative|**YES**|**NO**|Probabilistic label switching via transition matrix.|
|**Random Noise**|Perturbative|**NO**|**YES**|Adds uncorrelated or correlated additive noise.|
|**Swapping**|Perturbative|**YES**|**YES**|Randomly swaps attribute values across tuples.|
|**Rank Swapping**|Perturbative|**YES**|**YES**|Swaps values within a bounded rank distance p.|