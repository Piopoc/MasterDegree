---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - microdata
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Macrodata and Microdata]]"
  - "(see -> maps/NotebookLM Mind Map.png)"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering the **risks** of releasing microdata: the three disclosure types, re-identification attacks, natural defenses, sanitization, and the privacy-utility desiderata. Created 2026-10-09 from §3 of [[Macrodata and Microdata]] plus the (see -> maps/NotebookLM Mind Map.png). Key points: identity / attribute / inferential disclosure, outliers as prime victims, four natural defenses, and the two desiderata that every protection technique must balance.

## 1. What is Microdata

Microdata consists of **individual-level records** representing unique entities (patients, citizens, corporations) that require privacy protection.

## 2. The Three Types of Disclosure

```
                  ┌───────────────────┐
                  │ Microdata Risks   │
                  └─────────┬─────────┘
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
Identity Disclosure   Attribute Disclosure  Inferential Disclosure
(direct re-id)        (infer sensitive      (statistical property
                       values)               inference)
```

1. **Identity Disclosure** - re-identification of a specific individual through direct or combined unique attributes (name, SSN, address).
2. **Attribute Disclosure** - learning an individual's *sensitive attribute value* via combinations of quasi-identifiers.
3. **Inferential Disclosure** - high-probability deduction of personal information from the statistical properties of the published dataset.

## 3. Risk Profile and Vulnerabilities

- **Outliers / highly visible tuples:** individuals with rare attributes (e.g. rare medical conditions) are the most vulnerable and exposed subjects - they are isolated by their own values.
- **Re-identification attacks:** joining sanitized microdata with external datasets through shared **quasi-identifiers** (see the taxonomy in [[Data Security and Privacy Fundamentals]]).

## 4. Natural Defenses of Microdata

Four properties of real-world data slow attackers down *for free*:

1. **Sampling subsets** - datasets rarely cover 100% of a population; uncertain whether the target is even present.
2. **Temporal lag** - years often separate collection from release; data is outdated.
3. **Natural noise** - inherent collection noise hinders accurate cross-referencing.
4. **Format heterogeneity** - inconsistent structures across datasets complicate linkages.

These are *defenses*, not guarantees - explicit protection techniques are still required ([[Microdata Masking and Synthetic Data]]).

## 5. Sanitization and the Core Desiderata

**Sanitization** starts by removing all explicit identifiers: names, exact street addresses, SSNs, phone numbers. Sanitization alone is **insufficient** because quasi-identifier linkage risk remains.

Data custodians must manage the fundamental **Microdata Desiderata** trade-off:

1. **Protect user privacy** - modify, hide, or transform data so individual re-identification becomes impossible.
2. **Maintain key statistical properties** - preserve analytical utility; over-scrambling transforms data into useless white noise.

Every technique in Microdata Masking and Synthetic Data is an attempt to land between these two poles; [[Privacy Evaluation Metrics]] measures where a given release lands.

## 6. Exam Hooks

- The three disclosures answer three different attacker questions: *who is this?* (identity), *what is their value for X?* (attribute), *what is probably true about them?* (inferential).
- Why do outliers matter most? Because rarity = uniqueness = re-identification leverage.
- Desiderata framing is the standard answer structure for "discuss any protection technique": state which side of the trade-off it pushes.

## Related

- [[Data Security and Privacy Fundamentals]] - attribute taxonomy
- [[Microdata Masking and Synthetic Data]] - the technique catalog
- [[Macrodata Protection]] - aggregate-level twin of this note
- [[k-Anonymity]] - the formalized identity-linking threat

## Sources

- [[Macrodata and Microdata]] (AY 2026/2027; also the *Types of Disclosures* branch of the mind map)
- (see -> maps/NotebookLM Mind Map.png)
