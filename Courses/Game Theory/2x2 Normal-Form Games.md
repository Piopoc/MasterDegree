---
type: course-note
date: 2026-10-26
tags:
  - course
  - game-theory
  - two-player-games
  - best-response
course: "[[Game Theory]]"
sources:
  - "[[GuidaEs]]"
ai-first: true
---

## For Future Agent

Study note for the [[Game Theory]] course on solving static 2x2 normal-form games through best-response coincidence, with four worked cases (a)-(d) covering dominant strategies, unique pure equilibria, absence of pure equilibria, coordination equilibria, and interior mixed equilibria. Created 2026-10-26 from [[GuidaEs]].

## 1. Solution Procedure for 2x2 Games

To find Nash equilibria in a 2×2 payoff matrix:

1. **Best responses of A (rows)**: fix B's first column, pick A's best row; fix B's second column, pick A's best row.
2. **Best responses of B (columns)**: fix A's first row, pick B's best column; fix A's second row, pick B's best column.
3. **Pure equilibria**: cells where both players' best responses coincide.
4. **Mixed equilibrium**: with m_A = (p, 1-p), p = P(A = F), and m_B = (q, 1-q), q = P(B = f), impose the indifference equations U_B(F, m_B) = U_B(G, m_B) to solve q* and U_A(m_A, f) = U_A(m_A, g) to solve p*. An internal mixed equilibrium exists iff p*, q* ∈ (0, 1).

## 2. Case (a) - Strict Dominance for B, Unique Pure NE

**Payoff matrix:**

| A \ B | f | g |
|---|---|---|
| **F** | 2, 4 | 0, 1 |
| **G** | 1, 6 | 3, 5 |

- BR_A(f) = F (2 > 1); BR_A(g) = G (3 > 0).
- BR_B(F) = f (4 > 1); BR_B(G) = f (6 > 5) — **f strictly dominates g for B**.
- Coincidence: cell **(F, f)** → unique pure NE = {(F, f)}, payoff (2, 4).
- Mixed check: U_B(m_A, f) = U_B(m_A, g) ⇒ 4p + 6(1-p) = 1p + 5(1-p) ⇒ p* = -1/2 ∉ [0, 1] → no internal mixed equilibrium.

## 3. Case (b) - No Pure NE, Interior Mixed NE

**Payoff matrix:**

| A \ B | f | g |
|---|---|---|
| **F** | 0, 4 | 3, 0 |
| **G** | 6, 0 | 0, 5 |

- BR_A(f) = G; BR_A(g) = F. BR_B(F) = f; BR_B(G) = g.
- No cell satisfies reciprocity → **no pure Nash equilibrium**.
- Mixed: A's indifference U_A(F, m_B) = U_A(G, m_B) ⇒ 3(1-q) = 6q ⇒ q* = 1/3. B's indifference U_B(m_A, f) = U_B(m_A, g) ⇒ 4p = 5(1-p) ⇒ p* = 5/9.
- **NE = {(m_A = (5/9, 4/9), m_B = (1/3, 2/3))}**.

## 4. Case (c) - Two Pure NE plus an Interior Mixed NE

**Payoff matrix:**

| A \ B | f | g |
|---|---|---|
| **F** | 9, 3 | 2, 2 |
| **G** | 0, 0 | 3, 9 |

- Pure NE: **(F, f)** and **(G, g)** (mutual best responses in both cells).
- Mixed: A's indifference ⇒ 9q + 2(1-q) = 3(1-q) ⇒ q* = 1/10. B's indifference ⇒ 3p = 2p + 9(1-p) ⇒ p* = 9/10.
- **NE = {(F, f), (G, g), (m_A = (9/10, 1/10), m_B = (1/10, 9/10))}** — a coordination-like structure with a mixed equilibrium on top.

## 5. Case (d) - Strict Dominance for Both Players

**Payoff matrix:**

| A \ B | f | g |
|---|---|---|
| **F** | 2, 2 | 0, 6 |
| **G** | 6, 0 | 1, 1 |

- **G strictly dominates F for A** (6 > 2 against f; 1 > 0 against g). **g strictly dominates f for B** (6 > 2 against F; 1 > 0 against G).
- Unique pure NE: **(G, g)**, payoff (1, 1). Strict dominance rules out any internal mixed equilibrium with full support.

## 6. Exam Hooks

- Always check best responses for both players before concluding; a cell is an equilibrium only if it is a best response for *both* players simultaneously.
- Strictly dominated strategies are never played with positive probability in any Nash equilibrium.
- If the indifference solution gives p* or q* outside [0, 1], no internal mixed equilibrium exists.

## Related

- [[Static Games of Complete Information]] - normal-form definition, best response, Nash equilibrium, indifference principle
- [[Game Theory]] - course hub

## Sources

- [[GuidaEs]] (AY 2026/2027)
