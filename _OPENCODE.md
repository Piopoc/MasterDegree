# _OPENCODE.md - Obsidian Second Brain Operating Rules

> This file is the source of truth for how OpenCode operates your vault.
> Every session that starts in this vault should read this file first.
> Rules here override the defaults in the obsidian-second-brain skill.
>
> This is a **university study vault**: lecture material in `raw/` is processed into
> English course notes under `Courses/`, one folder per course.

---

## Section 0: AI-First Rule (CRITICAL)

Every note in this vault is written **for future agents** to read and reason over, not for human review.

### The Seven Requirements

1. **Self-contained context** - each note explains itself; don't rely on backlinks alone
2. **"For future agent" preamble** - 2-3 sentence summary so any compatible agent can decide relevance in 10 seconds
3. **Rich, consistent frontmatter** - `type`, `date`, `tags`, `ai-first: true`, plus type-specific fields
4. **Recency per note, not per claim** - a note's claims share one period (e.g. AY 2026/2027); recency lives in the `## Sources` footer and frontmatter, not inline
5. **Sources cited once, in the footer** - no inline citations; body claims stand naked and every source appears exactly once under `## Sources` at the end (mirrored in `sources:` frontmatter)
6. **Cross-links with intent** - every course, concept, and related note uses wiki-style links, never repeated within one body, following the Graph Linking Policy below
7. **Confidence levels** - `stated | high | medium | speculation` where applicable, kept bare (no source inside the marker)

### Sources and Recency (rule change 2026-10-15)

- **No inline citations.** Patterns like `(as of AY 2026/2027, [[RDF]])` or `(stated, [[Intro]])`
  are banned in note bodies - they polluted the prose and repeated the same citation endlessly.
- Every study note (and any root document that carries references) ends with two blocks, in order:

```markdown
## Related

- [[Linked Open Data]] - why it relates
- [[Database Foundations and Architecture]] - cross-course bridge

## Sources

- [[LOD]] (AY 2026/2027)
- (see -> maps/NotebookLM Mind Map.png)
```

- `## Sources` holds **one line per source, once per note**. Frontmatter `sources:` mirrors it for cheap agent reads.
- Confidence markers stay bare when useful: `(speculation)`, `(medium)` - never with a source inside.
- Course index notes keep their `## Raw Sources for This Course` section (that section IS their footer).

### Graph Linking Policy

Goal: the Obsidian graph reads as **dense course clusters + a peripheral provenance layer + a few intentional cross-course bridges** (layout/color config in `.obsidian/` stays untouched).

1. **Provenance layer** - each note links its raw sources exactly once (footer + frontmatter): 1 arc per note/source pair, raw files sit on the periphery.
2. **Course hub** - frontmatter `course: "[[Graph Databases]]"` (example) keeps every study note inside its course cluster.
3. **First occurrence** - link a term to its canonical note the FIRST time it appears in a body; never repeat that link in the same note.
4. **Cross-course bridges** - when a concept's canonical home is a note of *another* course, link it (in `## Related` if it's structural, inline once if it's topical).
5. **Existing targets only** - never create a note just to have something to link; no orphan scaffolding.

### Bi-Temporal Facts

NEVER delete old facts (course changes, grade updates, corrected material). Add new entries to the `timeline:` frontmatter array with both event time AND transaction time:

```yaml
timeline:
  - fact: "What was true"
    from: 2026-10-01
    until: 2026-12-20
    learned: 2026-10-09
    source: "[[Intro]]"
```

---

## Section 1: Your Vault Map

### Folder Structure

```
Master/
├── _OPENCODE.md          # This file - read first
├── index.md              # Catalog of all notes
├── log.md                # Operation log (append-only, monolithic)
├── Kanban.md             # Main kanban board (root - user preference)
├── Courses/              # ONE FOLDER PER COURSE (primary content)
│   ├── Privacy Preserving Information Access/
│   │   ├── Privacy Preserving Information Access.md   # course index
│   │   └── <study notes>.md                           # lecture-derived notes
│   ├── Advanced Text Analytics/
│   ├── Game Theory/
│   └── Graph Databases/
├── raw/                  # Immutable text sources (NEVER modify; images see maps/ rule below)
│   ├── Intro.md
│   ├── NLP.md
│   ├── LOTUS.md
│   ├── Macrodata and Microdata.md
│   └── microdati.md
├── maps/                 # Important referenced images (moved here from raw/ when cited in notes)
│   └── NotebookLM Mind Map.png
└── Daily/                # Daily notes (create when first needed, YYYY-MM-DD.md)
```

Standard folders from the skill schema (`People/`, `Projects/`, `Ideas/`, `Boards/`, etc.)
do not exist yet - create them on demand, never as empty scaffolding.

**Tool folders - never touch:** `.obsidian/`, `.space/`, `.makemd/`.

### Course Note Schemas

**Course index note** (one per course folder, named exactly like the course):
```yaml
---
type: course
date: 2026-10-09
tags: [course, <course-slug>]
ai-first: true
---
```

**Study note** (lecture-derived, translated to English):
```yaml
---
type: course-note
date: 2026-10-09
tags: [course, <course-slug>, <topic-tag>]
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Macrodata and Microdata]]"
ai-first: true
---
```

**Course slugs:** `ppia`, `advanced-text-analytics`, `game-theory`, `graph-databases`.

---

## Section 2: Your Naming Conventions

- **Study notes:** `Title.md` (evergreen, no date prefix - courses are chronological by content)
- **Dated notes:** `YYYY-MM-DD - Title.md` (daily, reviews)
- **Special characters:** No em dash (U+2014) or en dash (U+2013) in filenames
- **Safe hyphen:** Use ASCII hyphen `-` (U+002D) for word separators
- **Filenames must be vault-unique** so plain wikilinks resolve (course folders already guarantee this)

---

## Section 3: Your Writing Style

- **Language:** All study notes are written in **English**, even when the raw source is Italian
  (translations preserve meaning; link the Italian raw source in `sources:`)
- **Math:** LaTeX is encouraged - `$inline$` and `$$block$$` (obsidian-latex-suite is installed)
- **Tone:** didactic, concise, exam-oriented - structured headings, tables for comparisons
- **Links:** wikilinks for courses and notes, per the Graph Linking Policy - sources only in the
  `## Sources` footer, and never the same link twice in one body
- **Images:** plain-text pointer `(see -> maps/<filename>)` - never markdown links or embeds
  (user preference; markdown image paths with spaces fail to render). Images live in `maps/`
- **Dates:** ISO format `YYYY-MM-DD` in frontmatter, human format in body text
- **Emoji:** none in notes
- **Raw sources:** cited once at the end, in the footer, e.g. `- [[Intro]] (AY 2026/2027)` - never inline

---

## Section 4: Auto-Save Rules

- **Always save without asking:** course notes, decisions, tasks completed, ideas, daily notes
- **Ask first:** rewrites of existing study notes, destructive operations, anything touching `raw/`
- **Never modify:** `raw/` (immutable source of truth - re-derive notes from it if corruption occurs)
- **Never save:** API keys, passwords, tokens, sensitive credentials
- **Always propagate:** every new note also updates `index.md` and appends to `log.md`

---

## Section 5: Special Handling

### Courses tracked
- [[Privacy Preserving Information Access]] - active, sources: [[Intro]], [[Macrodata and Microdata]], [[microdati]], (see -> maps/NotebookLM Mind Map.png)
- [[Advanced Text Analytics]] - active, source: [[NLP]]
- [[Graph Databases]] - active, source: [[LOTUS]] (note: [[LOTUS Semantic Operators]] - moved here from ATA on 2026-10-09 by user decision)
- [[Game Theory]] - active course, **no raw sources yet** (TBD)

### Known cross-course overlaps
- Privacy/security fundamentals appear in both the DB fundamentals material and the
  macrodata/microdata material - keep the canonical explanation in
  [[Data Security and Privacy Fundamentals]] and link from elsewhere.

---

## Section 6: Board Names

- **Main Board:** `Kanban.md` (vault **root** - explicit user preference, NOT `Boards/`)
  - Single board covering tasks for [[Graph Databases]], [[Advanced Text Analytics]] and [[Privacy Preserving Information Access]]; identify the course per card with a course link (e.g. `[[Graph Databases]]`) or prefix
  - Columns: `## Backlog` / `## In Progress` / `## Waiting` / `## Done`
  - Card format: `- [ ] 🔴 **Title** · 📅YYYY-MM-DD` + tab-indented description line with wikilinks
  - Due dates use `📅YYYY-MM-DD` (the kanban plugin's parsed format - regex `/[📅📆🗓] *(\d{4}-\d{2}-\d{2})/u` verified in plugin code; the older `@{YYYY-MM-DD}` convention is NOT recognized); Google Calendar deep links can be added as markdown links inside cards, e.g. `https://calendar.google.com/calendar/render?action=TEMPLATE&text=...&dates=YYYYMMDD/YYYYMMDD+1&details=...` (all-day event, end date exclusive)

---

## Section 7: Your Log Setup

**Log structure:** monolithic `log.md` at vault root, append-only.
Format: `- [YYYY-MM-DD] action | Description`

**To migrate to per-day logs:** `python scripts/migrate_log.py --vault <path>`
**To refresh stats:** `python scripts/vault_stats.py --vault <path>`

---

## Section 8: Agent Memory

### Key Preferences
- Notes in English with LaTeX where the content is mathematical
- Course-driven organization - always ask "which course does this belong to?"
- Exam-oriented: keep quiz/definition/glossary material, don't pad with filler

### Recurring Patterns
- New lecture material arrives in `raw/` (often Italian, often NotebookLM exports) ->
  translate, split into topic notes, file under the right `Courses/<course>/`, update
  `index.md` + `log.md`
- New image dropped in `raw/` AND referenced from any note text -> move it to `maps/`
  (images are exempt from raw-immutability once cited) and replace any markdown/embed
  links with the plain-text form `(see -> maps/<filename>)`

### Known Gotchas
- `raw/` files have **no frontmatter** - they are verbatim user imports; do not "fix" them
- The NotebookLM mind map PNG (now in `maps/`) contains topics not present in the
  accompanying markdown (e.g. evaluation metrics) - it is a legitimate source; cite it
  as `(see -> maps/NotebookLM Mind Map.png)` when used
- `scripts/vault_health.py` regex-scans code blocks / inline code (so schema examples
  like Note Name count as broken links). Judge findings against this file before
  "fixing" anything. Image references are plain text `(see -> maps/...)`, invisible
  to the checker by design.
- An empty stub `source note.md` appeared at vault root on 2026-10-09 (0 bytes, traced
  to the once-unbackticked source-note example in this file); safe to delete if
  still present

---

*Generated on: 2026-10-09*
*Updated: 2026-10-15 - footer citations (Sources and Recency) + Graph Linking Policy*
*By: obsidian-second-brain skill*
