---
type: course-note
date: 2026-10-26
tags:
  - course
  - game-theory
  - prose-to-game
  - iestds
course: "[[Game Theory]]"
sources:
  - "[[GuidaEs]]"
ai-first: true
---

## For Future Agent

Study note for the [[Game Theory]] course formalizing a prose problem (thesis supervisor choice) as a 3×3 normal-form game with congestion. Two players (Charlotte, Daniel) choose among Xavier (X), Yuan (Y), Zingberry (Z); nominal values v(X)=40, v(Y)=60, v(Z)=50; a coinciding choice cuts each player's payoff to 70% of the base value. Two pure equilibria (Y, Z) and (Z, Y), strict dominance of X, and one interior mixed equilibrium. Created 2026-10-26 from [[GuidaEs]].

## 1. Players, Strategies, Parameters

- **Players**: Charlotte (C) and Daniel (D).
- **Strategies**: S_C = S_D = {X, Y, Z} where X = Xavier, Y = Yuan, Z = Zingberry.
- **Nominal supervision values**: v(X) = 40, v(Y) = 60, v(Z) = 50.
- **Congestion rule**: if both pick the same supervisor, availability halves and each retains 70% of the base value: 0.70 · v(·).

## 2. Payoffs for the 9 Profiles

| Profile | C's payoff | D's payoff |
|---|---|---|
| (X, X) | 40 × 0.70 = 28 | 28 |
| (X, Y) | 40 | 60 |
| (X, Z) | 40 | 50 |
| (Y, X) | 60 | 40 |
| (Y, Y) | 60 × 0.70 = 42 | 42 |
| (Y, Z) | 60 | 50 |
| (Z, X) | 50 | 40 |
| (Z, Y) | 50 | 60 |
| (Z, Z) | 50 × 0.70 = 35 | 35 |

## 3. Normal-Form Matrix (3×3)

| C \ D | Xavier (X) | Yuan (Y) | Zingberry (Z) |
|---|---|---|---|
| **Xavier (X)** | 28, 28 | 40, 60 | 40, 50 |
| **Yuan (Y)** | 60, 40 | 42, 42 | 60, 50 |
| **Zingberry (Z)** | 50, 40 | 50, 60 | 35, 35 |

## 4. Pure Nash Equilibria

Best responses:

- C: D = X → Y (60 > 50 > 28); D = Y → Z (50 > 42 > 40); D = Z → Y (60 > 40 > 35).
- D: C = X → Y; C = Y → Z; C = Z → Y.

Mutual best responses → **pure NE: (Y, Z) = (60, 50) and (Z, Y) = (50, 60)**.

## 5. Iterated Elimination of Strictly Dominated Strategies (IESTDS)

Xavier X is **strictly dominated by Yuan Y** for both players (60 > 28, 42 > 40, 60 > 40 for C; symmetric for D). X is never played with positive probability in any Nash equilibrium. Eliminating X leaves a 2×2 game on {Y, Z}:

| C \ D | Yuan (Y) | Zingberry (Z) |
|---|---|---|
| **Yuan (Y)** | 42, 42 | 60, 50 |
| **Zingberry (Z)** | 50, 60 | 35, 35 |

## 6. Mixed Equilibrium

With m_C = (0, q, 1-q), m_D = (0, p, 1-p) where q, p = P(choose Y):

- C's indifference: 42p + 60(1-p) = 50p + 35(1-p) ⇒ p* = 25/33.
- D's indifference (structural symmetry): q* = 25/33.

**Complete equilibrium set**: (Y, Z); (Z, Y); mixed (m_C = (0, 25/33, 8/33), m_D = (0, 25/33, 8/33)).

## 7. Exam Hooks

- When formalizing prose, extract players, strategy sets, and parameters first, then build the payoff table cell by cell before writing the matrix.
- Strict dominance (IESTDS) can shrink the game before computing equilibria; dominated strategies carry zero probability in every NE.
- In the reduced 2×2 game, the mixed equilibrium makes each player indifferent between Y and Z.

## Related

- [[Static Games of Complete Information]] - best response, Nash equilibrium, strict dominance, mixed equilibrium, indifference principle
- [[Game Theory]] - course hub

## Sources

- [[GuidaEs]] (AY 2026/2027)
