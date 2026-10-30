---
type: course-note
date: 2026-10-26
tags:
  - course
  - game-theory
  - static-games
  - nash-equilibrium
course: "[[Game Theory]]"
sources:
  - "[[GuidaEs]]"
ai-first: true
---

## For Future Agent

Study note for the [[Game Theory]] course covering the methodological framework for static games of complete information in normal (strategic) form. Created 2026-10-26 by translating and analyzing [[GuidaEs]], an Italian exercise guide. Key points: the normal-form game triple G = ⟨N, (S_i), (u_i)⟩, pure vs. mixed strategies and supports, expected utility, best response, Nash equilibrium in pure and mixed strategies, the principle of indifference, and Pareto optimality.

## 1. The Normal-Form Game

A static game of complete information in normal (strategic) form models simultaneous interaction among rational agents. Formally, an n-player static game is the triple:

$$G = \left\langle N, (S_i)_{i \in N}, (u_i)_{i \in N} \right\rangle$$

where:

- **Players (N)**: the finite set of rational decision-makers, N = {1, 2, ..., n}. In two-player games the actors are conventionally **A** (Player 1, row) and **B** (Player 2, column).
- **Static**: every player moves simultaneously (or without observing the others' moves); complete information: the structure of the game (players, strategies, payoffs) is common knowledge.

## 2. Pure Strategies and Strategy Profiles

- **Pure strategies (S_i)**: the discrete set of deterministic actions available to player i ∈ N.
- **Strategy profile**: an n-tuple s = (s_1, ..., s_n) ∈ S where S = ∏_{i∈N} S_i is the joint strategy space.

## 3. Mixed Strategies and Support

- **Mixed strategies (Δ(S_i))**: the probability simplex over S_i. A mixed strategy m_i ∈ Δ(S_i) satisfies ∑_{s_i∈S_i} m_i(s_i) = 1 and m_i(s_i) ≥ 0.
- **Support**: supp(m_i) is the set of pure strategies to which m_i assigns strictly positive probability.

## 4. Expected Utility

- **Utility function**: u_i: S → ℝ assigns each pure profile s ∈ S a real payoff for player i.
- **Expected utility under mixed strategies**: for m = (m_1, ..., m_n),

$$U_i(m) = \sum_{s \in S} \left( \prod_{j \in N} m_j(s_j) \right) u_i(s)$$

## 5. Best Response

For a given opponents' profile s_{-i}, player i's best response is the (pure or mixed) strategy maximizing utility:

$$\text{BR}_i(s_{-i}) = \arg\max_{s_i \in S_i} u_i(s_i, s_{-i})$$

## 6. Nash Equilibrium in Pure Strategies

A pure profile s* = (s_1*, ..., s_n*) ∈ S is a **Nash equilibrium** iff every component is a best response to the others:

$$\forall i \in N, \quad u_i(s_i^*, s_{-i}^*) \geq u_i(s_i, s_{-i}^*), \quad \forall s_i \in S_i$$

## 7. Nash Equilibrium in Mixed Strategies and the Principle of Indifference

A mixed profile m* ∈ Δ(S) is a Nash equilibrium iff no player can increase expected utility by deviating unilaterally. For **internal mixed equilibria** (non-singleton support), the **principle of indifference** applies: if a mixed strategy m_i* assigns strictly positive probability to two or more pure strategies, those pure strategies must yield player i the same expected payoff given m*_{-i}.

## 8. Pareto Optimality

A profile s ∈ S is **Pareto-optimal** (Pareto-undominated) if no other profile s' ∈ S satisfies u_i(s') ≥ u_i(s) for all i ∈ N with at least one strict inequality u_j(s') > u_j(s) for some player j.

## 9. Exam Hooks

- Memorize the normal-form triple and the two equilibrium conditions (pure: mutual best response; mixed: no profitable unilateral deviation).
- The principle of indifference is the workhorse for computing interior mixed equilibria: equalize expected payoffs across the support.
- Pareto optimality is an efficiency notion, distinct from Nash equilibrium: a Nash equilibrium need not be Pareto-optimal.

## Related

- [[Game Theory]] - course hub

## Sources

- [[GuidaEs]] (AY 2026/2027)
