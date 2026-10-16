# Index

> Catalog of all vault pages. Read this first when navigating the vault - faster and cheaper on tokens than searching.
> Format: `- [[Intro]] - brief description`. Update whenever a note is created or deleted.

## Courses

### Privacy Preserving Information Access
- [[Privacy Preserving Information Access]] - course index: identity, note list, conceptual map, raw sources
- [[Data Security and Privacy Fundamentals]] - attribute taxonomy, security vs. privacy, GDPR, privacy as evolutionary need
- [[Database Foundations and Architecture]] - DB/DBMS/UoD, ANSI/SPARC levels, design phases, relational model, platform case studies
- [[SQL Injection and Defenses]] - SQLi anatomy, OWASP ranking, PreparedStatement, tautology and DDL attacks
- [[Macrodata Protection]] - release paradigms, sensitive cells, suppression/rounding/CTA, (n,k)-rule and p-percentage rule
- [[Microdata Disclosure Risks and Sanitization]] - identity/attribute/inferential disclosure, outliers, natural defenses, desiderata
- [[Microdata Masking and Synthetic Data]] - full masking + synthetic-data technique catalog with applicability matrix
- [[Privacy Evaluation Metrics]] - confidentiality measurement, information loss, R-U map (mind-map level, details TBD)
- [[k-Anonymity]] - identity-linking threat, formal definition and q-blocks, generalization/suppression, k-minimality, Mondrian algorithm

### Advanced Text Analytics
- [[Advanced Text Analytics]] - course index: identity (partly TBD), note list, reassignment record
- [[NLP Linguistic Structures and Annotation]] - word vs. token, annotation pipeline, phrase/clause hierarchy, dependency vs. constituency grammar
- [[Knowledge Resources for Text Analytics]] - gazetteers, terminologies, controlled vocabularies, taxonomies, thesauri, ontologies, knowledge bases/graphs; mention recognition vs concept normalization
- [[Annotation and Evaluation of Information Extraction]] - span annotation, BIO/BILOU, Cohen's kappa, P/R/F1, exact vs relaxed matching, micro vs macro averaging, relation and pipeline evaluation

### Game Theory
- [[Game Theory]] - course index placeholder (no raw material ingested yet)

### Graph Databases
- [[Graph Databases]] - course index: identity (partly TBD), note list, ingest checklist
- [[LOTUS Semantic Operators]] - semantic operators, langex, reference algorithms, accuracy-guaranteed optimization, PVLDB 2025 citation (reassigned from Advanced Text Analytics 2026-10-09)
- [[Linked Open Data]] - open data licensing, TBL design principles, URI/URL distinction, content negotiation, hash vs slash URIs, 5-star schema
- [[RDF Data Model]] - triples, identity resolution and graph merging, CURIEs/namespaces, literals, blank nodes, reification, named graphs, serializations

## Raw Sources (immutable)

- [[Intro]] - "Fondamenti di Basi di Dati, Sicurezza e Privacy" - Italian quiz/glossary on DB fundamentals, SQLi, privacy
- [[Macrodata and Microdata]] - English briefing on macrodata and microdata protection (PPIA, AY 2026/2027)
- [[microdati]] - Italian catalog of microdata masking and synthetic-data techniques
- [[NLP]] - English briefing on linguistic structures and computational annotations in NLP
- [[ATA_L05]] - English briefing on knowledge resources for text analytics (Advanced Text Analytics, Lecture 5)
- [[ATA_L06]] - English briefing on annotation and evaluation of information extraction (Advanced Text Analytics, Lecture 6)
- [[LOTUS]] - Italian summary of the Patel et al. PVLDB 2025 LOTUS paper
- [[LOD]] - English briefing on Linked Open Data: principles, architecture, 5-star deployment (Graph Databases)
- [[RDF]] - English technical briefing on the Resource Description Framework (Graph Databases)
- [[k-anonymity]] - English briefing on the k-anonymity framework: definition, generalization/suppression, minimality, Mondrian (PPIA)

## Maps (important referenced images)

- (see -> maps/NotebookLM Mind Map.png) - NotebookLM mind map of the macro/microdata source (includes evaluation-metrics branch; moved here from raw/ when cited)

## Daily

- [[2026-10-10]] - daily note: vault work session recap (health cleanup, kanban + GCal setup, maps workflow, LOD/RDF ingest), open follow-ups
- [[2026-10-16]] - daily note: ATA Lectures 5 & 6 ingest into Advanced Text Analytics (Knowledge Resources; Annotation and Evaluation)

## Root

- [_OPENCODE.md](_OPENCODE.md) - operating rules for agents in this vault (read first)
- [[Kanban]] - kanban board for course tasks (Graph Databases / Advanced Text Analytics / PPIA)
- [[LaTeX Box Toolkit]] - recipes for LaTeX boxes in notes (definition boxes, cross-link boxes)
- [log.md](log.md) - append-only operation log
- [index.md](index.md) - this file
- `maps/` - folder of important images, cited as plain text (see -> maps/...)

---
*Stats: 18 course files (14 study notes, 4 course indexes) + 10 raw sources + 1 map image. Citations live in each note's `## Sources` footer; structural links in `## Related`. Last refreshed: 2026-10-16.*
