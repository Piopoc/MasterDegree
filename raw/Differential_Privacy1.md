---
type: source
title: "Copia di DP1.pdf"
sources:
  - "raw/sources/Copia-di-DP1.pdf"
pages: 59
source_pages: 59
missing_pages: []
created: 2026-11-16
updated: 2026-11-16
---

<!-- page 1 -->

UNIVERSITAS PATAVINA LIBER
1222

UNIVERSITÀ DI PADOVA

Dipartimento
di Ingegneria
dell'Informazione

<img src="images/bbox_706_38_966_227.jpg" />

# DIFFERENTIAL PRIVACY: PART I

PRIVACY PRESERVING INFORMATION ACCESS

Master Degree in Cybersecurity, Master Degree in Computer Engineering, and Master Degree In International Cybersecurity and Cyberintelligence

A.Y. 2026/2027

GUGLIELMO FAGGIOLI

Intelligent Interactive Information Access (IIIA) Hub
Department of Information Engineering
University of Padua

<img src="images/bbox_867_780_966_955.jpg" />


<!-- page 2 -->

## FUNDAMENTAL LAW OF INFORMATION RECOVERY

Overly accurate answers to too many questions will destroy privacy in a spectacular way.

Corollary: Data Cannot be Fully Anonymized and Remain Useful.

This result, which can be mathematically proven, applies to all techniques for privacy-preserving data analysis, and not just to differential privacy.


<!-- page 3 -->

## DATA CANNOT BE FULLY ANONYMIZED AND REMAIN USEFUL

The richer the data, the easier it is to carry out linkage attacks.

Conversely, fully anonymized data require complete removal of quasi-identifiers, making it impossible to extract useful information.

Differential privacy neutralizes linkage attacks: being differentially private is a property of the data access mechanism, and is unrelated to the presence or absence of information available to the malevolent user to carry that might enable linkage attacks.


<!-- page 4 -->

## RE-IDENTIFICATION IS NOT THE ONLY RISK

We already know that: k-anonymization should reduce the risk of re-identification, but it suffers from the background knowledge and skewness vulnerabilities.


<!-- page 5 -->

## QUERIES OVER LARGE SETS ARE NOT PROTECTIVE

We might assume that it is easy to apply techniques such as k-anonymity to large datasets and this might help protecting private information.

Nevertheless, even large datasets are vulnerable to differencing attack.


<!-- page 6 -->

## QUERIES OVER LARGE SETS ARE NOT PROTECTIVE

differencing attack: assume that we know the subject of interest, called X, is in a database which contains a (binary) sensitive attribute b.

By running two queries  $q_{1}$ and  $q_{2}$, defined as follows:

q_{1} tells us how many subjects on the dataset have the attribute b = true

q_{2} tells us how many subjects, not named X, have the attribute b = true.

the difference between the output for  $q_{1}$ and  $q_{2}$ tells us the value of b for the subject of interest (if it's 0, then b=true for the subject of interest).


<!-- page 7 -->

## A DIFFERENCING ATTACK

adult["income"].sum()

> 6,039,000

adult[adult["name"] != "Steven"]["income"].sum()

> 6,020,000

Steven income: 19,000


<!-- page 8 -->

## QUERY AUDITING IS PROBLEMATIC

We might consider to audit queries, refusing to answer to queries which might disclose information, based on queries already executed (e.g., we might not answer to  $q_{2}$ in the previous example).

This has 2 limitations:

not disclosing something is still informative

it is infeasible computationally


<!-- page 9 -->

## SUMMARY STATISTICS ARE NOT SAFE

The differencing attack highlights why summary statistics do not provide privacy.

Besides, reconstruction attacks done through record linkage allow to reconstruct secret information in linear time.


<!-- page 10 -->

## ORDINARY FACTS SHOULD NOT BE NEGLECTED

There is almost no fact that does not disclose information about the subject.

“The subject was used to buy bread regularly for several years, then switched suddenly to buy it rarely.”


<!-- page 11 -->

## ORDINARY FACTS SHOULD NOT BE NEGLECTED

There is almost no fact that does not disclose information about the subject.

“The subject was used to buy bread regularly for several years, then switched suddenly to buy it rarely.”

<img src="images/bbox_485_477_513_597.jpg" alt="In the center of the image is a dark gray downward-pointing arrow, composed of a vertical rectangle at the top and a triangular arrowhead at the bottom. The background is pure white, with no other objects, scenes, or text. The layout is centered and symmetrical, with a simple and clear design, commonly used to indicate direction or guide the visual focus." />

“The subject has been diagnosed with type 2 diabetes”


<!-- page 12 -->

## “JUST A FEW” POLICY

Sacrifice the privacy of “just a few”, to provide more informative data.

Outliers are most likely those that will be in the “just a few” set: they are also likely those for which the privacy should be protected the most.

We do not know what “just a few” is.

Differential privacy allows to achieve similar results, without the need for moral compromises.


<!-- page 13 -->

## THE PROBLEM

How can I protect my privacy completely? By not putting my record in the database!

Can we build an approach that allows a data analyst to learn about me as much information as if I did not contribute at all to the database?

My contribution/Non contribution corresponds to have two databases: one with me and one without.


<!-- page 14 -->

## AN EXAMPLE

Assume I can somehow be harmed by a certain trial - for example a trial about the correlation between the consumption of coffee and being restless.

Being restless is a risk factor: an insurance company might decide to increase the insurance premium to coffee drinkers if the correlation was proved.

I therefore might prefer avoid taking part into the trial to avoid giving evidence of such correlation.

Differential privacy aims at producing statistics which are the same independently of whether or not I was in the study.


<!-- page 15 -->

## DIFFERENTIAL PRIVACY IS NOT AN ALGORITHM

Differential privacy is a definition, not an algorithm.

For a given computational task T and a given value of  $\varepsilon$ there will be many algorithms falling into the differentially private framework for achieving T in an  $\varepsilon$-differentially private manner


<!-- page 16 -->

## DIFFERENTIAL PRIVACY

Differential privacy will provide privacy by process; in particular it will introduce randomness.


<!-- page 17 -->

## COIN TOSS (RANDOMIZED RESPONSE)

Privacy by randomness algorithm.

Flip a coin:

if “tail”, respond truthfully

if “head”, flip coin again:

if “tail”, respond “YES”

if “head”, respond “NO”


<!-- page 18 -->

## DENIABILITY OF THE OUTCOME

Technique developed by social scientists to collect information about embarrassing or illegal behaviours: the privacy is achieved thanks to the fact that the outcome is always deniable.

the probability of observing a “false NO” or a “false YES” is  $\frac{1}{4}$ each. then, we know that among the “NO”s  $\frac{1}{4}$ are yes and similarly, among “YES”s only  $\frac{3}{4}$ are true.


<!-- page 19 -->

## DENIABILITY OF THE OUTCOME

<img src="images/bbox_136_261_783_753.jpg" alt="This is a bar chart. The horizontal axis is labeled “25% Forced NO” and “25% Forced YES”. The legend indicates that pink represents “Tossed Head” and green represents “Tossed Tail”. The left bar is composed of pink and red, while the right bar is composed of green and light green. The chart includes coordinate axes and diagonal lines, with a white background and a clean layout." />


<!-- page 20 -->

## DENIABILITY OF THE OUTCOME

How many “true YES” are there in the answers?

Called Y the proportion of “YES” observed and p is the proportion of “true YES”:

Y =


<!-- page 21 -->

## DENIABILITY OF THE OUTCOME

How many “true YES” are there in the answers?

Called Y the proportion of “YES” observed and p is the proportion of “true YES”:

$$ Y=\boxed{3/4p} $$

Among the p true yes  $\frac{1}{2}$ answered yes freely, and  $\frac{1}{4}$ answered yes thanks to the second coin toss


<!-- page 22 -->

## DENIABILITY OF THE OUTCOME

How many “true YES” are there in the answers?

Called Y the proportion of “YES” observed and p is the proportion of “true YES”:

$$ Y=3/4p+\boxed{1/4(1-p)}=1/4+2/4p $$

Among the 1-p true no  $\frac{1}{4}$ answered answered yes thanks to the second coin toss


<!-- page 23 -->

## DENIABILITY OF THE OUTCOME

How many “true YES” are there in the answers?

Called Y the proportion of “YES” observed and p is the proportion of “true YES”:

$$ Y=3/4p+1/4(1-p)=1/4+2/4p $$

and thus:

$$ p=2Y-1/2 $$


<!-- page 24 -->

## PROBABILITY SIMPLEX

Probability Simplex: Given a discrete set B, the probability simplex over B, denoted  $\Delta(B)$ is defined to be:

$$ \Delta(B)=\left\{x\in\mathbb{R}^{|B|}\;x_{i}\geq0\;\mathrm{f o r~a l l}\;i\;\mathrm{a n d}\;\sum_{i=1}^{|B|}x_{i}=1\right\} $$

the probability simplex is nothing more than a vector of real numbers between 0 and 1 which add up to 1.


<!-- page 25 -->

## RANDOMIZED ALGORITHM

Randomized Algorithm: A randomized algorithm $\mathcal{M}$ with domain $A$ and discrete range $B$ is associated with a mapping $M: A \to \Delta(B)$.

On input $a \in A$, the algorithm $\mathcal{M}$ outputs $\mathcal{M}(a) = b$ with probability $(M(a))_b$ for each $b \in B$.

The probability space is over the coin flips of the algorithm  $\mathcal{M}$.


<!-- page 26 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_123_196_831_925.jpg" alt="This is a schematic diagram of a table. The left column is labeled “A” and contains 0.1, 0.2, ..., 94.5; the right column is labeled “B” and contains A, B, C, D, E, F. The center of the table is marked with the letter “M”. The overall layout is a rectangular grid, with a light gray background and black text, used to display the correspondence between the two columns." />


<!-- page 27 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_123_194_832_924.jpg" alt="The figure illustrates a data mapping process. The left column A contains values such as 0.1 to 94.5, with an arrow pointing to the middle column M, which contains values such as .2 to .15. The right column B contains letters A to F. The bottom is labeled Δ(B). The layout is arranged from left to right, with arrows indicating the direction of data flow, and the text is clear." />


<!-- page 28 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_123_194_831_924.jpg" alt="The figure illustrates a mapping relationship. The left column A contains values such as 0.1, and the right column B contains letters A to F. The middle column M contains values such as .2, .4, and .1. An arrow points from A to M, and the text on the right states that M associates a certain probability simplex to the input value 0.1, with the bottom labeled Δ(B)." />


<!-- page 29 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_123_194_832_924.jpg" alt="The figure illustrates a data mapping process. The left column A contains 0.1 to 94.5, the middle column M contains .1 to .08, and the right column B contains A to F. Arrows indicate that the values in A are mapped to the corresponding values in M, and the final result is labeled as Δ(B). The layout is clear, with text labels indicating the content of each column." />


<!-- page 30 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_123_193_831_925.jpg" alt="The figure illustrates a mapping relationship. The left column A contains values such as 0.1 to 94.5, and the middle column M contains values such as .1 to .08, with an arrow pointing from A to M. The right column B contains letters A to F. Red text in the center states: “a different input can have a different simplex over the output range.” The bottom is labeled Δ(B)." />


<!-- page 31 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_17_192_910_926.jpg" alt="This figure illustrates the randomization process. On the left, the input 0.1 is processed by the randomizer M to generate a set of values, which are then output by the probability Pr=0.4 to the right. The output is divided into six categories: A, B, C, D, E, and F, with the final result labeled as Δ(B). The layout is arranged from left to right, containing arrows and text annotations." />


<!-- page 32 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_17_192_912_926.jpg" alt="This figure illustrates the randomization process. The left column A contains values such as 0.1 and 14.3, with an arrow pointing to the middle column M, which contains values such as .2 and .1. The arrow from M points to the right column B, which contains letters A through F. The label Pr=0.1 is marked in the middle, and the bottom is labeled Δ(B). The overall layout is a horizontal flowchart, with text clearly indicating the data flow and probability." />


<!-- page 33 -->

## RANDOMIZED ALGORITHM

<img src="images/bbox_16_192_910_926.jpg" alt="This figure illustrates the randomization process. The left column A contains values such as 0.1 and 0.2, with 0.2 pointing to the middle column M. M contains values such as .1 and .03, with a probability of 0.03. The right column B contains letters A through F, with B pointing to the final result. The bottom is labeled Δ(B), indicating the randomization result." />


<!-- page 34 -->

## COIN TOSS

<img src="images/bbox_123_192_832_924.jpg" alt="This is a flowchart. The left side, labeled A, contains Y and N; the middle section, labeled M, contains two rows of .??; and the right side, labeled B, contains Y and N. Arrows connect the left and right sides, with the middle section serving as a transition node. The bottom is labeled Δ(B). The overall layout is horizontal, illustrating the process of data flow from A to B." />


<!-- page 35 -->

## COIN TOSS

<img src="images/bbox_122_192_832_924.jpg" alt="This is a flowchart. The left side A contains Y and N, the middle M contains two sets of .75 and .25, and the right side B contains Y and N. Arrows connect the nodes: Y points to the .75 in M, and N points to the .75 in M; the .75 in M points to Y and N in B, and the .25 in M points to Y and N in B. The bottom is labeled Δ(B)." />


<!-- page 36 -->

## A NOTE ON THE OUTPUT

Even though we might use the randomized algorithm to output a single value, this is not very interesting: remember the coin-toss  $\rightarrow$ the response might be simply random.

A much more relevant information is the distribution of the outputs: by looking at the (false) distribution of YES and NO in the coin toss example, we were able to compute the expected proportion of true YES.


<!-- page 37 -->

## A NOTE ON THE OUTPUT

Therefore, from now on, we will interpret  $\mathcal{M}(x)$ as a distribution over the domain of x.

For example, the distribution of weights in our population, the distribution of subjects committing illegal behaviours and so on.


<!-- page 38 -->

## THE DATABASE HISTOGRAM

We will think of databases x as being collections of records from a universe X.

Each database x can be further represented using a vector  $x \in \mathbb{N}^{|\mathcal{X}|}$, in which each entry  $x_i$ represents the number of elements in the database x of type  $i \in \mathcal{X}$.

This definition represents the histogram of the database: given the domain of all possible records, it describes the number of records (in a certain database) falling into each equivalence class.


<!-- page 39 -->

## DISTANCE BETWEEN DATABASES

The  $\ell_{1}$ norm of a database x is denoted  $\|x\|_{1}$ and is defined as:

$$ \left|\left|x\right|\right|_{1}=\sum_{i=1}^{\left|x\right|}\left|x_{1}\right| $$

$\ell_{1}$ distance between two databases x and y is  $\|x - y\|_{1}$

$\|x\|_{1}$ is a measure of the size of a database x (i.e., the number of records it contains)

$\|x-y\|_{1}$ is a measure of how many records differ between x and y.


<!-- page 40 -->

## DIFFERENTIAL PRIVACY

## Differential Privacy:

intuitively guarantees that a randomized algorithm behaves similarly on similar input databases.

“behaves similarly” means that the output of the randomized algorithm will be similar for both databases.


<!-- page 41 -->

## NEIGHBOURING DATASETS

Differential privacy relies on the concept of neighbouring databases: databases that differ for only one record.

If the output is (almost) the same on two neighbouring databases, then any record of the database might be fake: we would observe (almost) the same result!

if  $\|x-y\|_1 = 1$, then x and y are two neighbouring datasets.


<!-- page 42 -->

## DIFFERENTIAL PRIVACY

A randomized algorithm $\mathcal{M}$ with domain $\mathbb{N}^{|\mathcal{X}|}$ is $(\varepsilon, \bar{\delta})$-differentially private if for all $S \subseteq \text{Range}(\mathcal{M})$ and for all $x, y \in \mathbb{N}^{|\mathcal{X}|}$ such that $||x - y||_1 \leq 1$:

$$ \Pr[\mathcal{M}(x)\in S]\leq e^{\varepsilon}\Pr[\mathcal{M}(y)\in S]+\delta $$

where the probability space is over the coin flips of the mechanism $\mathcal{M}$. If $\bar{\delta}=0$, we say that $M$ is $\varepsilon$-differentially private.


<!-- page 43 -->

## DIFFERENTIAL PRIVACY

considering $\bar{\mathbf{o}} = 0$, since $y$ and $x$ are neighbouring datasets (which is a symmetrical property) it is also true that:

$$ \Pr[\mathcal{M}(y)\in S]\leq e^{\varepsilon}\Pr[\mathcal{M}(x)\in S] $$

therefore:

$$ e^{-\varepsilon}\Pr[\mathcal{M}(y)\in S]\leq\Pr[\mathcal{M}(x)\in S] $$

thus:

$$ e^{-\varepsilon}\Pr[\mathcal{M}(y)\in S]\leq\Pr[\mathcal{M}(x)\in S]\leq e^{\varepsilon}\Pr[\mathcal{M}(y)\in S] $$


<!-- page 44 -->

## WHICH EPSILON SHOULD WE USE?

if  $\varepsilon$ is small...

$$ e^{0}\operatorname*{P r}[\mathcal{M}(y)\in S]\leq\operatorname*{P r}[\mathcal{M}(x)\in S]\leq e^{0}\operatorname*{P r}[\mathcal{M}(y)\in S] $$

$$ \Pr[\mathcal{M}(y)\in S]\leq\Pr[\mathcal{M}(x)\in S]\leq\Pr[\mathcal{M}(y)\in S] $$

$$ \Pr[\mathcal{M}(y)\in S]=\Pr[\mathcal{M}(x)\in S] $$

2 different (neighbouring) datasets, same output.

maximum privacy: the output does not depend on the data (i.e., its noise).


<!-- page 45 -->

## WHICH EPSILON SHOULD WE USE?

Giving a precise rule-of-thumb is very hard.

If  $\varepsilon$ is very small, even if a mechanism is not  $(\varepsilon, 0)$-differentially private, then it might be  $(2\varepsilon, 0)$-differentially private, which might be still acceptable.


<!-- page 46 -->

## WHICH EPSILON SHOULD WE USE?

Failure to be (15, 0)-differentially private tells that there exist neighboring databases and an output o for which the ratio of probabilities of observing o conditioned on the database being, respectively, x or y, is large.

If o is particularly unlikely, then x and y looks particularly “strange” or artifact, and thus the malevolent data scientist wouldn’t gain much information.

as a rule of thumb, most common values for  $\varepsilon$ are between 0.1 and 5 - typical values are  $\ln(2)$ and  $\ln(3)$.


<!-- page 47 -->

## WHICH EPSILON SHOULD WE USE?

<table border=1><tr><td>Authors</td><td>Value(s) of  $\varepsilon$</td><td>Application</td></tr><tr><td>McSherry-Mahajan [32]</td><td>0.1—10</td><td>Network Trace Analysis</td></tr><tr><td>Chaudhuri-Monteleoni [7]</td><td>0.1</td><td>Logistic Regression</td></tr><tr><td>Machanavajjhala et al. [30]</td><td>&lt; 7</td><td>Census Data</td></tr><tr><td>Korolova et al. [24]</td><td>ln2, ln5, ln10</td><td>Click Counts</td></tr><tr><td>Bhaskar et al. [5]</td><td>1.4</td><td>Frequent Items</td></tr><tr><td>Machanavajjhala et al. [31]</td><td>0.5—3</td><td>Recommendation System</td></tr><tr><td>Bonomi et al. [6]</td><td>0.01—10</td><td>Record Linkage</td></tr><tr><td>Li et al. [28]</td><td>0.1—1</td><td>Frequent Items</td></tr><tr><td>Ny-Pappas [38]</td><td>ln3</td><td>Kalman Filtering</td></tr><tr><td>Chaudhuri et al. [8]</td><td>0.1—2</td><td>Principal Component Analysis</td></tr><tr><td>Narayan-Haeberlen. [34]</td><td>0.69</td><td>Distributed Database Joins</td></tr><tr><td>Chen et al. [10]</td><td>1.0 – 5.0</td><td>Queries over Distributed Clients</td></tr><tr><td>Acs-Castelluccia [2]</td><td>1</td><td>Smart Electric Meters</td></tr><tr><td>Uhler et al. [41]</td><td>0.1—0.4</td><td>Genome Data</td></tr><tr><td>Xiao et al. [43]</td><td>0.05—1</td><td>Histograms</td></tr><tr><td>Li-Miklau. [27]</td><td>0.1—2.5</td><td>Linear Queries</td></tr><tr><td>Chen et al. [9]</td><td>0.5—1.5</td><td>Trajectory Data</td></tr><tr><td>Cormode et al. [12]</td><td>0.1—1</td><td>Location Data</td></tr><tr><td>Chaudhuri et al. [40]</td><td>0.01—0.5</td><td>Empirical Risk Minimization</td></tr></table>

Table 1: Values of  $\varepsilon$ in the literature

J. Hsu et al., "Differential Privacy: An Economic Method for Choosing Epsilon," 2014 IEEE 27th Computer Security Foundations Symposium, 2014, pp. 398-410, doi: 10.1109/CSF.2014.35.


<!-- page 48 -->

## DIFFERENTIAL PRIVACY:  $\delta$

a typical value for  $\bar{\delta}$ is far less than the inverse of any polynomial in the size of the database.

If  $\bar{\delta} = 1/||x||_1$ permits to “preserve the privacy”, by publishing a small number of records (“Just a few” policy).


<!-- page 49 -->

## DIFFERENTIAL PRIVACY:  $\delta$

$\delta = 0$ grants that, for every run of the randomized algorithm  $\mathcal{M}$, the output is (almost) equally likely to be observed on every neighboring database, simultaneously.

(ε,  $\bar{\delta}$)-differential privacy says that for every pair of neighboring databases x, y, it is unlikely (with probability  $\bar{\delta}$) that, the observed value M(x) will be much more or much less likely to be generated when the database is x than when the database is y.


<!-- page 50 -->

## EPSILON AND DELTA

epsilon: privacy budget. the lower, the higher the noise and thus data protection.

delta: probability of failing in protecting the privacy. It represents a form of relaxation of DP, where we assume that, for some rare inputs, the privacy bounds are larger than expected.


<!-- page 51 -->

## PRIVACY LOSS

A very important measure is the privacy loss incurred if the output o is observed and which can be computed as:

$$ \mathcal{L}_{\mathcal{M}(x)||\mathcal{M}(y)}^{o}=\ln\left(\frac{\Pr[\mathcal{M}(x)=o]}{\Pr[\mathcal{M}(y)=o]}\right) $$

An important lemma tells us that:

$$ \left|\mathcal{L}_{\mathcal{M}(x)||\mathcal{M}(y)}^{o}\right|\leq\varepsilon,\mathrm{w i t h~p r o b a b i l i t y}1-\delta $$


<!-- page 52 -->

## PRIVACY LOSS

We can now revisit our coin toss algorithm. Assume to have two databases, with a single record, b and b' which are neighbouring.

if they are neighbouring, they must differ for a single record: then assuming, without loss of generality, b=Y, b'=N.

we know that  $\Pr[M(b) = Y] = \frac{3}{4}$, while  $P[M(b') = Y] = \frac{1}{4}$. thus:

$$ \varepsilon\geq\left|\log\left(\frac{3/4}{1/4}\right)\right|=\log3 $$

The coin toss algorithm is (log 3, 0)-differentially private.


<!-- page 53 -->

## COMPOSITION

The combination of two differentially private mechanisms is again a differentially private mechanism.

You can use the simplest differential privacy mechanisms as building blocks for more complex DP solutions.


<!-- page 54 -->

## COMPOSITION

<img src="images/bbox_148_245_377_679.jpg" alt="The image shows a simple circuit diagram. On the left is a battery with an orange top and a gray body, labeled “Battery”. On the right, two yellow light bulbs are connected in parallel, with their terminals connected to the circuit. The overall layout is clear, with the battery on the left and the bulbs on the right, forming a closed loop. The background is pure white, with no other elements." />

If (independent) mechanisms are placed in series, the privacy guarantee (upper bound) is the sum of the privacy guarantees.

<img src="images/bbox_556_238_918_671.jpg" alt="The image shows a parallel circuit diagram. On the left is a battery with an orange top and a gray body, labeled “Battery”. On the right, two yellow light bulbs are connected in parallel, with their terminals connected to the circuit. The overall layout is clear, with the battery on the left and the bulbs on the right, forming a closed loop. The background is pure white, with no other elements." />

If (independent) mechanisms are placed in parallel - on different partitions of the dataset - the privacy guarantee is the maximum of  $\varepsilon$.


<!-- page 55 -->

## GROUP DIFFERENTIAL PRIVACY

What if in our data we have a family of k, rather than a person? Would the privacy of the family remain protected?


<!-- page 56 -->

## GROUP DIFFERENTIAL PRIVACY: THEOREM

Any $(\varepsilon, 0)$-differentially private mechanism $\mathcal{M}$ is $(k\varepsilon, 0)$-differentially private for groups of size $k$. That is, for all $||x - y||_1 \leq k$ and all $S \subseteq \text{Range}(\mathcal{M})$:

$$ \Pr[\mathcal{M}(x)\in S]\leq e^{k\varepsilon}\Pr[\mathcal{M}(y)\in S] $$

where the probability space is over the coin flips of the mechanism $\mathcal{M}$.


<!-- page 57 -->

## POST-PROCESSING

Let $\mathcal{M}: \mathbb{N}^{|\mathcal{H}|} \to R$ be a randomized algorithm that is $(\varepsilon, \bar{\delta})$-differentially private. Let $f: R \to R'$ be an arbitrary randomized mapping. Then $f \circ \mathcal{M}: \mathbb{N}^{|\mathcal{H}|} \to R'$ is $(\varepsilon, \bar{\delta})$-differentially private.

A function applied to a randomized algorithm, returns again a randomized algorithm: it is impossible to go back to the original input.


<!-- page 58 -->

## POST-PROCESSING

Differential Privacy is immune to post processing: it is not possible to compute a function of the output of a differentially private algorithm to make it less differentially private.


<!-- page 59 -->

## PROPERTIES OF THE DIFFERENTIAL PRIVACY

Automatic neutralization of linkage attacks (aggregate statistics)

Quantification of the privacy loss

Composition

Group Privacy

Closure Under Post-Processing
