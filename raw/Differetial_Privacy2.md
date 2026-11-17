# DIFFERENTIAL PRIVACY: PART II

PRIVACY PRESERVING INFORMATION ACCESS

Master Degree in Cybersecurity, Master Degree in Computer Engineering, and Master Degree In International Cybersecurity and Cyberintelligence

A.Y. 2026/2027

GUGLIELMO FAGGIOLI

Intelligent Interactive Information Access (IIIA) Hub Department of Information Engineering University of Padua![](images/bbox_867_780_966_955.jpg)

## RANDOMIZED RESPONSE

We already know a differentially private algorithm: the randomized response or coin toss.

Flip a coin:

if “tail”, respond truthfully

if “head”, flip coin again:

if “tail”, respond “YES”

if “head”, respond “NO”

## RANDOMIZED RESPONSE

We are also able to compute its privacy loss:

$$ \frac{\Pr_{x}(Y)}{\Pr_{y}(Y)}=\frac{\Pr[R=Y\mid T=Y]}{\Pr[R=Y\mid T=N]}=\frac{3/4}{1/4}=3 $$

And thus we say that the randomized response is a (ln 3 - 0) differentially private algorithm.

We can do better than this...

## $\ell_{1}$-SENSITIVITY

remember that we are interested in queries that return distributions (functions).

We introduce the notion of $\ell_{1}$-sensitivity of a function.

## $\ell_{1}$-SENSITIVITY

Given a function $f: \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$ its $\ell_1$-sensitivity is:

$$ \Delta f=\max_{x,y\in\mathbb{N}^{|\mathcal{X}|}||x-y||{1}=1}||f(x)-f(y)||{1} $$

The $\ell_{1}$-sensitivity describes the maximum change in the function induced by a single instance.

Intuitively, it also gives the magnitude of the “perturbation” required to hide the participation of a single individual.

## $\ell_{1}$-SENSITIVITY

Given a function $f: \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$ its $\ell_1$-sensitivity is:

$$ \Delta f=\max_{x,y\in\mathbb{N}^{|\mathcal{X}|}}||f(x)-f(y)||_{1} $$

The $\ell_{1}$-sensitivity describes the maximum change in the function induced by a single instance.

We operate under the assumptions that:

- neighbouring datasets differ by one added/removed record
- each individual has a unique value for the sensitive attribute

participation of a single individual.

## THE LAPLACE MECHANISM

As its name suggests, the Laplace mechanism relies on the Laplace distribution and requires to add independent Laplace noise to each coordinate of the query output.

## THE LAPLACE MECHANISM

Given a function $f: \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$ the Laplace mechanism is defined as:

$$ \mathcal{M}{L}(x,f,\varepsilon)=f(x)+(Y{1},\ldots,Y_{k}) $$

Where $Y_{1}, ..., Y_{k}$ are drawn from Lap(0, $\Delta f/\varepsilon$), with Lap(x|μ, b) defined as:

$$ Lap(x|\mu,b)=\frac{1}{2b}\cdot\exp\left(-\frac{|x-\mu|}{b}\right) $$![This is a 2D function curve plot. The horizontal axis ranges from -10 to 10, and the vertical axis from 0 to 0.5. The plot contains four curves: a red line (μ=0, b=1) with a sharp peak at 0; a black line (μ=0, b=2) with a slightly lower peak; a blue line (μ=0, b=4) with a lower peak; and a green line (μ=-5, b=4) with a broad peak. A legend is located in the upper right corner.](images/bbox_672_592_865_860.jpg)

## THE LAPLACE MECHANISM

The Laplace mechanism is $(\varepsilon, 0)$-differentially private:

$$ \begin{aligned}\frac{p_{x}(z)}{p_{y}(z)}&=\prod_{i=1}^{k}\frac{\frac{\varepsilon}{2\Delta f}\exp\left(-\frac{\varepsilon|f(x){i}-z{i}|}{\Delta f}\right)}{\frac{\varepsilon}{2\Delta f}\exp\left(-\frac{\varepsilon|f(y){i}-z{i}|}{\Delta f}\right)}\&=\prod_{i=1}^{k}\exp\left(\frac{\varepsilon\big(|f(y){i}-z{i}|-|f(x){i}-z{i}|\big)}{\Delta f}\right)\&\leq\prod_{i=1}^{k}\exp\left(\frac{\varepsilon|f(x){i}-f(y){i}|}{\Delta f}\right)\&=\exp\left(\frac{\varepsilon|f(x)-f(y)|_{1}}{\Delta f}\right)\&\leq e^{\varepsilon}\end{aligned} $$

(reverse triangle inequality)

(sensitivity: $|f(x) - f(y)|_1 \leq \Delta f$ for $x \sim y$).

## THE LAPLACE MECHANISM: ACCURACY

Let $f: \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$ and $y = \mathcal{M}_L(x, f, \varepsilon)$. Then $\forall \delta \in (0, 1]$:

$$ \Pr\left[\left|\left|f(x)-y\right|\right|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

This allows us to determine how “far” is our perturbation from the real value.

## COUNTING QUERIES

Counting queries are the most common example of “statistical” queries on a dataset.

“How many instances have this characteristic?”, “What is the proportion of instances satisfying a certain property?”

What is the sensitivity of a counting query?

## COUNTING QUERIES

Counting queries are the most common example of “statistical” queries on a dataset.

“How many instances have this characteristic?”, “What is the proportion of instances satisfying a certain property?”

What is the sensitivity of a counting query?![The image displays a simple black line drawing on a pure white background. The main subject is a straight line segment with an arrowhead at its right end, pointing diagonally upward to the right. The line is slightly thicker than the arrowhead, and the overall shape resembles a simplified arrow or indicator symbol. There are no other objects, scenes, or text in the image.](images/bbox_179_706_221_795.jpg)

$$ \Delta f=\max_{x,y\in\mathbb{N}^{|x|}\mid|x-y|{1}=1}||f(x)-f(y)||{1} $$

How much changing a single instance changes the count?

## COUNTING QUERIES

Counting queries are the most common example of “statistical” queries on a dataset.

“How many instances have this characteristic?”, “What is the proportion of instances satisfying a certain property?”

What is the sensitivity of a counting query? 1

## COUNTING QUERIES

Counting queries are the most common example of “statistical” queries on a dataset.

“How many instances have this characteristic?”, “What is the proportion of instances satisfying a certain property?”

What is the sensitivity of a counting query? 1

How much should we perturb the result to make a counting query $(\varepsilon, 0)$-differentially private?

## COUNTING QUERIES

Counting queries are the most common example of “statistical” queries on a dataset.

“How many instances have this characteristic?”, “What is the proportion of instances satisfying a certain property?”

What is the sensitivity of a counting query? 1

How much should we perturb the result to make a counting query $(\varepsilon, 0)$-differentially private? Lap(1/ $\varepsilon$)

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1, 0)-differentially private Laplace mechanism?

## HISTOGRAM QUERIES

Histogram queries are (disjoint) count queries: we have n distinct ranges and we want to compute how many instances fall on each range.

How can we achieve $\left(\varepsilon, 0\right)$-differentially private queries?

determine the sensitivity

determine the perturbation distribution

## HISTOGRAM QUERIES

Like count queries, also for histogram queries switching between neighbouring datasets, under our definition of neighbouring datasets, changes only the count for a single bin of 1: the sensitivity is 1.

Add to each column noise independently drawn from a Lap(1/ε)

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names (fixed in advance) in the 2010 american census for a (1, 0)-differentially private Laplace mechanism?

$$ \Pr\left[\left|\left|f(x)-y\right|\right|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1, 0)-differentially private Laplace mechanism?

$$ \Pr\left[|f(x)-y|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1, 0)-differentially private Laplace mechanism?

$$ \Pr\left[\left|f(x)-y\right|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1,0)-differentially private Laplace mechanism?

$$ \Pr\left[|f(x)-y|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

## FIRST NAMES

With probability 95% how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1, 0)-differentially private Laplace mechanism?

$$ \Pr\left[\left|f(x)-y\right|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1,0)-differentially private Laplace mechanism?

$$ \Pr\left[\left|\left|f(x)-y\right|\right|_{\infty}\geq\ln\left(\frac{k}{\delta}\right)\cdot\left(\frac{\Delta f}{\varepsilon}\right)\right]\leq\delta $$

$$ \Pr\left[\left|\left|f(x)-y\right|\right|_{\infty}\geq\ln\left(\frac{10000}{0.05}\right)\cdot\left(\frac{1}{1}\right)\right]\leq0.05 $$

## FIRST NAMES

With probability 95%, how accurate is the (perturbed) histogram of the top 10000 names in the 2010 american census for a (1, 0)-differentially private Laplace mechanism?

With probability at least 95%, every one of the 10,000 released counts differs from its true value by at most approximately 12.21 people (with over 300,000,000 people participating to the census).

## COUNT AND HISTOGRAM QUERIES

What if we change our definition of neighbouring datasets?

Besides adding or removing a record, we can also change it.

The sensitivity of a count query is:

The sensitivity of an histogram query is:

## COUNT AND HISTOGRAM QUERIES

What if we change our definition of neighbouring datasets?

Besides adding or removing a record, we can also change it.

The sensitivity of a count query is: 1

The sensitivity of an histogram query is: 2

## REPORT NOISY MAX

Consider m counting queries that may overlap, rather than being restricted to disjoint histogram bins (i.e., each individual can contribute to many queries (e.g., disease = {asthma, diabetes, cancer, ...}) – we also go back to add/remove adjacency.

The report noisy max that returns the index of the largest (Lap(1/ε) perturbed) count, is (ε, 0) - differentially private.

Notice that we do not release the counts: they would be much more informative than $\varepsilon$: single user can appear in all the m count queries, thus $\Delta f = m$.

## GAUSSIAN NOISE

We can define a perturbation process based on a Gaussian noise which allows to have a differentially private mechanism.

$$ \mathcal{M}{\mathcal{G}}(x,\varepsilon,\delta)\sim\mathcal{N}\left(x,\frac{2\ln\left(1.25/\delta\right)\cdot\left(\Delta{2}f\right)^{2}}{\varepsilon^{2}}\right) $$

Where $\Delta_{2}$ is the $\ell_{2}$-sensitivity of the function.

If $0 < \bar{\delta} < 1$ and $0 < \varepsilon < 1$, then this mechanism is DP.

Laplace mechanism can be used with $\bar{\delta}=0$ ( $\varepsilon$-DP), Gaussian cannot (there is a risk of releasing more information than what we would like...).

## GAUSSIAN NOISE

Why using it?

If the function f for which we are releasing the values returns a vector of size k and not a single value, its $\ell_{2}$-sensitivity might be smaller than its $\ell_{1}$-sensitivity by a factor of $\sqrt{\text{sqrt}(k)}$.

Gaussian noise can improve accuracy for vector queries when their $\ell_{2}$-sensitivity is substantially smaller than their $\ell_{1}$-sensitivity. The benefit depends on the query and privacy parameters.

## THE EXPONENTIAL MECHANISM

“How many people have blue eyes?” → Perturbing the answer with the Laplace mechanism does not “destroy” our answer: we introduce an error, but we will be “close” to the correct answer.

## THE EXPONENTIAL MECHANISM

“What is the best value for a certain threshold?” → a small perturbation that brings the best value over the threshold, might make useless the answer!

## THE EXPONENTIAL MECHANISM: THE AUCTION

We are going to hold an auction to sell a certain product.

Each (possible) buyer places their secret bid: they don't want the final price to be influenced by their bids.

- If we set the price of the product below or equal to the bid, then the buyer will buy the item at the price we have chosen.
- If we set the price above the bid, then the buyer won't buy the product.
- A trusted curator receives the bids and releases a differentially private price. This limits how much any one bid affects the distribution of the published price.

## ON THE EXPONENTIAL MECHANISM![This is a minimalist-style user avatar icon. The main subject is a human figure outlined in red lines, consisting of a circular head and a semicircular body, with a light pink fill. The background is pure white, with no other scenes or text. The layout is centered, and the design is simple and clear, commonly used to represent a user or personal account.](images/bbox_99_261_146_395.jpg)

4.10€![This is a minimalist-style user avatar icon. The main subject is a human figure outlined in blue lines, consisting of a circular head and a semicircular body, with a light blue fill. The background is pure white, with no other scenes or text. The layout is centered, and the design is simple and clear, commonly used to represent a user or personal account.](images/bbox_100_427_145_561.jpg)

1.00€![This is a minimalist-style user avatar icon. The main subject is a human figure outlined in green lines, consisting of a circular head and a semicircular body, with a light green fill. The background is pure white, with no other scenes or text. The layout is centered, and the design is simple and clear, commonly used to represent a user or personal account.](images/bbox_100_600_145_733.jpg)

1.00€![This is a minimalist-style user avatar icon. The main subject is a human figure outlined in yellow lines, consisting of a circular head and a semicircular body, with a light yellow fill. The background is pure white, with no other scenes or text. The layout is centered, and the design is simple and clear, commonly used to represent a user or personal account.](images/bbox_100_766_145_898.jpg)

1.00€![On a white background, a red apple with a green leaf is positioned at the bottom left. A tilted rectangular label is located at the top right, containing the black text “??€”. The overall layout is simple, with the apple and label arranged diagonally, and no other background elements.](images/bbox_649_260_832_604.jpg)

## ON THE EXPONENTIAL MECHANISM![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a red line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_99_261_145_395.jpg)

4.10€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a blue line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_100_427_144_561.jpg)

1.00€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a green line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_100_600_144_733.jpg)

1.00€![The image shows a red apple with a green leaf, connected to a tilted rectangular price tag above it. The tag is printed with the black text “1.00€”. The overall background is pure white, with a simple layout that highlights the price information, commonly used to represent a product or item price.](images/bbox_650_261_830_603.jpg)

If the price is 1: everyone buys.

Total gain = pricebuyers = 14 = 4.00

## ON THE EXPONENTIAL MECHANISM![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick red line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_99_260_145_395.jpg)

4.10€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick blue line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_100_427_144_561.jpg)

1.00€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick green line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_100_600_144_733.jpg)

1.00€![The image shows a red apple with a green leaf, connected to a tilted rectangular price tag above it. The tag is printed with the black text “2.00€”. The background is pure white, and the overall layout is simple, highlighting the price information.](images/bbox_650_260_830_603.jpg)

If the price is 2: only red buys.

Total gain = pricebuyers = 21 = 2.00

## ON THE EXPONENTIAL MECHANISM![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick red line. The background is pure white, with no other scenes or text. The layout is centered, and the colors are vivid, commonly used to represent a user or personal account.](images/bbox_99_260_145_395.jpg)

4.10€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick blue line. The background is pure white, with no other scenes or text. The layout is centered, and the colors are vivid, commonly used to represent a user or personal account.](images/bbox_100_427_144_561.jpg)

1.00€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick green line. The background is pure white, with no other scenes or text. The layout is centered, and the colors are vivid, commonly used to represent a user or personal account.](images/bbox_100_600_144_733.jpg)

1.00€![The image shows a red apple with a green leaf, connected to a tilted rectangular price tag. The tag is printed with the black text “4.10€”. The overall layout is simple, with a pure white background, highlighting the price information of the apple.](images/bbox_650_260_830_603.jpg)

If the price is 4.10: only red buys.

Total gain = pricebuyers = 4.101 = 4.10

## ON THE EXPONENTIAL MECHANISM![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick red line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_98_260_146_396.jpg)

4.10€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick blue line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_98_426_145_562.jpg)

1.00€![This is a minimalist-style user avatar icon. The main subject is a human figure composed of a circular head and a semicircular body, outlined with a thick green line. The background is pure white, with no other scene elements or text. The layout is centered, and the colors are simple and clear, commonly used to represent a user or personal account.](images/bbox_98_598_145_734.jpg)

1.00€![The image shows a red apple with a green leaf, connected to a tilted rectangular price tag above it. The tag is printed with the black text “4.20€”. The overall layout is simple, with a pure white background, highlighting the price information.](images/bbox_649_260_831_604.jpg)

If the price is 4.20: nobody buys.

Total gain = pricebuyers = 4.200 = 0

## ON THE EXPONENTIAL MECHANISM

Challenges:

Red might not be happy with the selling price decided based on their bid. They will participate only if the bids remain private.![The figure illustrates a bidding process. On the left, four user icons (red, blue, green, yellow) each send an envelope to a central black box labeled “DP BLACK BOX”. The box then outputs an arrow pointing to the text “Set the price to x€” on the right. The layout proceeds from left to right, with a white background and a clean, clear design.](images/bbox_112_525_902_902.jpg)

## ON THE EXPONENTIAL MECHANISM

Utility function:![This is a utility function chart. The horizontal axis represents the selling price, and the vertical axis represents revenue. The red line shows the utility function, with the high-satisfaction point marked at (4.10, 4.10). The background is divided into four color blocks, labeled unsatisfied, mostly satisfied, satisfied, and severely unsatisfied, respectively, illustrating the distribution of satisfaction levels at different prices.](images/bbox_66_239_842_927.jpg)

## THE EXPONENTIAL MECHANISM: THE AUCTION

SECRET (PRIVATE) BIDS![The image shows a simple black-and-white line icon. The main subject is a rectangular envelope with a V-shaped flap at the top. Centered below the envelope is the dark gray text “1€”. The overall layout is clean, with a pure white background and no other elements.](images/bbox_104_334_244_487.jpg)![The image shows a simple black-and-white line icon. The main subject is a rectangular envelope with a V-shaped flap at the top. Centered below the envelope is the dark gray text “1€”. The overall layout is clean, with a pure white background and no other elements.](images/bbox_321_334_461_487.jpg)![The image shows a simple black-and-white line icon. The main subject is a rectangular envelope with a V-shaped flap at the top. Centered below the envelope is the dark gray text “1€”. The overall layout is clean, with a pure white background and no other elements.](images/bbox_539_334_678_487.jpg)![The image shows a simple black-and-white line icon. The main subject is a rectangular envelope with a V-shaped flap at the top. Centered below the envelope is the dark gray text “4.1€”. The overall layout is clean, with a pure white background and no other elements.](images/bbox_756_334_895_487.jpg)

all buy

revenue: 1.0*4![This is a horizontal bar chart. The bar is divided into segments of different colors, with the gray area in the middle labeled “PRICE”. The segments on the left are yellow, green, and orange, with values 0.9, 1.0, and 1.1 respectively, and arrows pointing to the text “all buy” and “only 4 buys”. The segments on the right are yellow, green, and red, with values 3.9, 4.0, 4.1, and 4.2 respectively, and arrows pointing to the text “nobody buys” and “only 4 buys”.](images/bbox_29_643_968_817.jpg)

all buy

revenue: 0.9*4

only 4 buys

revenue: 1.1

nobody buys

revenue: 0

only 4 buys

revenue: 4.1 (max)

## THE EXPONENTIAL MECHANISM: THE AUCTION

Perturbing from 1.0 to 1.1 or from 4.1 to 4.2 causes huge drops in terms of optimality of the price!![This figure illustrates the price distribution. The horizontal axis is divided into two segments: the left segment, labeled “PRICE”, contains 0.9, 1.0, 1.1, and 1.2, with annotations for all-buy revenue of 1.0*4 and only 4 buys of 1.1; the right segment, labeled “PRICE”, contains 3.9, 4.0, 4.1, 4.2, and an ellipsis, with annotations for no-buy revenue of 0 and only 4 buys of 4.1 (maximum). The layout is clear, with arrows indicating the corresponding values.](images/bbox_26_550_969_916.jpg)

## THE EXPONENTIAL MECHANISM: UTILITY

the revenue is the utility that we experience upon a realization of a certain future outcome ("if I will set the price to 1, my revenue will be 4").

we can then define a function $u$ that, given a database, maps the set of possible future outcomes $\mathcal{R}$ (values from 0.1 to 4.2, with step 0.1) to a real value which represents the utility of the outcome:

$$ u:\mathbb{N}^{|\mathcal{X}|},\mathcal{R}\to\mathbb{R} $$

## UTILITY

|   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|
|0.9|1.0|1.1|1.2|...|PRICE|...|3.9|4.0|4.1|4.2|
|3.6|4.0|1.1|1.2|...|UTILITY|...|3.9|4.0|4.1|0|

## THE EXPONENTIAL MECHANISM: SENSITIVITY

The sensitivity of a utility function u with respect to the future realizations is:

$$ \Delta u=\max_{r\in\mathcal{R}}\max_{x,y;||x-y||_{1}\leq1}|u(x,r)-u(y,r)| $$

It describes how much, with respect to two neighbouring datasets, the utility can change with respect to the same future outcome.

## THE EXPONENTIAL MECHANISM: SENSITIVITY

|   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|
|0.9|1.0|1.1|1.2|...|PRICE|...|3.9|4.0|4.1|4.2|
|3.6|4.0|1.1|1.2|...|UTILITY|...|3.9|4.0|4.1|0|
|2.7|3.0|1.1|1.2|...|UTILITY (y $_{1}$)|...|3.9|4.0|4.1|0|
|2.7|3.0|1.1|1.2|...|UTILITY (y $_{2}$)|...|3.9|4.0|4.1|0|
|2.7|3.0|1.1|1.2|...|UTILITY(y $_{3}$)|...|3.9|4.0|4.1|0|
|2.7|3.0|0|0|...|UTILITY(y $_{4}$)|0|0|0|0|0|
|4.5|5|2.2|2.4|...|UTILITY(y $_{5}$)|...|7.8|8.0|8.2|4.2|
|0.9|1.0|1.1|1.2|...|SENSITIVITY|...|3.9|4.0|4.1|4.2|

## THE EXPONENTIAL MECHANISM: SENSITIVITY

|   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|
|0.9|1.0|1.1|1.2|...|PRICE|...|3.9|4.0|4.1|4.2|

At a fixed selling price r, adding or removing one buyer changes revenue, by either 0 or r.

Hence, the sensitivity is the maximum selling price r: for a global sensitivity, we need to cap the maximum selling price.

|   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|
|2.7|3.0|0|0|...|UTILITY(y_s)|0|0|0|0|0|
|4.5|5|2.2|2.4|...|UTILITY(y_s)|...|7.8|8.0|8.2|4.2|

|   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|
|0.9|1.0|1.1|1.2|...|SENSITIVITY|...|3.9|4.0|4.1|4.2|

## THE EXPONENTIAL MECHANISM: SENSITIVITY

|   |   |   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|
|0.9|1.0|1.1|1.2|...|PRICE|...|3.9|4.0|4.1|4.2|
|3.6|4.0|1.1|1.2|...|UTILITY|...|3.9|4.0|4.1|0|
|0.9|1.0|1.1|1.2|...|SENSITIVITY|...|3.9|4.0|4.1|4.2|

## THE EXPONENTIAL MECHANISM

The exponential mechanism $\mathcal{M}_E(x, u, \mathcal{R})$ selects and outputs an element $r \in \mathcal{R}$ with probability proportional to

$$ \exp(\varepsilon^{}u(x,r)/(2^{}\Delta u)) $$

The exponential mechanism preserves $(\varepsilon, 0)$-differential privacy.

With the exponential mechanism we reduce the chances of outputting values in r where the utility is 0.

## THE EXPONENTIAL MECHANISM: SENSITIVITY![The figure displays four horizontal bar charts, labeled PRICE, UTILITY, SENSITIVITY, and PROBABILITY* (ε=0.1). Each bar consists of colored segments with numerical values, with the first segment in yellow, the second in green, and the last in red. The layout is arranged vertically, with the text centered, and the background is white.](images/bbox_76_246_923_662.jpg)

$^{*}$ This is not the probability, but a value which is proportional to it.

## ACCURACY

if u is the utility achieved by the exponential mechanism, $opt_{u}$ the optimal utility, and $R_{opt}$ the set of optimal values (those that achieve the best utility), the following inequality bounds our errors:

$$ \Pr\left[u(\mathcal{M}{E}(x,u,\mathcal{R}))\leq opt{u}(x)-\frac{2\Delta u}{\varepsilon}\left(\ln\left(\frac{|\mathcal{R}|}{|\mathcal{R}_{opt}|}\right)+t\right)\right]\leq e^{-t} $$

## COMBINATION

Remember what we said last time about group privacy?

“Any $(\varepsilon, 0)$-differentially private mechanism $\mathcal{M}$ is $(k\varepsilon, 0)$-differentially private for groups of size $k$”

And what about the report noisy max algorithm?

“We cannot release all the noisy counts to avoid increasing the sensitivity (and thus the privacy loss).”

## COMBINATION![The image displays a horizontal arrow diagram on a white background. The text “M₁ ε₁-differentially private” is positioned at the top, with a double-headed arrow below it. The text “M₂ ε₂-differentially private” is in the middle, with a double-headed arrow below it. The text “...” is centered, and the text “Mₙ εₙ-differentially private” is at the bottom, with a double-headed arrow below it. The overall layout is concise, presenting a sequence of privacy protection measures.](images/bbox_203_252_591_679.jpg)![This is a black-and-white line drawing depicting a cylindrical database icon. The main body is a three-dimensional cylinder, with three horizontal lines on the surface representing data layers. The image is centered on a pure white background, with no other scene elements or visible text, featuring a minimalist style.](images/bbox_608_341_768_656.jpg)

The whole process is (up to) $(\varepsilon_1 + \varepsilon_2 + \ldots + \varepsilon_n)$-differentially private: each time we query our database we increase the privacy loss.

## SPARSE VECTOR: ABOVE THRESHOLD

Imagine a movie recommender choosing a film for Alice. It has a fixed group of users with similar tastes and a candidate list of films Alice hasn't watched.

For each candidate, it asks:

“Have roughly 50 users with tastes similar to Alice's liked this movie?”

## SPARSE VECTOR: ABOVE THRESHOLD![The figure displays a mathematical formula structure. The top is labeled “M₁ ε₁-differentially private”, the middle is labeled “M₂ ε₂-differentially private”, and the bottom is labeled “Mₙ εₙ-differentially private”. Each line is connected by a horizontal arrow, with the green letter “T” marked at the bottom of the arrows, and a red “⊥” marked at the bottom of the middle line. The layout is arranged vertically, with a white background.](images/bbox_202_252_593_719.jpg)![This is a black-and-white line drawing depicting a cylindrical database icon. The main body is a three-dimensional cylinder, with three horizontal lines on the surface representing data layers. The image is centered against a pure white background, with no other scene elements or visible text, in a minimalist style.](images/bbox_608_340_768_656.jpg)

Let say that, instead of answering with the “correct” (perturbed) answer, we simply say if it is above a certain threshold.

## SPARSE VECTOR: ABOVE THRESHOLD![The figure displays a horizontal line structure. The top is labeled “M₁ ε₁-differentially private”, the middle is labeled “M₂ ε₂-differentially private”, and the bottom is labeled “Mₙ εₙ-differentially private”. Red “⊥” symbols are marked on the left side of the lines, and the bottom is marked with a green “T” symbol. The overall layout is a vertical arrangement, with text in black and gray and symbols in red and green.](images/bbox_202_252_593_718.jpg)![This is a black-and-white line drawing depicting a cylindrical database icon. The main body is a three-dimensional cylinder, with two horizontal lines on the side representing the database layer structure. The image is centered against a pure white background, with no other scene elements or visible text, in a minimalist style.](images/bbox_608_340_768_656.jpg)

More precisely, we continue answering as long as our results are below the threshold: once we get the first “above threshold” result, we stop.

## SPARSE VECTOR: ABOVE THRESHOLD

Above-Threshold(D, Q, T, $\varepsilon$):

Let $T_{p} = T + \text{Lap}(2/\varepsilon)$;

for $q_{i}$ in Q:

Let $v_{i} = \text{Lap}(4/\varepsilon)$;

if $q_i(D) + v_i \geq T_p$:

output T

break

else:

output $\perp$

Where D is a dataset while Q is a stream of queries (whose sensitivity is 1, as the count query).

## WHO ARE WE PROTECTING?

We are protecting the other users whose likes support Alice's recommendation: the people in her similarity group.

For example, suppose Bob belongs to that group. The privacy guarantee compares:

The recommendation process including Bob's likes.

The same process with Bob's entire record removed.

The probabilities of any released outcome differ by at most a factor of $e^{\varepsilon}$. This limits how much Alice can learn about Bob's participation or preferences from the recommendation.

## SPARSE VECTOR - OTHER APPROACHES

By properly adapting the parameter of the Laplace and the threshold we can also expand our above threshold algorithm and remain $\varepsilon$ differentially private. In particular:

“Sparse” allows to output up to “c” above threshold results

“NumericSparse” allows to output up to “c” above threshold results and their numerical value

Obviously, to achieve such result, we increase the noise and lose accuracy.

pseudocode here: [https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf](https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf)