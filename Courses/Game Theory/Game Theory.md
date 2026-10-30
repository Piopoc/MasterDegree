---
type: course
date: 2026-10-09
tags:
  - course
  - game-theory
ai-first: true
---

## For Future Agent

Course index for **Game Theory** - first raw material ingested 2026-10-26: [[GuidaEs]] (Italian exercise guide) translated and split into 5 study notes under this folder. Any future Game Theory lecture notes, slides, or exports dropped in `raw/` should be translated/split into notes in `Courses/Game Theory/` and linked from here. Related: [[Privacy Preserving Information Access]], [[Advanced Text Analytics]], [[Graph Databases]].

## Course Identity

| Field | Value |
|---|---|
| Course | Game Theory |
| Institution | TBD (not stated in sources) |
| Academic Year | 2026/2027 (assumed - same as other tracked courses; speculation) |
| Instructor | TBD (not stated in sources) |
| Status | active - first material ingested 2026-10-26 ([[GuidaEs]], exercise guide) |

## Notes

1. [[Static Games of Complete Information]] - normal-form framework: players, pure/mixed strategies, expected utility, best response, Nash equilibrium (pure & mixed), principle of indifference, Pareto optimality
2. [[2x2 Normal-Form Games]] - best-response procedure for 2×2 matrices; four worked cases (a)-(d): dominant strategies, unique pure NE, no-pure-NE + mixed NE, coordination equilibria, strict dominance
3. [[3x4 Games - No Pure Equilibrium and Mixed Equilibria]] - 3×4 game (S_A = {X,Y,Z}, S_B = {J,K,L,M}): no pure NE, two verified mixed NE profiles, four Pareto-optimal pure profiles
4. [[Thesis Supervisor Selection Game]] - prose-to-game formalization (3×3, congestion rule 0.70·v): pure NE (Y,Z) and (Z,Y), strict dominance of X via IESTDS, mixed equilibrium p* = q* = 25/33
5. [[Tax Evasion and Inspection Game]] - parametric 2×2 game (Hide/Pay full vs Check/Do not check): Regime A pure NE (H,D); Regime B mixed NE q* = E/(p(L+F)), r* = L/(p(L+F)); comparative statics

## First Ingest Checklist (completed 2026-10-26)

1. ✅ User dropped source material into `raw/` ([[GuidaEs]], Italian exercise guide, 2026-10-26).
2. ✅ Translated to English; split by topic into 5 study notes in this folder.
3. ✅ LaTeX added for formal content (utility functions, Nash equilibrium, mixed strategies, parametric payoffs).
4. ✅ Course index updated and `log.md` appended.
