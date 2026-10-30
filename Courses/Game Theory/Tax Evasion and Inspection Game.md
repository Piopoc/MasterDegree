---
type: course-note
date: 2026-10-26
tags:
  - course
  - game-theory
  - parametric-game
  - comparative-statics
course: "[[Game Theory]]"
sources:
  - "[[GuidaEs]]"
ai-first: true
---

## For Future Agent

Study note for the [[Game Theory]] course on a parametric 2×2 tax-evasion and inspection game (Taxpayer: Hide/Pay full; Inspector: Check/Do not check) with parameters R, S, L, E, F, p. Two equilibrium regimes: pure (Hide, Do not check) when inspection is inefficient, and a mixed equilibrium under strategic deterrence. Comparative statics show the counterintuitive result that harsher penalties reduce audit frequency rather than evasion propensity. Created 2026-10-26 from [[GuidaEs]].

## 1. Players, Strategies, Parameters

- **Players**: Taxpayer (T) and Fiscal Inspector (I).
- **Strategies**: S_T = {H, P} (Hide/evade, Pay full); S_I = {C, D} (Check, Do not check).
- **Parameters**: R = net income after regular taxes (R > 0); S = total taxes due (S > 0); L = illicit gain from evasion (0 < L ≤ S); E = inspector's operational cost per check (E > 0); F = monetary fine collected on detection (F > 0); p = detection probability after a check (0 < p ≤ 1).

## 2. Payoff Functions (4 Profiles)

| Profile | T's payoff u_T | I's payoff u_I |
|---|---|---|
| (P, D) | R | S |
| (P, C) | R | S - E |
| (H, D) | R + L | S - L |
| (H, C) | p(R - F) + (1-p)(R + L) = R + (1-p)L - pF | p(S + F) + (1-p)(S - L) - E = S - (1-p)L + pF - E |

## 3. Parametric 2×2 Matrix

| T \ I | Check (C) | Do not check (D) |
|---|---|---|
| **Hide (H)** | R + (1-p)L - pF, S - (1-p)L + pF - E | R + L, S - L |
| **Pay full (P)** | R, S - E | R, S |

## 4. Best Responses

- **Inspector**: against T = P, **D** is strict (S > S - E). Against T = H, I checks iff S - (1-p)L + pF - E > S - L ⇔ **p(L + F) > E**.
- **Taxpayer**: against I = D, **H** is strict (R + L > R). Against I = C, T pays iff R > R + (1-p)L - pF ⇔ **pF > (1-p)L**.

## 5. Regime A - Inspection Cost Inefficient (p(L + F) ≤ E)

D is strictly dominant for the inspector; given D, H is strictly dominant for the taxpayer. **Unique pure NE: (H, D)** with payoffs (R + L, S - L).

## 6. Regime B - Strategic Deterrence and Active Audit (p(L + F) > E and pF > (1-p)L)

Best responses cycle: (H, C) → (P, C) → (P, D) → (H, D) → (H, C), so no pure equilibrium exists. Let q = P(T = H), r = P(I = C).

- Inspector's indifference: q* = **E / (p(L + F))** (admissible in (0, 1) by p(L + F) > E).
- Taxpayer's indifference: r* = **L / (p(L + F))**.

**Mixed NE**: (m_T^* = (E/(p(L+F)), 1 - E/(p(L+F))), m_I^* = (L/(p(L+F)), 1 - L/(p(L+F)))).

## 7. Comparative Statics and Interpretation

- **Evasion rate q***: depends only on the inspector's parameters (E, p, F), not on the tax burden S or the evasion gain L. Raising the fine F or detection p does not directly reduce evasion propensity; it reduces the inspection frequency the inspector needs.
- **Audit frequency r***: rises with the illicit gain L and falls with the fine F and detection p. Sanctions and detection act as **indirect deterrence**: a higher F lets the state sustain the same compliance with fewer costly controls, minimizing public inspection spending E.

## 8. Exam Hooks

- Derive payoffs profile by profile before assembling the parametric matrix; keep all parameters symbolic.
- Compare regimes by checking parameter conditions first (pure vs. mixed existence), then solve indifference equations.
- In control games with mixed equilibria, the equilibrium behavior of one player is pinned down by the *other player's* parameters - a classic comparative-statics trap.

## Related

- [[Static Games of Complete Information]] - best response, pure and mixed Nash equilibrium, indifference
- [[Game Theory]] - course hub

## Sources

- [[GuidaEs]] (AY 2026/2027)
