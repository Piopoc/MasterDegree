---
type: course-note
date: 2026-11-17
tags:
  - course
  - ppia
  - differential-privacy
  - randomized-response
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Differential_Privacy1]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering **Differential Privacy, Part I** (59-slide lecture): why anonymization and aggregate statistics fail (Fundamental Law of Information Recovery, differencing attack, query auditing, "just a few" policy), the core intuition (learn as much about me as if I had not contributed), randomized response with its full derivation, the formal machinery (probability simplex, randomized algorithm $M: A \to \Delta(B)$, database histograms, $\ell_1$ distance, neighbouring databases), the $(\varepsilon,\delta)$-DP definition, how to choose $\varepsilon$ and $\delta$, privacy loss, and the five properties (linkage neutralization, quantified loss, composition, group privacy, post-processing). Created 2026-11-17 from `raw/Differential_Privacy1.md`. Concrete mechanisms (Laplace, sensitivity) are **not** in this part - they are covered in Part II (see Related). Worked numeric examples were added to make the slides self-explanatory; slide typos are corrected silently in the formulas and listed in Section 12.

## 1. Why We Need Something New

### 1.1 The Fundamental Law of Information Recovery

> Overly accurate answers to too many questions will destroy privacy in a spectacular way.

**Corollary: data cannot be fully anonymized and remain useful.** This is a provable result and it holds for *every* privacy-preserving analysis technique, not just DP. It frames the whole course as a **trade-off**: privacy and utility cannot both be maximal.

- The **richer** the data, the easier **linkage attacks** become (quasi-identifiers + external data, see [[Microdata Disclosure Risks and Sanitization]]).
- **Fully** anonymized data would require removing all quasi-identifiers - and then nothing useful is left.

DP sidesteps this: **being differentially private is a property of the data access mechanism**, not of the data, so it holds regardless of what background knowledge the attacker has for a linkage attack.

### 1.2 Re-identification Is Not the Only Risk

[[k-Anonymity]] reduces the risk of re-identification, but remains vulnerable to **background knowledge** and **skewness** attacks (the reason ℓ-diversity and t-closeness exist). Protecting identities is not the same as protecting what can be *learned* about a person.

### 1.3 Large Datasets and Aggregates Do Not Protect: the Differencing Attack

Intuition says "if I only answer queries over many people, nobody is exposed". Wrong.

**Setup:** the attacker knows the target X is in the database, which holds a binary sensitive attribute $b$.

- $q_1$ = how many subjects have $b = \text{true}$?
- $q_2$ = how many subjects **not named X** have $b = \text{true}$?

$$
q_1 - q_2 = \begin{cases} 1 & \Rightarrow b_X = \text{true} \\ 0 & \Rightarrow b_X = \text{false} \end{cases}
$$

Both queries are over large sets, both look harmless, yet together they reveal X exactly. The slide example on the `adult` dataset:

```python
adult["income"].sum()                              # 6,039,000
adult[adult["name"] != "Steven"]["income"].sum()   # 6,020,000
# Steven's income = 6,039,000 - 6,020,000 = 19,000
```

### 1.4 Query Auditing Does Not Fix It

Idea: track past queries and refuse any query that, combined with earlier ones, would disclose something (e.g. refuse $q_2$). Two limitations:

1. **Refusing is itself informative** - "I won't answer" tells the attacker the answer would have been revealing.
2. **Computationally infeasible** in general - checking all combinations of past queries does not scale.

### 1.5 Summary Statistics Are Not Safe

The differencing attack shows that aggregates do not give privacy by themselves. Moreover, **reconstruction attacks** (via record linkage) can rebuild secret information in **linear time** from enough summary statistics. (Cross-reference: these are the same weaknesses listed for query-based releases in [[Macrodata Protection]], where DP is named as the protection technique.)

### 1.6 Ordinary Facts Are Not Harmless

Almost every fact leaks something. *"The subject bought bread regularly for years, then suddenly started buying it rarely"* looks innocent - but it may be the trace of *"the subject has been diagnosed with type 2 diabetes"*. You cannot pre-classify attributes as safe or sensitive.

### 1.7 The "Just a Few" Policy

Tempting compromise: sacrifice the privacy of "just a few" to publish more informative data. Problems:

- the "few" are most likely the **outliers** - exactly the people who most need protection;
- nobody can say **how many** "a few" is.

DP reaches comparable utility **without this moral compromise**: everyone gets the same quantified guarantee.

## 2. The Core Idea

> How can I protect my privacy completely? By **not putting my record** in the database.

DP turns this into a goal: *can an analyst learn about me only as much as they would if I had not contributed at all?* My participation vs non-participation corresponds to **two databases, one with me and one without**. If the analysis outputs (nearly) the same thing on both, my participation costs me (nearly) nothing.

**Coffee trial example.** A study tests whether coffee drinking correlates with being restless. If proven, an insurer might raise premiums for coffee drinkers, so I might refuse to take part. DP aims to produce statistics that are **(almost) the same whether or not I was in the study** - so joining cannot be used as evidence against me specifically.

Note the subtlety: if the correlation is real, the study *will* find it, and the insurer may still raise premiums for all coffee drinkers. DP does not prevent learning facts about the population; it ensures that what is learned does not depend on **my** record.

Two framing statements:

- **DP is a definition, not an algorithm.** For a task $T$ and a value of $\varepsilon$ there are many algorithms that achieve $T$ in an $\varepsilon$-DP way.
- **DP gives privacy by process**: it introduces **randomness** into the answer.

## 3. Randomized Response (Coin Toss)

A technique from the social sciences for surveying embarrassing or illegal behaviour ("Have you ever done X?").

```text
Flip a coin.
  Tails -> answer truthfully.
  Heads -> flip again:
      Tails -> answer "YES"
      Heads -> answer "NO"
```

**Plausible deniability**: any single answer may have been forced by the coin, so no individual answer proves anything. Yet the *population* proportion can still be estimated.

### 3.1 Response Probabilities

| True value | P(answer YES) | P(answer NO) |
|---|---|---|
| YES | $\frac{1}{2} + \frac{1}{4} = \frac{3}{4}$ (truthful + forced YES) | $\frac{1}{4}$ (forced NO = "false NO") |
| NO | $\frac{1}{4}$ (forced YES = "false YES") | $\frac{1}{2} + \frac{1}{4} = \frac{3}{4}$ |

Overall, 25% of respondents give a forced NO and 25% a forced YES, whatever their truth; the remaining 50% answer truthfully.

### 3.2 Estimating the True Proportion

Let $p$ = true proportion of YES, $Y$ = observed proportion of YES answers.

- Among the $p$ true-YES: $\frac{1}{2}$ say YES truthfully, $\frac{1}{4}$ say YES by the second coin -> contribution $\frac{3}{4}p$.
- Among the $1-p$ true-NO: $\frac{1}{4}$ say YES by the second coin -> contribution $\frac{1}{4}(1-p)$.

$$
Y = \frac{3}{4}p + \frac{1}{4}(1-p) = \frac{1}{4} + \frac{1}{2}p
\qquad\Longrightarrow\qquad
p = 2Y - \frac{1}{2}
$$

**Worked example.** 1,000 respondents, 600 answer YES -> $Y = 0.6$ -> $p = 2(0.6) - 0.5 = 0.7$: about 70% truly did X, though no single "YES" incriminates anyone. Sanity checks: $p = 0 \Rightarrow Y = 0.25$ (only forced YES); $p = 1 \Rightarrow Y = 0.75$.

The price of privacy is **variance**: the estimate is noisier than direct questioning, and the noise shrinks only as the sample grows.

## 4. Formal Building Blocks

### 4.1 Probability Simplex

For a discrete set $B$:

$$
\Delta(B) = \left\{ x \in \mathbb{R}^{|B|} \;:\; x_i \ge 0 \ \forall i,\ \ \sum_{i=1}^{|B|} x_i = 1 \right\}
$$

Simply: **all probability distributions over $B$** - vectors of non-negative numbers summing to 1. For $B = \{Y, N\}$, $\Delta(B)$ is every pair $(q, 1-q)$ with $q \in [0,1]$.

### 4.2 Randomized Algorithm

A randomized algorithm $\mathcal{M}$ with domain $A$ and discrete range $B$ is associated with a mapping

$$
M : A \to \Delta(B)
$$

On input $a \in A$, $\mathcal{M}(a) = b$ with probability $(M(a))_b$. The probability space is over the **coin flips of the algorithm**, not over the data.

Reading the slide diagrams: each input value $a$ is assigned **its own** distribution over the outputs - e.g. input $0.1$ might produce output $A$ with probability $0.4$, $B$ with $0.1$, etc. - and **a different input can have a different simplex** over the output range. Running the algorithm = sampling from the distribution attached to the input.

**Randomized response in this language**: $A = B = \{Y, N\}$,

$$
M(Y) = (0.75,\ 0.25), \qquad M(N) = (0.25,\ 0.75)
$$

### 4.3 The Output Is a Distribution

A single output of a randomized algorithm is not interesting (one coin-toss answer may be pure noise). What matters is the **distribution of outputs**: from the noisy distribution of YES/NO we recovered the true proportion. From now on $\mathcal{M}(x)$ is read as a **distribution** (e.g. the distribution of weights in a population, of people committing an illegal act).

### 4.4 Databases as Histograms

A database $x$ is a collection of records from a universe $\mathcal{X}$ (all possible record types). It is represented as a vector

$$
x \in \mathbb{N}^{|\mathcal{X}|}, \qquad x_i = \text{number of records of type } i \in \mathcal{X}
$$

i.e. the **histogram** of the database: how many records fall into each equivalence class of the universe.

**Example.** Universe $\mathcal{X} = \{\text{(smoker, sick)}, \text{(smoker, healthy)}, \text{(non-smoker, sick)}, \text{(non-smoker, healthy)}\}$. A database with 3, 5, 1, 11 such people is $x = (3, 5, 1, 11)$.

### 4.5 Distance Between Databases

$$
\|x\|_1 = \sum_{i=1}^{|\mathcal{X}|} |x_i|
$$

- $\|x\|_1$ = **size** of the database (number of records): $20$ in the example.
- $\|x - y\|_1$ = **how many records differ** between $x$ and $y$.

With $y = (3, 5, 1, 10)$ (one healthy non-smoker removed): $\|x - y\|_1 = 1$.

### 4.6 Neighbouring Databases

$x$ and $y$ are **neighbouring** if they differ in exactly one record: $\|x - y\|_1 = 1$. In histogram form this means **adding or removing one person** - exactly "me in the database vs me not in it" from Section 2.

Why it gives privacy: if the output is (almost) the same on any two neighbouring databases, **any single record could be fake** and we would observe (almost) the same result - so the output cannot be pinned on any individual.

(Note: *changing* one person's record is a removal plus an addition, so $\|x - y\|_1 = 2$ in the histogram view - high.)

## 5. The Definition

Intuitively: **a randomized algorithm is DP if it behaves similarly on similar databases**, where "behaves similarly" means its output distributions are close.

> A randomized algorithm $\mathcal{M}$ with domain $\mathbb{N}^{|\mathcal{X}|}$ is **$(\varepsilon, \delta)$-differentially private** if for all $S \subseteq \text{Range}(\mathcal{M})$ and for all $x, y \in \mathbb{N}^{|\mathcal{X}|}$ with $\|x - y\|_1 \le 1$:
>
> $$\Pr[\mathcal{M}(x) \in S] \le e^{\varepsilon}\, \Pr[\mathcal{M}(y) \in S] + \delta$$
>
> The probability is over the coin flips of $\mathcal{M}$. If $\delta = 0$, $\mathcal{M}$ is **$\varepsilon$-differentially private** (pure DP).

How to read it:

- $S$ is **any** set of outcomes - any event the attacker might observe ("the reported count is above 100", "the answer is YES").
- The bound must hold for **every** pair of neighbours - in particular the worst-case individual.
- $e^\varepsilon$ is a **multiplicative** factor: an event can become at most $e^\varepsilon$ times more likely because of one person.

### 5.1 The Two-Sided Bound (Pure DP)

With $\delta = 0$, neighbourhood is symmetric, so the inequality also holds with $x$ and $y$ swapped: $\Pr[\mathcal{M}(y) \in S] \le e^{\varepsilon}\Pr[\mathcal{M}(x) \in S]$. Combining:

$$
e^{-\varepsilon}\Pr[\mathcal{M}(y) \in S] \;\le\; \Pr[\mathcal{M}(x) \in S] \;\le\; e^{\varepsilon}\Pr[\mathcal{M}(y) \in S]
$$

The probabilities on the two neighbours are within a factor $e^{\pm\varepsilon}$ of each other.

## 6. Choosing $\varepsilon$

**Limit $\varepsilon \to 0$**: $e^0 = 1$, so $\Pr[\mathcal{M}(x) \in S] = \Pr[\mathcal{M}(y) \in S]$ - two different databases, identical output. **Maximum privacy, zero utility**: the output does not depend on the data at all (it is pure noise).

**Large $\varepsilon$**: weak guarantee, more accurate answers.

| $\varepsilon$ | $e^\varepsilon$ (max likelihood ratio between neighbours) |
|---|---|
| 0.1 | 1.105 |
| $\ln 2 \approx 0.69$ | 2 |
| 1 | 2.718 |
| $\ln 3 \approx 1.10$ | 3 |
| 5 | 148.4 |

Points from the lecture:

- **No precise rule of thumb exists.** If $\varepsilon$ is very small, a mechanism that fails to be $(\varepsilon, 0)$-DP may still be $(2\varepsilon, 0)$-DP, which may be acceptable.
- **Failing a large $\varepsilon$** (e.g. not being $(15, 0)$-DP) means there exist neighbours $x, y$ and an output $o$ whose probability ratio is huge. But if $o$ is extremely unlikely, then $x, y$ are "strange", artificial databases, and the attacker may not gain much in practice.
- **Common values: between 0.1 and 5; typical choices $\ln 2$ and $\ln 3$.**
- Literature survey (Hsu et al., 2014) shows the spread across applications, e.g. 0.1 for logistic regression, 0.1-0.4 for genome data, $\ln 2$ / $\ln 5$ / $\ln 10$ for click counts, $\ln 3$ for Kalman filtering, 0.5-3 for recommender systems, up to 0.01-10 for record linkage.

## 7. Choosing $\delta$

$\delta$ is the **probability of failing** to protect privacy - a relaxation saying that for some rare outcomes the privacy bound may be larger than $e^\varepsilon$.

- $\delta = 0$: for **every** run, the output is (almost) equally likely on every neighbouring database simultaneously.
- $\delta > 0$: for every pair of neighbours, it is **unlikely (probability at most $\delta$)** that the observed output is much more or much less likely under $x$ than under $y$.

**Why $\delta$ must be tiny.** A typical $\delta$ is **far smaller than the inverse of any polynomial in the database size**. Counter-example: with $\delta = 1/\|x\|_1$, a mechanism that publishes **one random record in full** satisfies the definition - each person is exposed only with probability $1/n$. That is the "just a few" policy again, dressed up as DP. So $\delta \ge 1/n$ gives no real protection.

| Parameter | Meaning | Effect when lower |
|---|---|---|
| $\varepsilon$ | **privacy budget**: max influence of one record | more noise, more protection, less utility |
| $\delta$ | probability the $\varepsilon$ bound fails | fewer "catastrophic" outcomes allowed |

## 8. Privacy Loss

For an observed output $o$, the **privacy loss** is the log-ratio of its probability under the two neighbours:

$$
\mathcal{L}^{o}_{\mathcal{M}(x) \| \mathcal{M}(y)} = \ln\left( \frac{\Pr[\mathcal{M}(x) = o]}{\Pr[\mathcal{M}(y) = o]} \right)
$$

It measures how much observing $o$ helps distinguish "database with me" from "database without me". Key lemma (as stated in the lecture):

$$
\left| \mathcal{L}^{o}_{\mathcal{M}(x) \| \mathcal{M}(y)} \right| \le \varepsilon \quad \text{with probability } 1 - \delta
$$

For pure DP ($\delta = 0$) the loss is bounded by $\varepsilon$ **always**.

### 8.1 Randomized Response Is $(\ln 3, 0)$-DP

Two neighbouring single-record databases $b$ and $b'$; without loss of generality $b = Y$, $b' = N$. From Section 3.1:

$$
\Pr[M(b) = Y] = \frac{3}{4}, \qquad \Pr[M(b') = Y] = \frac{1}{4}
\qquad\Longrightarrow\qquad
\varepsilon \ge \left| \ln\frac{3/4}{1/4} \right| = \ln 3
$$

The output $N$ gives the same ratio ($\frac{3/4}{1/4}$ the other way), so the worst case is exactly $\ln 3$: **the coin-toss algorithm is $(\ln 3, 0)$-differentially private**. Interpretation: whatever a respondent answers, it is at most **3 times** more likely under one truth than under the other.

(Generalization, not in the slides: with first-coin truth probability $q$, $\Pr[Y|Y] = q + \frac{1-q}{2}$, $\Pr[Y|N] = \frac{1-q}{2}$, so $\varepsilon = \ln\frac{1+q}{1-q}$; $q = \frac{1}{2}$ gives $\ln 3$ - high.)

## 9. Properties of DP

### 9.1 Composition

The combination of DP mechanisms is again DP, so simple mechanisms are **building blocks** for complex DP analyses. The lecture uses a circuit analogy (batteries and bulbs):

| Arrangement | Situation | Total guarantee |
|---|---|---|
| **Series (sequential)** | independent mechanisms run on the **same** data | $\varepsilon_{\text{tot}} = \sum_i \varepsilon_i$ (upper bound) |
| **Parallel** | independent mechanisms on **disjoint partitions** of the data | $\varepsilon_{\text{tot}} = \max_i \varepsilon_i$ |

**Example.** Three queries at $\varepsilon = 0.5$ each on the whole database -> total $1.5$. The same three queries, each on a different disjoint age group -> total $0.5$, because each person is in only one partition.

This is why $\varepsilon$ is called a **budget**: every query spends some, and once it is spent, further answers would exceed the guarantee - a direct answer to the differencing attack and to the Fundamental Law (too many accurate answers).

### 9.2 Group Privacy

What if the database contains a family of $k$ people rather than one?

> **Theorem.** Any $(\varepsilon, 0)$-DP mechanism is $(k\varepsilon, 0)$-DP for groups of size $k$: for all $\|x - y\|_1 \le k$ and all $S \subseteq \text{Range}(\mathcal{M})$,
>
> $$\Pr[\mathcal{M}(x) \in S] \le e^{k\varepsilon}\, \Pr[\mathcal{M}(y) \in S]$$

Why: walk from $x$ to $y$ one record at a time through $k$ neighbouring steps; each step costs at most a factor $e^\varepsilon$, so $e^{\varepsilon} \cdots e^{\varepsilon} = e^{k\varepsilon}$. Protection **degrades linearly in $\varepsilon$** (exponentially in the ratio) with group size. Example: randomized response with $\varepsilon = \ln 3$ and a family of 2 -> ratio up to $3^2 = 9$.

### 9.3 Post-Processing

> Let $\mathcal{M} : \mathbb{N}^{|\mathcal{X}|} \to R$ be $(\varepsilon, \delta)$-DP and $f : R \to R'$ an arbitrary (even randomized) mapping. Then $f \circ \mathcal{M}$ is $(\varepsilon, \delta)$-DP.

**DP is immune to post-processing**: no computation on the output - however clever, and without access to the raw data - can make it *less* private. Analysts can round, plot, feed the results to a model, or combine them with public data without breaking the guarantee. (Contrast: k-anonymous releases can be broken by combining them with background knowledge.)

### 9.4 Summary of Properties

1. **Automatic neutralization of linkage attacks** (also on aggregate statistics) - independent of the attacker's auxiliary data.
2. **Quantified privacy loss** - $\varepsilon$ (and $\delta$) are numbers, comparable across mechanisms.
3. **Composition** - budgets add up (sequential) or take the max (parallel).
4. **Group privacy** - graceful, quantified degradation for groups ($k\varepsilon$).
5. **Closure under post-processing**.

## 10. DP vs Earlier Techniques in This Course

| | k-Anonymity (and ℓ-diversity / t-closeness) | Differential Privacy |
|---|---|---|
| Property of | the **released data** | the **access mechanism** |
| Threat model | assumed quasi-identifiers and attacker knowledge | any background knowledge |
| Method | generalization / suppression | randomness (noise) |
| Guarantee | syntactic (each q-block $\ge k$) | quantitative ($\varepsilon$, $\delta$) |
| Multiple releases | can be combined to break it | composes with a known total budget |
| Post-processing | may leak with extra knowledge | provably safe |

Randomized response also appears in the perturbative catalog of [[Microdata Masking and Synthetic Data]] (there in its "one question chosen from a set" variant); this note is its canonical DP treatment.

## 11. Exam Hooks

- State the Fundamental Law and its corollary; explain why DP is a property of the **mechanism**.
- Describe a differencing attack and why query auditing fails (refusal leaks; infeasible).
- Randomized response: probability table, derivation $Y = \frac{1}{4} + \frac{p}{2}$, $p = 2Y - \frac{1}{2}$, and proof it is $(\ln 3, 0)$-DP.
- Define simplex, randomized algorithm $M: A \to \Delta(B)$, histogram representation, $\ell_1$ norm/distance, neighbouring databases.
- Write the $(\varepsilon, \delta)$-DP definition from memory, explain every quantifier, derive the two-sided bound.
- $\varepsilon$: budget, $\varepsilon \to 0$ = no utility, typical 0.1-5, $\ln 2$ / $\ln 3$. $\delta$: failure probability, must be $\ll 1/\text{poly}(n)$, why $\delta = 1/n$ is the "just a few" policy.
- Privacy loss formula and lemma.
- Composition (sum vs max), group privacy ($k\varepsilon$, with the chain-of-neighbours argument), post-processing.

## 12. Source Corrections

Slide typos fixed in the formulas above (the raw is an OCR'd PDF export):

- $\ell_1$ norm: the slide sums $|x_1|$ for $i = 1 \ldots |x|$; correct is $\sum_{i=1}^{|\mathcal{X}|} |x_i|$.
- Post-processing domain written $\mathbb{N}^{|\mathcal{H}|}$; it is $\mathbb{N}^{|\mathcal{X}|}$.
- The bars/overlines on $\delta$ ($\bar{\delta}$, $\bar{\mathbf{o}}$) are OCR artifacts of plain $\delta$.
- The slide sentence "among the NOs 1/4 are yes, among YESs only 3/4 are true" is loose wording: the precise statement is the conditional table in Section 3.1 (each group has a 1/4 chance of a forced wrong answer).
- The slide on $\delta = 1/\|x\|_1$ puts "preserve the privacy" in quotes ironically: such a $\delta$ allows publishing a few records in clear.

## Related

- [[Differential Privacy Mechanisms]] - Part II: sensitivity, Laplace, Gaussian, exponential and sparse vector mechanisms that instantiate this definition
- [[k-Anonymity]] - syntactic anonymization and its background-knowledge and skewness attacks, which DP is designed to avoid
- [[Macrodata Protection]] - query-based releases, differencing and reconstruction attacks, DP as protection technique
- [[Microdata Masking and Synthetic Data]] - perturbative masking, including randomized response
- [[Privacy Evaluation Metrics]] - the privacy/utility trade-off that $\varepsilon$ parameterizes
- [[Recommender Systems]] - popularity counts as a DP-friendly release

## Sources

- [[Differential_Privacy1]] (AY 2026/2027)
- J. Hsu et al., "Differential Privacy: An Economic Method for Choosing Epsilon", IEEE CSF 2014 - epsilon survey table, cited via the slides above
