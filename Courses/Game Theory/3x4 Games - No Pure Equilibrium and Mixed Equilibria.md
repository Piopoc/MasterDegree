---
type: course-note
date: 2026-10-26
tags:
  - course
  - game-theory
  - nxn-games
  - mixed-strategies
  - pareto-optimality
course: "[[Game Theory]]"
sources:
  - "[[GuidaEs]]"
ai-first: true
---

## For Future Agent

Study note for the [[Game Theory]] course on a static 3×4 game (S_A = {X, Y, Z}, S_B = {J, K, L, M}) with no pure Nash equilibrium, two verified interior mixed equilibria, and four Pareto-optimal pure profiles. Created 2026-10-26 from [[GuidaEs]].

## 1. The 3×4 Payoff Matrix

| A \ B | J | K | L | M |
|---|---|---|---|---|
| **X** | 6, 7 | 5, 5 | 3, 8 | 8, 1 |
| **Y** | 4, 9 | 9, 2 | 0, 4 | 2, 3 |
| **Z** | 8, 4 | 2, 8 | 4, 2 | 3, 6 |

## 2. Best-Response Sets

- BR_A(J) = {Z}; BR_A(K) = {Y}; BR_A(L) = {Z}; BR_A(M) = {X}.
- BR_B(X) = {L}; BR_B(Y) = {J}; BR_B(Z) = {K}.

## 3. No Pure Nash Equilibrium

Reciprocity check on all 12 pure cells:

- (X, L): BR_B(X) = L but BR_A(L) = Z ≠ X.
- (Y, J): BR_B(Y) = J but BR_A(J) = Z ≠ Y.
- (Z, K): BR_B(Z) = K but BR_A(K) = Y ≠ Z.
- (X, M): BR_A(M) = X but BR_B(X) = L ≠ M.

No pure profile is mutually best-responding → **no pure Nash equilibrium**.

## 4. Mixed NE Profile 1

m_A = (2/3, 0, 1/3), m_B = (5/11, 4/11, 2/11, 0).

- For A: U_A(X, m_B) = U_A(Y, m_B) = U_A(Z, m_B) = 56/11 — all pure strategies of A, including Y outside supp(m_A) = {X, Z}, yield the same expected payoff, so A's best-response condition holds.
- For B: U_B(m_A, J) = U_B(m_A, K) = U_B(m_A, L) = 6, while U_B(m_A, M) = 8/3 < 6 → m_B is a best response to m_A.

## 5. Mixed NE Profile 2

m_A = (0, 4/11, 7/11), m_B = (7/11, 4/11, 0, 0).

- For A: strategies in supp(m_A) = {Y, Z} yield 64/11, while X outside the support yields 62/11 < 64/11 → m_A is a best response.
- For B: strategies in supp(m_B) = {J, K} yield 64/11, while L (30/11) and M (54/11) yield less → m_B is a best response.

## 6. Pareto-Optimal Pure Profiles

Evaluating all 12 pure profiles, the Pareto-undominated ones are:

- **(X, J)** → (6, 7)
- **(Y, J)** → (4, 9) — B's maximum payoff
- **(Y, K)** → (9, 2) — A's maximum payoff
- **(Z, J)** → (8, 4)

All other profiles are strictly dominated by one of these.

## 7. Exam Hooks

- To rule out pure equilibria, compare BR_A(s_{-i}) against BR_B(s_i) cell by cell across the whole matrix.
- Verify a mixed equilibrium by checking both conditions: equal payoffs across the support, and no strictly better off-support strategy.
- Pareto optimality compares payoff vectors componentwise: a profile is Pareto-optimal if no other profile makes everyone weakly better off and someone strictly better off.

## Related

- [[Static Games of Complete Information]] - mixed equilibrium conditions, principle of indifference, Pareto optimality
- [[Game Theory]] - course hub

## Sources

- [[GuidaEs]] (AY 2026/2027)
