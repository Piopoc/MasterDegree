---
type: course
date: 2026-10-09
tags:
  - course
  - ppia
ai-first: true
---

## For Future Agent

Course index for **Privacy Preserving Information Access (PPIA)** - the hub note linking all study notes for this course. Created 2026-10-09 when the vault's `Courses/` structure was bootstrapped from `raw/` material. The course identity (University of Padua, AY 2026/2027, taught by Guglielmo Faggioli) is stated in [[Macrodata and Microdata]]. Related: [[Advanced Text Analytics]], [[Game Theory]], [[Graph Databases]].

## Course Identity

| Field | Value |
|---|---|
| Course | Privacy Preserving Information Access |
| Institution | University of Padua |
| Academic Year | 2026/2027 |
| Instructor | Guglielmo Faggioli |
| Status | active |

## Notes

1. [[Data Security and Privacy Fundamentals]] - data/attribute taxonomy, security vs. privacy, GDPR and legal framework, privacy as evolutionary need.
2. [[Database Foundations and Architecture]] - DB/DBMS/UoD, ANSI/SPARC, design phases, relational model, platform case studies.
3. [[SQL Injection and Defenses]] - SQLi anatomy, OWASP history, PreparedStatement, tautology vs. DDL attacks.
4. [[Macrodata Protection]] - release paradigms, sensitive cells, suppression/rounding/CTA, $(n,k)$-rule and $p$-percentage rule.
5. [[Microdata Disclosure Risks and Sanitization]] - identity/attribute/inferential disclosure, outliers, natural defenses, desiderata.
6. [[Microdata Masking and Synthetic Data]] - full technique catalog (non-perturbative, perturbative, synthetic) with applicability matrix.
7. [[Privacy Evaluation Metrics]] - confidentiality measurement, information loss, R-U map (mind-map level, details TBD).
8. [[k-Anonymity]] - identity-linking threat, formal definition and q-blocks, generalization/suppression, k-minimality metrics, Mondrian algorithm.

## Conceptual Map

```
Fundamentals ──▶ Macrodata Protection ──▶ Microdata Protection ──▶ Evaluation
(Data Security      (Macrodata            (Microdata Disclosure    (Privacy
 and Privacy         Protection)           Risks and Sanitization,  Evaluation
 Fundamentals)                             Microdata Masking        Metrics)
                                           and Synthetic Data,
                                           k-Anonymity)
                                           and Synthetic Data)
```

Links live in the numbered note list above (wikilinks are not rendered inside code blocks). This mirrors the structure of the (see -> maps/NotebookLM Mind Map.png) for the core reading.

## Raw Sources for This Course

- [[Intro]] - "Fondamenti di Basi di Dati, Sicurezza e Privacy" (Italian, quiz + glossary format)
- [[Macrodata and Microdata]] - full briefing on macro/microdata protection (English)
- [[microdati]] - detailed masking-technique catalog (Italian)
- [[k-anonymity]] - English briefing on the k-anonymity framework (definition, generalization/suppression, Mondrian)
- (see -> maps/NotebookLM Mind Map.png) - mind map of the macro/microdata source (includes evaluation-metrics branch absent from the markdown)
