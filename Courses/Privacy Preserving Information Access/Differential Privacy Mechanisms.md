---
type: course-note
date: 2026-11-17
tags:
  - course
  - ppia
  - differential-privacy
  - laplace-mechanism
  - exponential-mechanism
  - sparse-vector
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Differetial_Privacy2]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering **Differential Privacy, Part II**: the concrete mechanisms that achieve the definition introduced in [[Differential Privacy Foundations]]. Contents: $\ell_1$-sensitivity, the Laplace mechanism with its $(\varepsilon,0)$-DP proof and accuracy bound, count and histogram queries (both adjacency notions), the 2010-census first-names example, Report Noisy Max, the Gaussian mechanism and $\ell_2$-sensitivity, the exponential mechanism via the auction/pricing example (utility, sensitivity, accuracy), sequential composition recap, and the sparse vector technique (AboveThreshold, Sparse, NumericSparse). Created 2026-11-17 from `raw/Differetial_Privacy2.md` (the raw filename carries a typo, kept because raw is immutable). All slide numbers were recomputed and verified; worked examples beyond the slides are flagged, slide typos are listed in Section 11.

## 1. From Randomized Response to Better Mechanisms

Recap: randomized response (coin toss) is DP with privacy loss

$$
\frac{\Pr[R = Y \mid T = Y]}{\Pr[R = Y \mid T = N]} = \frac{3/4}{1/4} = 3 \quad\Longrightarrow\quad (\ln 3, 0)\text{-DP}
$$

"We can do better than this": randomized response perturbs **every individual answer**, so the error on an aggregate count grows with the population. The mechanisms below perturb **the query answer once**, so the noise depends only on how much one person can move that answer (the sensitivity), not on the database size.

**Worked comparison (not in the slides, high).** Count the YES among $n = 10{,}000$ people with true proportion around $0.5$:

| Mechanism | $\varepsilon$ | Std. deviation of the estimated count |
|---|---|---|
| Randomized response, estimator $n(2Y - \frac{1}{2})$ | $\ln 3$ | $2\sqrt{n\,Y(1-Y)} = 100$ people |
| Laplace on the true count, $\text{Lap}(1/\varepsilon)$ | $\ln 3$ | $\sqrt{2}/\ln 3 \approx 1.29$ people |

The catch: the Laplace mechanism needs a **trusted curator** who sees the real data and adds noise centrally; randomized response needs no one to be trusted (each respondent randomizes locally). That is the price of the accuracy gain.

## 2. $\ell_1$-Sensitivity

Queries return vectors (distributions, histograms): $f : \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$. Their **$\ell_1$-sensitivity** is

$$
\Delta f = \max_{\substack{x, y \in \mathbb{N}^{|\mathcal{X}|} \\ \|x - y\|_1 = 1}} \| f(x) - f(y) \|_1
$$

- It is the **maximum change in the output induced by a single record** (worst case over all neighbouring databases, not the actual data).
- Intuitively, it is the **magnitude of perturbation needed to hide one individual's participation**.

Working assumptions in the lecture:

- neighbouring datasets differ by **one added or removed record** (the histogram adjacency of Part I);
- each individual contributes **one record** (a unique value for the sensitive attribute).

## 3. The Laplace Mechanism

### 3.1 Laplace Distribution

$$
\text{Lap}(x \mid \mu, b) = \frac{1}{2b} \exp\left(-\frac{|x - \mu|}{b}\right)
$$

A symmetric, sharply peaked distribution centred on $\mu$; the **scale** $b$ controls the spread (variance $2b^2$). The slide plot shows $(\mu, b) = (0,1), (0,2), (0,4), (-5,4)$: larger $b$ gives a lower, wider curve. Writing $\text{Lap}(b)$ means $\mu = 0$.

### 3.2 Definition

$$
\mathcal{M}_L(x, f, \varepsilon) = f(x) + (Y_1, \ldots, Y_k), \qquad Y_i \overset{\text{iid}}{\sim} \text{Lap}\left(\frac{\Delta f}{\varepsilon}\right)
$$

Independent Laplace noise on **each coordinate** of the answer, with scale = sensitivity / budget. More sensitive query or smaller $\varepsilon$ means more noise.

### 3.3 Proof That It Is $(\varepsilon, 0)$-DP

Let $x \sim y$ be neighbours, $p_x, p_y$ the output densities, $z \in \mathbb{R}^k$ any output:

$$
\begin{aligned}
\frac{p_x(z)}{p_y(z)}
&= \prod_{i=1}^{k} \frac{\exp\left(-\frac{\varepsilon |f(x)_i - z_i|}{\Delta f}\right)}{\exp\left(-\frac{\varepsilon |f(y)_i - z_i|}{\Delta f}\right)}
= \prod_{i=1}^{k} \exp\left(\frac{\varepsilon\left(|f(y)_i - z_i| - |f(x)_i - z_i|\right)}{\Delta f}\right) \\
&\le \prod_{i=1}^{k} \exp\left(\frac{\varepsilon |f(x)_i - f(y)_i|}{\Delta f}\right)
\qquad \text{(reverse triangle inequality)} \\
&= \exp\left(\frac{\varepsilon \|f(x) - f(y)\|_1}{\Delta f}\right)
\le e^{\varepsilon}
\qquad \text{(sensitivity: } \|f(x) - f(y)\|_1 \le \Delta f\text{)}
\end{aligned}
$$

The normalizing constants $\frac{\varepsilon}{2\Delta f}$ cancel because both densities use the same scale. Since the ratio is bounded pointwise, integrating over any set $S$ gives the DP inequality with $\delta = 0$; by symmetry $p_y/p_x \le e^\varepsilon$ too.

### 3.4 Accuracy

For $y = \mathcal{M}_L(x, f, \varepsilon)$ and any $\delta \in (0, 1]$:

$$
\Pr\left[ \| f(x) - y \|_\infty \ge \ln\left(\frac{k}{\delta}\right) \cdot \frac{\Delta f}{\varepsilon} \right] \le \delta
$$

With probability at least $1 - \delta$, **every** one of the $k$ coordinates is within $\ln(k/\delta) \cdot \Delta f / \varepsilon$ of its true value. The error grows only **logarithmically** in the number of coordinates. (Here $\delta$ is a failure probability of the accuracy statement, not the DP $\delta$.)

## 4. Count and Histogram Queries

### 4.1 Counting Queries

"How many instances have this characteristic?" / "What proportion satisfies a property?" - the most common statistical queries.

- **Sensitivity = 1**: adding or removing one person changes a count by at most 1.
- **$(\varepsilon, 0)$-DP release**: add $\text{Lap}(1/\varepsilon)$ noise.

### 4.2 Histogram Queries

$n$ **disjoint** ranges (bins), count the instances in each. Under add/remove adjacency, one record changes **only one bin, by 1**, so $\Delta f = 1$ for the whole vector - not $n$. Add independent $\text{Lap}(1/\varepsilon)$ noise to every bin.

This is parallel composition seen from the sensitivity side: disjoint bins mean each person "pays" in only one place.

### 4.3 Worked Example: First Names in the 2010 US Census

**Question.** With probability 95%, how accurate is a $(1, 0)$-DP Laplace histogram of the top 10,000 first names?

$k = 10{,}000$, $\delta = 0.05$, $\Delta f = 1$, $\varepsilon = 1$:

$$
\Pr\left[\|f(x) - y\|_\infty \ge \ln\left(\frac{10{,}000}{0.05}\right) \cdot \frac{1}{1}\right] \le 0.05,
\qquad \ln(200{,}000) \approx 12.21
$$

**Answer.** With probability at least 95%, **every** released count is within about **12.21 people** of the truth - on a census of over 300 million people. Excellent utility with a strong guarantee.

The list of 10,000 names must be **fixed in advance**: choosing "the top 10,000" from the private data is itself a data-dependent release and would need its own privacy accounting.

### 4.4 Changing the Adjacency: Replace-One-Record

If neighbours may also differ by **changing** one record (not only adding/removing):

| Query | Add/remove adjacency | Replace adjacency | Why |
|---|---|---|---|
| Count | 1 | 1 | the changed record either stops or starts satisfying the predicate |
| Histogram | 1 | **2** | the record leaves one bin ($-1$) and enters another ($+1$): $\ell_1$ change $= 2$ |

Sensitivity always depends on the chosen neighbouring relation - state it before computing $\Delta f$.

## 5. Report Noisy Max

Setting: $m$ counting queries that **may overlap** (one person can contribute to many, e.g. disease $\in$ {asthma, diabetes, cancer, ...}); back to add/remove adjacency.

**Algorithm.** Add independent $\text{Lap}(1/\varepsilon)$ noise to each count and **release only the index** of the largest noisy count (e.g. "the most common disease is diabetes"). This is $(\varepsilon, 0)$-DP.

**Why not release the counts?** One person can appear in all $m$ queries, so the vector of counts has $\Delta f = m$; releasing them would need $\text{Lap}(m/\varepsilon)$ noise (or cost $m\varepsilon$). Releasing only the argmax keeps the cost at $\varepsilon$ with noise of scale $1/\varepsilon$ - a much cheaper answer to a narrower question.

## 6. The Gaussian Mechanism

$$
\mathcal{M}_G(x, f, \varepsilon, \delta) \sim \mathcal{N}\left(f(x),\ \sigma^2\right),
\qquad \sigma^2 = \frac{2 \ln(1.25/\delta)\,(\Delta_2 f)^2}{\varepsilon^2}
$$

where the **$\ell_2$-sensitivity** is $\Delta_2 f = \max_{\|x - y\|_1 = 1} \|f(x) - f(y)\|_2$.

- For $0 < \delta < 1$ and $0 < \varepsilon < 1$ the mechanism is **$(\varepsilon, \delta)$-DP**.
- It **cannot** give pure DP ($\delta = 0$): Gaussian tails make the density ratio unbounded far from the mean, so with small probability more is released than $e^\varepsilon$ allows. The Laplace mechanism works with $\delta = 0$.

**Why use it then?** For a vector answer of size $k$, $\|v\|_2 \le \|v\|_1 \le \sqrt{k}\,\|v\|_2$, so $\Delta_2 f$ can be smaller than $\Delta f$ by up to a factor $\sqrt{k}$. Example (high): if one person can change each of $k = 100$ coordinates by 1, $\Delta f = 100$ but $\Delta_2 f = 10$. Gaussian noise then improves accuracy for vector queries; the benefit depends on the query and on $(\varepsilon, \delta)$.

| | Laplace | Gaussian |
|---|---|---|
| Sensitivity | $\ell_1$ ($\Delta f$) | $\ell_2$ ($\Delta_2 f$) |
| Guarantee | $(\varepsilon, 0)$ - pure | $(\varepsilon, \delta)$, $\delta > 0$ required |
| Best for | scalar / low-dimensional, disjoint answers | high-dimensional vectors with small $\ell_2$ sensitivity |

## 7. The Exponential Mechanism

### 7.1 Motivation

- "How many people have blue eyes?" - Laplace noise introduces an error but the answer stays **close** to the truth: perturbation degrades utility smoothly.
- "What is the **best** value for a threshold / price?" - a small perturbation can push the answer over a cliff and make it **useless**. Adding noise to the output is the wrong tool when the output is a **choice** from a set.

The exponential mechanism perturbs **the selection**, not the value.

### 7.2 The Auction (Running Example)

A seller sets one price for an item (an apple). Each potential buyer submits a **secret bid**; buyers with bid $\ge$ price buy at that price, others do not. Bids: **4.10€** (red), **1.00€**, **1.00€**, **1.00€**.

| Price | Buyers | Revenue |
|---|---|---|
| 1.00 | all 4 | 4.00 |
| 2.00 | red only | 2.00 |
| 4.10 | red only | **4.10** (optimal) |
| 4.20 | nobody | 0 |

Red would not like to be charged a price computed from their own bid, so they participate only if bids stay private: a **trusted curator** (a "DP black box") collects the bids and publishes a differentially private price, limiting how much any one bid can shift the distribution of the published price.

**The cliff.** Moving the price from 1.0 to 1.1 drops revenue from 4.0 to 1.1; moving from 4.1 to 4.2 drops it from 4.1 to 0. Noise of 0.1 on the optimal price is catastrophic - Laplace-on-the-answer fails here.

### 7.3 Utility Function

The revenue is the **utility** of a future outcome ("if I set the price to 1, my revenue will be 4"). Formally

$$
u : \mathbb{N}^{|\mathcal{X}|} \times \mathcal{R} \to \mathbb{R}
$$

maps a database and an outcome $r$ in the set of possible outcomes $\mathcal{R}$ (here prices 0.1 to 4.2 in steps of 0.1, $|\mathcal{R}| = 42$) to a real score.

| Price | 0.9 | 1.0 | 1.1 | 1.2 | ... | 3.9 | 4.0 | 4.1 | 4.2 |
|---|---|---|---|---|---|---|---|---|---|
| Utility | 3.6 | 4.0 | 1.1 | 1.2 | ... | 3.9 | 4.0 | 4.1 | 0 |

### 7.4 Sensitivity of the Utility

$$
\Delta u = \max_{r \in \mathcal{R}} \ \max_{\|x - y\|_1 \le 1} |u(x, r) - u(y, r)|
$$

How much **one** neighbouring change can move the utility of the **same** outcome. Slide table (neighbours $y_1 \ldots y_3$ remove a 1€ bidder, $y_4$ removes red, $y_5$ adds a bidder who bids at least 4.2):

| Price | 0.9 | 1.0 | 1.1 | 1.2 | ... | 3.9 | 4.0 | 4.1 | 4.2 |
|---|---|---|---|---|---|---|---|---|---|
| $u(x)$ | 3.6 | 4.0 | 1.1 | 1.2 | ... | 3.9 | 4.0 | 4.1 | 0 |
| $u(y_1), u(y_2), u(y_3)$ | 2.7 | 3.0 | 1.1 | 1.2 | ... | 3.9 | 4.0 | 4.1 | 0 |
| $u(y_4)$ | 2.7 | 3.0 | 0 | 0 | ... | 0 | 0 | 0 | 0 |
| $u(y_5)$ | 4.5 | 5.0 | 2.2 | 2.4 | ... | 7.8 | 8.0 | 8.2 | 4.2 |
| **Sensitivity at $r$** | 0.9 | 1.0 | 1.1 | 1.2 | ... | 3.9 | 4.0 | 4.1 | 4.2 |

At fixed price $r$, adding or removing one buyer changes revenue by **either 0 or $r$**. So the sensitivity at $r$ is $r$, and the global $\Delta u = \max \mathcal{R}$: **to have a finite global sensitivity we must cap the maximum price** (here 4.2).

### 7.5 Definition

> The exponential mechanism $\mathcal{M}_E(x, u, \mathcal{R})$ selects and outputs $r \in \mathcal{R}$ with probability proportional to
>
> $$\exp\left(\frac{\varepsilon\, u(x, r)}{2\,\Delta u}\right)$$
>
> It is $(\varepsilon, 0)$-differentially private.

- Higher-utility outcomes are **exponentially** more likely; outcomes with utility 0 (like price 4.2) become the least likely.
- The output is always a **valid element** of $\mathcal{R}$ (never a nonsense price), and it works for non-numeric outcome sets too.
- The factor 2 in the denominator pays for the normalizing constant, which itself depends on the database.
- The slide's "PROBABILITY*" row ($\varepsilon = 0.1$) shows unnormalized weights, proportional to the probability, not the probability itself.

**Worked numbers for the auction (recomputed, not in the slides, high).** With $\Delta u = 4.2$ and $|\mathcal{R}| = 42$ (uniform choice would give expected revenue 2.44):

| $\varepsilon$ | $\Pr[4.1]$ | $\Pr[1.0]$ | $\Pr[4.2]$ | Expected revenue |
|---|---|---|---|---|
| 0.1 | 0.024 | 0.024 | 0.023 | 2.46 |
| 1 | 0.029 | 0.028 | 0.018 | 2.57 |
| 5 | 0.054 | 0.050 | 0.005 | 3.01 |
| 20 | 0.167 | 0.131 | 0.000 | 3.73 |

With only four bidders, a single bid is a huge share of the data, so at reasonable $\varepsilon$ the mechanism is close to uniform. The exponential mechanism pays off when the **utility gap** between good and bad outcomes is large compared with $\Delta u / \varepsilon$ - i.e. with many participants.

### 7.6 Accuracy

With $\text{OPT}_u(x)$ the optimal utility and $\mathcal{R}_{\text{OPT}}$ the set of outcomes achieving it:

$$
\Pr\left[ u(\mathcal{M}_E(x, u, \mathcal{R})) \le \text{OPT}_u(x) - \frac{2\Delta u}{\varepsilon}\left(\ln\frac{|\mathcal{R}|}{|\mathcal{R}_{\text{OPT}}|} + t\right) \right] \le e^{-t}
$$

The loss with respect to the optimum grows only **logarithmically** with the number of outcomes. On the four-bidder auction with $\varepsilon = 1$, $t = \ln 20$ (95%) the guaranteed revenue is $4.1 - 8.4\,(\ln 42 + \ln 20) < 0$: the bound is vacuous, consistent with the table above (high).

## 8. Composition Recap

Two facts from Part I set up the sparse vector technique:

- **Group privacy**: any $(\varepsilon, 0)$-DP mechanism is $(k\varepsilon, 0)$-DP for groups of size $k$.
- **Report Noisy Max**: we cannot release all the noisy counts without increasing the sensitivity, and thus the privacy loss.

**Sequential composition**: running $\mathcal{M}_1, \ldots, \mathcal{M}_n$, each $\varepsilon_i$-DP, on the same database is (up to) $(\varepsilon_1 + \cdots + \varepsilon_n)$-DP. **Every query increases the privacy loss**, so answering a long stream of queries naively exhausts the budget.

## 9. Sparse Vector Technique

### 9.1 Scenario: Recommending a Film to Alice

A recommender has a fixed group of users with tastes similar to Alice's and a list of candidate films she has not watched. For each candidate it asks: *"Have roughly 50 similar users liked this movie?"*

Instead of releasing each (noisy) count, it answers only **above / below threshold**, and **stops at the first "above"**: $\bot, \bot, \ldots, \bot, \top$.

### 9.2 AboveThreshold

```text
AboveThreshold(D, Q, T, eps):
    T_p = T + Lap(2/eps)              # noisy threshold, drawn once
    for q_i in Q:
        v_i = Lap(4/eps)              # fresh noise per query
        if q_i(D) + v_i >= T_p:
            output TOP                # above threshold
            halt
        else:
            output BOTTOM             # below threshold
```

$D$ is the dataset, $Q$ a (possibly adaptive) stream of queries each with sensitivity 1, like counting queries. The whole run is **$\varepsilon$-DP**.

**Why it matters (high, standard result made explicit).** By sequential composition, $n$ answered queries would cost $n\varepsilon$. Here the cost is **$\varepsilon$ total, independent of how many $\bot$ answers are given**: only the single $\top$ "spends" budget. The below-threshold answers are almost free because they reveal so little. This is what makes it practical for scanning long candidate lists.

### 9.3 Who Is Protected?

Not Alice (she receives the recommendation) but the **users in her similarity group** whose likes support it. If Bob is in that group, the guarantee compares the process **with Bob's likes** and the same process **with Bob's entire record removed**: the probability of any released outcome differs by at most a factor $e^{\varepsilon}$. This limits how much Alice can learn about Bob's participation or preferences from her recommendation.

### 9.4 Variants

By rescaling the Laplace noise and the threshold the algorithm can be extended while staying $\varepsilon$-DP:

| Variant | Outputs | Cost |
|---|---|---|
| **AboveThreshold** | first $\top$, then halt | baseline noise |
| **Sparse** | up to $c$ above-threshold results | more noise (scales with $c$) |
| **NumericSparse** | up to $c$ above-threshold results **and their noisy values** | even more noise, part of the budget spent on the values |

More output means more noise and lower accuracy - the usual privacy-utility trade-off. Pseudocode: Dwork and Roth, *The Algorithmic Foundations of Differential Privacy* (see Sources).

## 10. Mechanism Selection Cheat Sheet

| Task | Mechanism | Noise / rule | Guarantee |
|---|---|---|---|
| Numeric answer, low dimension | Laplace | $\text{Lap}(\Delta f / \varepsilon)$ per coordinate | $(\varepsilon, 0)$ |
| Count | Laplace | $\text{Lap}(1/\varepsilon)$ | $(\varepsilon, 0)$ |
| Histogram (disjoint bins) | Laplace | $\text{Lap}(1/\varepsilon)$ per bin (add/remove) | $(\varepsilon, 0)$ |
| Which of $m$ overlapping counts is largest | Report Noisy Max | $\text{Lap}(1/\varepsilon)$, release index only | $(\varepsilon, 0)$ |
| High-dimensional vector, small $\ell_2$ sensitivity | Gaussian | $\sigma^2 = 2\ln(1.25/\delta)(\Delta_2 f)^2/\varepsilon^2$ | $(\varepsilon, \delta)$ |
| Choose best element of a set (price, threshold, label) | Exponential | $\Pr[r] \propto \exp(\varepsilon u / 2\Delta u)$ | $(\varepsilon, 0)$ |
| Long stream, few positives expected | Sparse vector (AboveThreshold) | $T + \text{Lap}(2/\varepsilon)$, $\text{Lap}(4/\varepsilon)$ per query | $(\varepsilon, 0)$ total |

Exam hooks:

- Define $\ell_1$-sensitivity; compute it for count (1), histogram (1 or 2 depending on adjacency), $m$ overlapping counts ($m$), auction utility (max price).
- Define the Laplace mechanism and reproduce the $(\varepsilon, 0)$-DP proof step by step.
- Apply the accuracy bound: first-names example, $\ln(200{,}000) \approx 12.21$.
- Explain why Report Noisy Max releases only the index.
- Gaussian vs Laplace: $\ell_2$ vs $\ell_1$, why $\delta > 0$ is unavoidable, the $\sqrt{k}$ gap.
- Why the exponential mechanism exists (cliffs in utility), its formula, the meaning of $\Delta u$, why the price must be capped.
- AboveThreshold pseudocode, the two noise scales, why the cost does not grow with the number of $\bot$, who is protected.

## 11. Source Corrections

The raw is an OCR'd slide export; fixed silently in the formulas above:

- The second $\ell_1$-sensitivity slide drops the constraint $\|x - y\|_1 = 1$ under the max; without it the maximum is over all database pairs and is meaningless.
- Gaussian mechanism mean written as $x$; it is $f(x)$. The $\bar{\delta}$ is plain $\delta$.
- "smaller ... by a factor of $\sqrt{\text{sqrt}(k)}$" is a duplicated root: the factor is $\sqrt{k}$.
- Laplace proof: the line separator `\&` is an OCR artifact of `\\` (aligned environment).
- Exponential mechanism: `\varepsilon^{}u(x,r)/(2^{}\Delta u)` means $\varepsilon\, u(x,r) / (2\Delta u)$.
- "(ln 3 - 0) differentially private" means $(\ln 3, 0)$-DP; "Total gain = pricebuyers = 14" means price $\times$ buyers $= 1 \times 4$.
- AboveThreshold "output T" is the symbol $\top$ (above threshold), paired with $\bot$.
- "only 4 buys" in the price-bar figure means "only the 4.10 bidder buys".

## Related

- [[Differential Privacy Foundations]] - Part I: definition, privacy loss, composition, group privacy, post-processing that these mechanisms instantiate
- [[Recommender Systems]] - the similarity-group recommendation setting of the sparse vector example and DP popularity counts
- [[Macrodata Protection]] - count and frequency tables, the classical (non-DP) counterpart of histogram queries
- [[Privacy Evaluation Metrics]] - the privacy-utility trade-off measured by the accuracy bounds here

## Sources

- [[Differetial_Privacy2]] (AY 2026/2027)
- C. Dwork and A. Roth, *The Algorithmic Foundations of Differential Privacy*, Foundations and Trends in TCS, 2014 - https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf (referenced by the slides for Sparse/NumericSparse pseudocode)
