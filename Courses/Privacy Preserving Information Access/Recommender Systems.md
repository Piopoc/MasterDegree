---
type: course-note
date: 2026-11-04
tags:
  - course
  - ppia
  - recommender-systems
  - collaborative-filtering
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[IR and RS]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course (Lecture 4, part 2) covering **recommender systems**: the cognitive motivation (paradox of choice, decision fatigue), the user-as-query architecture, the rating matrix, the content-based vs collaborative-filtering taxonomy (popularity, memory-based k-NN, model-based matrix factorization) and the four evaluation framings (classification, ranking, AUC, rating prediction). Created 2026-11-04 from the L4 technical report [[IR and RS]]; part 1 is [[Information Retrieval Foundations]]. Key points: user $u \in U$ acting as the query, explicit vs implicit feedback, pros/cons of each CF variant, RMSE for rating prediction, and why privacy is a *trust* requirement here rather than only a legal one.

## 1. Why Recommender Systems Exist

The cognitive premise is the **paradox of choice**: users like having options up to a threshold; beyond it, choosing consumes more cognitive resources than the choice is worth. The result is either **avoidance** (no selection at all) or **anxiety** about whether the choice was correct. The handwritten annotation on the slide names the phenomenon **decision fatigue**.

A recommender system is therefore not a search box with extra steps: it exists to *reduce the decision space* before the user has to act. That is also why its output is a ranking rather than a set - the user needs an order, not a catalog.

## 2. Definition and Architecture

A recommender takes a user $u \in U$, an item set $I$, and computes a numerical score $s$ for every item $i \in I$; a higher score means higher predicted interest.

```text
              [ Recommender System ]
                        |
     +------------------+------------------+
     v                                     v
User (u in U)                        Item Set (I)
  |  (the user acts as the query)        |
  +------------------+------------------+
                     v
        Compute score s for each item i
                     v
            Ranked Recommendations
```

The structural symmetry with IR is the point: documents are replaced by items, the query is replaced by **the user**, and the retrieval score becomes a preference score.

### Four properties stated on the slides

| Property | Content |
|---|---|
| **User as query** | A recommender is an *indirect query to a search engine* in which *the user themself is the query* |
| **Personalization need** | Personalized systems are needed to exploit **user feedback, fresh information, and rich information** |
| **Implicit feedback** | When feedback is uncertain or unavailable, the system infers preference from behaviour rather than ratings |
| **Trust** | User trust is essential; privacy protection is required **to make the system trustworthy**, not only to comply with the law |

The trust point is the direct bridge to the rest of this course: a recommender only works if users keep feeding it data, and they stop feeding it data once they do not trust what happens to that data.

## 3. The Rating Matrix

Interactions are stored in a **rating matrix** of users $\times$ items. Its entries do **not** have to be star ratings: they can be

- **binary** signals (clicked / not clicked, liked / not liked), or
- **continuous** values (e.g. watch duration on a video platform).

The matrix is typically **sparse** - each user has interacted with a tiny fraction of the catalog - and that sparsity is what every collaborative technique has to work around.

## 4. Taxonomy

```text
                      [ Recommender Systems ]
                                 |
        +------------------------+------------------------+
        v                                                 v
Content-Based                              Collaborative Filtering
(item & user attribute matching)                        |
                                    +-------------------+-------------------+
                                    v                                       v
                              Non-Personalized                        Personalized
                            (popularity filtering)                          |
                                                          +-----------------+-----------------+
                                                          v                                   v
                                                   Memory-Based (k-NN)         Model-Based (Matrix Factorization)
```

### Content-based

- Items are characterized by **explicit attributes** (colour, cost, genre, actors, material).
- Users are profiled by the attributes of the items they preferred.
- Recommendation = match between item attributes and the user profile.

It needs no other users, but it can only recommend what resembles what the user already consumed - no serendipity, and the quality is capped by the richness of item metadata.

### Collaborative filtering (CF)

CF exploits the behaviour of **many users**, which requires users to be willing to share data (explicitly or implicitly).

#### Popularity filtering (non-personalized)

Recommends the items with the highest overall interaction.

| PROS | CONS |
|---|---|
| Trivially cheap to compute | No personalization at all |
| Presents no privacy issue - the top ranks can be computed with **differential privacy** | - |
| Strong practical performance | - |

The handwritten caveat: popularity targets the **most interacted** items, *which are not necessarily the best ones*. Popularity measures attention, not quality.

#### Memory-based CF (personalized)

Extends **k-nearest neighbours**, keeping the full rating matrix in memory and computing user-user (or item-item) similarity to infer missing ratings:

$$s_{u,i} = \frac{\sum_{v \in U_i} r_{v,i}\,\cos(u,v)}{r_{\max}\cdot \lvert\{v \in U_i\}\rvert}$$

where $U_i$ is the set of users who rated item $i$: their ratings for $i$ are weighted by similarity to the target user $u$, normalized by the maximum rating $r_{\max}$ and the number of raters.

- **PROS** - simple computation, effective output.
- **CONS** - high memory cost (the whole matrix must stay resident), so it does not scale to large catalogs.

#### Model-based CF (matrix factorization)

Factorizes the rating matrix into two smaller matrices, users $\times$ **latent features** and latent features $\times$ items.

The handwritten note states the motivation exactly: it is applied *to reduce memory consumption*. The side effect is a form of compression: raw interactions are projected onto latent vectors, which removes noise and represents the intrinsic preference of the user and the intrinsic character of the item. This is the same dimensionality-reduction logic that later shows up as embedding models in modern recommenders.

## 5. Evaluating Recommender Systems

The metric depends on how the task is framed - and the four framings below are the exam-relevant result.

```text
                    [ Recommender System Evaluation ]
                                 |
     +--------------+------------+-------------+--------------+
     v              v                          v              v
Classification   Ranking               ROC / AUC       Rating Prediction
  (P, R, F1)   (nDCG, AP)      (Area Under ROC Curve)        (RMSE)
```

| Framing | Metrics | When it applies |
|---|---|---|
| **Classification** | Precision, Recall, F1-score | Ratings are binarized into relevant / not relevant |
| **Ranking** | nDCG, AP | The output is an ordered list (the natural reading of a recommender) |
| **ROC / AUC** | Area under the ROC curve (TPR vs FPR) | Common in recommender evaluation, rarely used in traditional IR |
| **Rating prediction** | RMSE | The system predicts an explicit numerical rating |

Definitions inherited from IR: **precision** = share of recommended items that are relevant, **recall** = share of relevant items that are recommended, **F1** = their harmonic mean, **nDCG** and **AP** exactly as in [[Information Retrieval Foundations]].

Root Mean Squared Error over the test users $\mathcal{U}_T$ and test items $\mathcal{I}_T$:

$$\text{RMSE} = \sqrt{\frac{\sum_{u \in \mathcal{U}_T}\sum_{i \in \mathcal{I}_T}\left(\hat{r}_{u,i} - r_{u,i}\right)^2}{\lvert \mathcal{U}_T\rvert \cdot \lvert \mathcal{I}_T\rvert}}$$

with $\hat{r}_{u,i}$ the predicted rating and $r_{u,i}$ the true one. RMSE is scale-dependent and penalizes large errors quadratically - which is what you want for rating prediction, but not what you want when the top of the ranking is what matters.

## 6. The Privacy Trade-off Inside the Recommender

The lecture's implicit argument, worth making explicit for this course:

- **Popularity filtering** is privacy-friendly almost for free: aggregate top-$k$ counts can be released with **differential privacy**, and no individual profile is needed.
- **Collaborative filtering** is where the cost appears: it works only if users *share* their interactions, so the rating matrix is itself a sensitive dataset - and it is exactly the kind of data that re-identification attacks target.
- **Personalization buys utility at the price of exposure**: the more the system knows about $u$, the better the ranking and the worse the privacy posture.

This is why the trust annotation matters: privacy protection is stated as a *design requirement for trustworthiness*, not as a compliance afterthought. It also closes the loop with the protection machinery in the rest of the course - masking, k-anonymity and differential privacy are the tools that make a usable-but-protected interaction matrix possible.

## 7. Exam Hooks

- Paradox of choice and **decision fatigue**: why "more options" is not "better".
- The user acts as the query; a recommender is an indirect query to a search engine.
- Rating matrix: users $\times$ items, binary or continuous, sparse.
- Draw the taxonomy tree and attach PROS/CONS to each leaf.
- Memory-based formula: similarity-weighted average of the raters, normalized by $r_{\max}$ and the number of raters - and its memory blow-up.
- Matrix factorization = memory reduction + latent features + denoising.
- Four evaluation framings with their metrics; RMSE formula; why nDCG for ranking and RMSE for rating prediction.
- Popularity filtering + differential privacy is the cheapest privacy-preserving configuration; CF is where the exposure lives.

## Related

- [[Information Retrieval Foundations]] - part 1 of the same lecture; source of P/R/F, AP, nDCG and the ranking framing
- [[Privacy Evaluation Metrics]] - the course's privacy-side measurement vocabulary, complementary to the utility metrics used here

## Sources

- [[IR and RS]] (AY 2026/2027)
