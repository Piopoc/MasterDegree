# Index

> Root entry-point catalog. It lists **course indexes, folders and structural files only** - not
> individual study notes or raw sources, because each course index already lists its own notes and
> each study note carries its own `## Sources` footer.
> Format: `- [[Note Name]] - brief description`. Update when a course, a daily note or a root file changes.

## Courses

Each course index is the single place where that course's study notes are listed.

- [[Privacy Preserving Information Access]] - course index (8 study notes): data security & privacy fundamentals, database foundations, SQL injection, macrodata protection, microdata disclosure & sanitization, masking & synthetic data, evaluation metrics, k-anonymity
- [[Advanced Text Analytics]] - course index (5 study notes): NLP linguistic structures & annotation, knowledge resources, information extraction annotation & evaluation, TF-IDF & static embeddings, RNNs & LSTMs
- [[Graph Databases]] - course index (3 study notes): LOTUS semantic operators, Linked Open Data, RDF data model
- [[Game Theory]] - course index (5 study notes): static-games framework, four 2x2 cases, 3x4 mixed equilibria, thesis-supervisor and tax-evasion games

## Daily

- [[2026-10-10]] - vault work session recap (health cleanup, kanban + GCal setup, maps workflow, LOD/RDF ingest), open follow-ups

## Folders and assets

- `raw/` - immutable source texts (17 files). Every raw file is cited from the `## Sources` footer of the study note it produced - follow those links, do not list raw files here.
- `maps/` - referenced images, cited in notes as plain text `(see -> maps/...)`
- `Courses/` - one folder per course; each holds its course index note and the study notes it lists
- `Daily/` - daily notes, created only on user request (see `_OPENCODE.md` Section 4)

## Root files

- [_OPENCODE.md](_OPENCODE.md) - operating rules for agents in this vault (read first)
- [[Kanban]] - kanban board for course tasks (Graph Databases / Advanced Text Analytics / PPIA)
- [[LaTeX Box Toolkit]] - recipes for LaTeX boxes in notes (definition boxes, cross-link boxes)
- [log.md](log.md) - append-only operation log (the session audit trail)
- [index.md](index.md) - this file

---
*Stats: 4 courses (21 course files = 17 study notes + 4 course indexes) + 17 raw sources + 1 map image. Two-level catalog: this file = entry points, course indexes = note lists, study notes = provenance. Last refreshed: 2026-10-26.*
