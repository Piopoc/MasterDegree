---
type: course-note
date: 2026-11-17
tags:
  - course
  - advanced-text-analytics
  - notebook
  - legal-nlp
  - regex
  - spacy
  - gliner
  - rdflib
  - sparql
  - skos
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L14_notebook_Part1]]"
  - "[[ATA_L14_notebook_Part2]]"
ai-first: true
---

## For Future Agent

Notebook companion for Lecture 14, covering both notebooks: **Part I** `ATA_L14_Legal_NER_companion.ipynb` (hybrid legal NER on one hand-crafted EU-law sentence: regexes for ECLI / case numbers / acts / provisions with named-group parsing, `EntityRuler` gazetteer, merge policy, toy role assigner, nested references in `doc.spans`, ELI-like URIs, strict evaluation and error taxonomy) and **Part II** `ATA_L14_Realistic_Legal_KG_GLiNER_ELI_EuroVoc.ipynb` (from extracted entities to an `rdflib` knowledge graph: ELI/CELEX registry, ELI ontology, local SKOS vocabulary, GLiNER for actors, dictionary topic spotting and its failure modes, Turtle, SPARQL, CELLAR/EuroVoc, LOMO). Extracted 2026-11-17 from `raw/ATA_L14_notebook_Part1.md` and `raw/ATA_L14_notebook_Part2.md`. Central caveats: Part I's P = R = F1 = 1.0 is measured on the very sentence the rules were written for; Part I's URI builder drops the point `(f)`; in Part II most graph elements are **hard-coded**, only act grounding and topic annotation are text-driven, and the topic normalizer maps the key phrase to the wrong concept. Theory note: [[Legal Information Extraction and Knowledge Graphs]].

---

# Part I - Hybrid Legal NER: Citations, Provisions, Parties, Roles

## 1. The Claim and the Running Example

One methodological claim: **legal IE is naturally hybrid**. Rules, curated resources and learned models each fit a different object class; a good system routes each class to the right component.

```text
In Case C-311/18, the Court of Justice dismissed the appeal brought by Smith
against the Ministry of Justice under Article 6(1)(f) of Regulation (EU) 2016/679.
The judgment is identified as ECLI:EU:C:2020:559.
```

The sentence contains every object class. It mixes **real identifiers** (`C-311/18` and `ECLI:EU:C:2020:559` are the real *Schrems II* judgment, 16 July 2020) with **fictitious procedure**: Schrems II was a **preliminary reference** from the Irish High Court, not an appeal, and its parties were the Data Protection Commissioner, Facebook Ireland and Maximillian Schrems. Lesson: in a preliminary reference the roles are "applicant / defendant *in the main proceedings*", not "appellant / respondent" - **the role inventory itself depends on the procedure type**.

### 1.1 Spectrum of Formality

| Object class | Example | Surface regularity | Technique |
|---|---|---|---|
| Case-law identifier (ECLI) | `ECLI:EU:C:2020:559` | very high, standardized | regex + parser |
| Case number | `Case C-311/18` | high, court convention | regex |
| Legislative act | `Regulation (EU) 2016/679` | high, varies over time | regex + registry |
| Provision reference | `Article 6(1)(f)` | high, hierarchical | regex with named groups |
| Court / institution | `Court of Justice` | closed multi-word set | gazetteer (`EntityRuler`) |
| Person / organization | `Smith`, `Facebook Ireland` | open, low regularity | statistical / generalist NER |
| Procedural role | *appellant*, *respondent* | not a surface property | contextual classification / RE |

An ECLI is a piece of a formal language embedded in prose; a role is not on the string at all. Training a neural model to find ECLIs spends data and capacity on what a 60-character regex does deterministically - and the regex is **auditable**.

## 2. The Regex Layer

### 2.1 ECLI

```python
ECLI_PATTERN = re.compile(r"\bECLI:[A-Z]{2}:[A-Z0-9]+:\d{4}:[A-Z0-9.]+\b")
```

```text
ECLI : <country or EU> : <court code> : <year> : <ordinal>
 (1)        (2)              (3)          (4)       (5)
```

`[A-Z]{2}` country (`EU`, `IT`), `[A-Z0-9]+` court (`C` = Court of Justice, `T` = General Court), `\d{4}` year, `[A-Z0-9.]+` ordinal (national ECLIs may contain dots).

**The trailing `\b` subtlety.** The text ends `...:2020:559.` The class allows `.`, so the greedy `+` first eats `559.`; `\b` then fails (`.` and end-of-string are both non-word); the engine backtracks one char and `\b` matches between `9` and `.`. Result is correct **by accident of backtracking**. Explicit version that forbids a trailing dot:

```python
r"\bECLI:[A-Z]{2}:[A-Z0-9]+:\d{4}:[A-Z0-9]+(?:\.[A-Z0-9]+)*\b"
```

Legal punctuation (dots, slashes, parentheses) is exactly where naive patterns break.

### 2.2 Case Numbers

```python
CASE_PATTERN = re.compile(r"\bCase\s+[A-Z]-\d+/\d+\b")
```

Form `<court letter>-<serial>/<two-digit year>`. Two design decisions:

1. The keyword `Case` is **inside the span** - an annotation-guideline choice. It must match the gold standard, otherwise strict evaluation counts a boundary error.
2. Joined cases (`Joined Cases C-293/12 and C-594/12`) and appeal suffixes (`C-123/21 P`) are not covered - fine for teaching, not for production.

### 2.3 Legislative Acts

```python
EU_ACT_PATTERN = re.compile(
    r"\b(?:Regulation|Directive|Decision)\s+\((?:EU|EC|EEC)\)\s+\d{4}/\d+\b"
)
```

Encodes the **post-2015 unified numbering** `Type (Domain) YYYY/N`. Older conventions differ:

| Period / type | Typical form |
|---|---|
| post-2015, all acts | `Regulation (EU) 2016/679`, `Directive (EU) 2019/790` |
| pre-2015 directives | `Directive 95/46/EC` (year/number/domain, no parentheses) |
| pre-2015 regulations | `Regulation (EC) No 45/2001` (number/year, with `No`) |

High precision on modern text, recall collapses on historical text: the notebook's `Directive (EC) 95/46` test is not matched (`\d{4}` needs a four-digit year) - and the real canonical form `Directive 95/46/EC` would not match either. This is **temporal distribution shift**: the extractor fails *silently* on older drafting conventions.

### 2.4 Provisions - Recognition and Parsing at Once

```python
PROVISION_PATTERN = re.compile(
    r"\bArticle\s+"
    r"(?P<article>\d+[A-Za-z]?)"
    r"(?:\((?P<paragraph>\d+)\))?"
    r"(?:\((?P<point>[a-z])\))?"
)
```

```text
Article 6(1)(f)  ->  { article: "6", paragraph: "1", point: "f" }
```

Named groups turn a span detector into a shallow **structural parser**.

- `\d+[A-Za-z]?` covers inserted articles (`Article 8a`), common in amended law.
- Optional groups are ordered left to right, but `Article 6(f)` would still be accepted with `(f)` in the *point* slot and no paragraph - correctness depends on drafting conventions.
- Not covered: ranges (`Articles 12 to 14`), enumerations (`Article 6(1)(a) and (f)`), sub-points (`(i)`), "first subparagraph". Enumerations raise **coordination ellipsis**: `(f)` in `6(1)(a) and (f)` means `6(1)(f)`, so the extractor must reconstruct text that is not there.

## 3. Recognition vs Parsing

```text
RECOGNITION                          PARSING
"there is an ECLI at chars 151-169"  "country=EU, court=C, year=2020, ordinal=559"
```

`parse_ecli` splits on `:` and **validates**: exactly five parts and the literal prefix `ECLI`. The two steps fail **independently** (right span / wrong fields, or right parser / wrong boundaries), so they need **separate metrics** (Section 9). Parsing turns a string into a **record**, and records are what can be linked to resources.

## 4. A Common Output Schema

`regex_entities` normalizes every extractor's output to:

```python
{"text": ..., "label": ..., "start": ..., "end": ..., "source": "regex"}
```

This schema is the **integration contract** of the hybrid system:

- **Character offsets, not token indices** - independent of tokenization, so a regex, a spaCy pipeline and a subword transformer can be compared and merged (offsets are the lingua franca of span annotation: brat standoff, W3C Web Annotation).
- **`source`** = provenance, needed for debugging, confidence calibration and legal auditability ("why does the system think this is a provision?").
- Output **sorted by `(start, end)`** - deterministic order simplifies merging and overlap detection.

## 5. Gazetteer with spaCy `EntityRuler`

```python
nlp = spacy.blank("en")
ruler = nlp.add_pipe("entity_ruler", config={"overwrite_ents": True})
ruler.add_patterns([
    {"label": "COURT",        "pattern": "Court of Justice",    "id": "COURT_CJEU"},
    {"label": "ORGANIZATION", "pattern": "Ministry of Justice", "id": "ORG_MINISTRY_JUSTICE"},
    {"label": "PERSON",       "pattern": "Smith",               "id": "PERSON_SMITH"},
])
```

1. `spacy.blank("en")` = tokenizer only (no tagger, parser, NER) - the ruler is the *only* source of `doc.ents`, isolating its contribution.
2. String patterns are **phrase patterns** matched by a `PhraseMatcher`: fast (about linear in document length regardless of gazetteer size) but exact-token. Lowercase `court of justice` will not match unless `phrase_matcher_attr: "LOWER"`.
3. `overwrite_ents=True` lets ruler matches override earlier components. No effect here; with a statistical `ner` in the pipeline, ruler position + this flag define the **precedence policy** between rules and model.
4. `id` fills `ent.ent_id_` - a **minimal normalization layer**: `Court of Justice`, `CJEU`, `the Court` can share one ID, the first step towards entity linking.

A gazetteer suits **closed, slowly changing, authoritative sets** (institutions, supreme courts, agencies). Putting `Smith` in it is a teaching shortcut; persons belong to learned NER.

## 6. Merging Sources

The notebook just concatenates and sorts - it works only because regex and ruler spans are **disjoint**. A real merge policy answers three questions:

| Case | Example | Policy |
|---|---|---|
| exact duplicate | same span + label from two sources | keep one, union provenance |
| label conflict on same span | `Court of Justice` = `ORG` (model) vs `COURT` (gazetteer) | source priority, specificity, or confidence |
| overlapping non-identical spans | `Article 6(1)(f)` vs `Article 6(1)(f) of Regulation (EU) 2016/679` | flat (longest-match / priority, cf. `spacy.util.filter_spans`) or nested (keep both) |

These choices are rarely discussed but strongly affect measured scores.

## 7. Legal Roles Are Contextual

```text
Smith                 type = PERSON         role = APPELLANT
Ministry of Justice   type = ORGANIZATION   role = RESPONDENT
```

The **type** belongs to the referent (Smith is a person in every document). The **role** belongs to a *relation* between the referent and a *proceeding*:

$$
\text{role} : \text{Party} \times \text{Proceeding} \rightarrow \{\text{APPELLANT}, \text{RESPONDENT}, \text{INTERVENER}, \dots\}
$$

It is a binary relation, not a unary predicate over spans - hence "role assignment is already close to relation extraction". The proceeding is implicit here (one case per document) but must become explicit as soon as a judgment cites other cases.

### 7.1 The Toy Role Assigner

```python
("APPELLANT",  re.compile(r"appeal\s+brought\s+by\s+(?P<name>[A-Z][a-z]+)")),
("RESPONDENT", re.compile(r"against\s+the\s+(?P<name>Ministry of Justice)")),
```

**Lexico-syntactic trigger patterns** (Hearst-style): the cue phrase marks the relation, the group captures the argument. Weaknesses:

- appellant: `[A-Z][a-z]+` captures a single capitalized token - truncates `Maximillian Schrems`, misses `McDonald's`: a **boundary error by construction**;
- respondent: the name is hard-coded - a gazetteer entry disguised as a contextual rule.

Better design: anchor the role argument to an **already recognized entity** (spaCy `Matcher` requiring `ENT_TYPE` in `{PERSON, ORGANIZATION}` after the trigger, or `DependencyMatcher` along `brought -agent-> by -pobj-> X`). Recognition decides *what* the entity is; role assignment decides *what it does in this case* - cleanly factorized.

### 7.2 Joining Type and Role

```python
typed_by_span = {(ent["start"], ent["end"]): ent for ent in spacy_result}
```

An exact `(start, end)` join works only because both components produce identical spans. If one includes the determiner (`the Ministry of Justice`) the join silently returns `None`. Robust alternatives: **overlap-based joins** (max character overlap, IoU threshold) or, better, the entity-anchored design above (no join needed).

## 8. Nested References

```text
Article 6(1)(f) of Regulation (EU) 2016/679
└─────┬──────┘    └──────────┬───────────┘
   PROVISION              LEGAL_ACT
└───────────────────┬──────────────────────┘
              LEGAL_REFERENCE
```

All three spans are legitimate: the provision, the act, and the full reference that fixes *which* Article 6. Legal references are **compositional**, built with connectives (*of*, *in conjunction with*, *as amended by*).

### 8.1 Why `doc.ents` Cannot Hold Them

`doc.ents` is a **flat, non-overlapping segmentation**: each token has one IOB tag, so it can belong to one entity only - the representational limit of sequence labeling (BIO/BILOU + CRF/softmax, Lecture 12). Literature solutions:

1. **Span-based NER** - enumerate spans up to a max length, classify each; overlaps allowed by construction (GLiNER belongs here).
2. **Layered sequence labeling** - one tagging layer per nesting level.
3. **Hypergraph / parsing-based** - nested mentions as tree constituents.

### 8.2 `doc.spans` and `SpanGroup`

```python
nested_doc.spans["legal_refs"] = legal_spans
```

`doc.spans` is a dict of named `SpanGroup`s that may **overlap arbitrarily**, and it is the training format of spaCy's `SpanCategorizer` (`spancat`) - so rule output can directly become training data for a learned nested model.

**`alignment_mode` caveat.** `Doc.char_span` maps char offsets to tokens. `"strict"` (default) returns `None` if offsets are not on token boundaries; `"contract"` shrinks to tokens fully inside; `"expand"` widens to the enclosing tokens. spaCy's English tokenizer handles parentheses with affix rules and may split `6(1)(f)` at positions that differ from the regex boundaries. `"expand"` is robust **but can silently enlarge a span** - a classic, hard-to-see source of evaluation noise. Always inspect `[t.text for t in nested_doc]`.

## 9. From Records to Identifiers

```text
"Article 6(1)(f) of Regulation (EU) 2016/679"
      | parse_legal_reference
{article: 6, paragraph: 1, point: f, legal_act: "Regulation (EU) 2016/679"}
      | toy_eli_from_reference
https://eur-lex.europa.eu/eli/reg/2016/679/art_6/par_1/oj
```

The legal analogue of biomedical normalization, with one key difference: in biomedicine normalization is **retrieval** over a synonym lexicon ([[Biomedical Named Entity Recognition and Concept Normalization]]); in law much of it is **construction**, because the identifier scheme is compositional and deterministic.

Three critical remarks:

1. **The point `(f)` is dropped** - only `art_` and `par_` segments are added, so `6(1)(a)` and `6(1)(f)` get the **same URI**. Information lost - and span-level F1 cannot see it.
2. **URI form** - the canonical GDPR ELI is `http://data.europa.eu/eli/reg/2016/679/oj`; `eur-lex.europa.eu/eli/...` is a presentation URL. Subdivision URIs are not guaranteed to resolve: treat a constructed URI as a **key**, not a dereferenceable link, unless checked against the resolver.
3. **Coverage** - only `Regulation (EU)`; directives (`dir`) and decisions (`dec`) need their own branches.

## 10. Evaluation

### 10.1 Strict Entity-Level P/R/F1

Correct only if **start, end and label** all match ($G$, $\hat{Y}$ = sets of `(start, end, label)` triples; CoNLL convention, `seqeval` strict mode):

$$
P = \frac{|G \cap \hat{Y}|}{|\hat{Y}|}, \qquad R = \frac{|G \cap \hat{Y}|}{|G|}, \qquad F_1 = \frac{2PR}{P+R}
$$

The regex system scores $P = R = F_1 = 1.0$ (4 gold, 4 predicted, all exact). **Read it critically**: the rules were written looking at this sentence and evaluated on it - a development-set measurement, not a generalization estimate. A credible protocol needs held-out data from other periods, courts and drafting traditions.

- **Strictness is a choice.** Predicting `C-311/18` against gold `Case C-311/18` counts as **one FP and one FN**. Relaxed variants give partial credit (overlap + type, exact boundary any type, SemEval-2013 Task 9's strict / exact / partial / type). In legal IE, where linking needs exact boundaries, strict is the right primary metric; relaxed ones are for diagnosis (see [[Annotation and Evaluation of Information Extraction]]).
- **Nested gold.** A flat system evaluated against a gold standard with `LEGAL_REFERENCE` spans is penalized by construction - the evaluation must match the representation.

### 10.2 Field-Level Parsing Accuracy

Notebook Section 19 scores article / paragraph / point separately - recognition and parsing have separate metrics. Better: **exact-record accuracy** (share of references with *all* fields right), since one wrong field (paragraph 2 instead of 1) changes the legal meaning entirely and per-field averages hide it.

## 11. Error Taxonomy and Remedies

| Error class | Example | Diagnostic signal | Remedy |
|---|---|---|---|
| **Boundary** | `Regulation` vs `Regulation (EU) 2016/679` | overlap but not exact | extend patterns; revise guidelines |
| **Type** | `Court of Justice` -> `ORGANIZATION` | exact span, wrong label | gazetteer precedence; finer labels |
| **Role** | appellant tagged respondent | right entity, wrong relation | better triggers; dependency rules; supervised RE |
| **Citation parsing** | fields wrong though string correct | right span, wrong fields | fix grammar / named groups; field tests |
| **Linking** | right act, wrong URI | right record, wrong ID | registry lookup; resolver validation |
| **Nested-entity** | only the outer span found | missing layer | span representation (`doc.spans`, `spancat`) |

The notebook labels `Article 6(2)(f)` vs gold `6(1)(f)` as a *parsing* error. Strictly, if the **predicted string** differs from the text, the fault is recognition (or OCR); a true parsing error keeps the string and mis-assigns a field. Not pedantic: **the remedy depends on the category**.

## 12. Optional GLiNER Component

GLiNER (Zaratiana et al., NAACL 2024): **span-based, label-conditioned** NER. A bidirectional encoder jointly encodes the text and natural-language label names (`"court"`, `"legal provision"`); each span is scored against each label embedding, threshold 0.5.

- **Labels at inference time** - no retraining to ask for `"legal act"`; suits jurisdiction-dependent inventories.
- **Span-based** - overlaps possible in principle, but **default decoding applies a greedy non-overlap constraint** unless configured otherwise.

Which component for which class:

- identifiers / citations -> regex (GLiNER finds the span but cannot *parse* it; a threshold adds uncertainty where none is needed);
- courts / institutions -> gazetteer (reliability + `ent_id_`);
- persons / organizations -> learned NER (unseen names);
- roles -> neither alone: asking GLiNER for `"appellant"` conflates type and relation; the model has no notion of the proceeding.

## 13. Reference Architecture (Part I)

```text
legal document
   ├── regex / grammars ──────────► ECLI, case nos., act nos., provisions
   │                                   └─► structural parsing (named groups)
   ├── gazetteers / EntityRuler ──► courts, institutions (+ resource IDs)
   ├── learned / generalist NER ──► persons, organisations, open categories
   ▼
 span integration (offsets, provenance, merge policy, flat vs nested)
   ▼
 role assignment (relation to the proceeding)
   ▼
 normalisation (records -> identifiers -> URIs)
   ▼
 structured legal information ──► Part II: knowledge graph
```

## 14. Part I Exercises (Condensed)

1. Robust ECLI regex without relying on backtracking; test `ECLI:NL:HR:2019:1234`, `ECLI:IT:CASS:2020:12345CIV`.
2. Extend the act pattern to `Directive 95/46/EC` and `Regulation (EC) No 45/2001`, parse to `{type, domain, year, number}` (number/year order differs).
3. `Article 6(1)(a) and (f)` -> two full records (coordination ellipsis).
4. Entity-anchored role assigner with spaCy `Matcher`; test on paraphrases ("Smith's appeal against the Ministry").
5. Compare `strict` / `contract` / `expand` alignment on all nested spans.
6. 20-sentence held-out EUR-Lex test set; strict + partial P/R/F1 per label; classify each error by Section 11.
7. Lossless `toy_eli_from_reference` (no collisions; directives and decisions).

---

# Part II - From Legal Entities to a Legal Knowledge Graph

## 15. Goal and Scenario

Part I asked *how to find and parse* spans; Part II asks **what the output should become**: a small RDF legal KG on standard vocabularies.

```text
text -> spans -> normalised resources -> typed relations -> queryable graph
```

```text
In Case C-311/18, the Court of Justice examined a dispute involving Maximillian
Schrems and Facebook Ireland. The Court interpreted Article 46(1) of Regulation
(EU) 2016/679 in relation to transfers of personal data to third countries and
the requirement to provide appropriate safeguards.
```

Now the content is realistic: Schrems II examined transfers to third countries on the basis of **appropriate safeguards** (Article 46 GDPR, notably standard contractual clauses).

| Kind | Instances | Authority behind the identifiers |
|---|---|---|
| Legislation | GDPR, Article 46(1) | registries (ELI, CELEX) |
| Case law | Case C-311/18 | court registries (case number, ECLI) |
| Actors | Court of Justice, Schrems, Facebook Ireland | authority tables / NER |
| Subject matter | data transfers, third countries, safeguards | controlled vocabularies (EuroVoc) |

Designing a legal KG largely means deciding **whose identifiers to reuse for what**.

## 16. Registry Lookup: ELI and CELEX

```python
LEGAL_RESOURCE_REGISTRY = {
    "Regulation (EU) 2016/679": {
        "title": "General Data Protection Regulation",
        "eli_uri": "http://data.europa.eu/eli/reg/2016/679/oj",
        "celex": "32016R0679", ...
    }
}
```

- **ELI** `http://data.europa.eu/eli/reg/2016/679/oj` - semantic HTTP URI: type `reg`, year, number, Official Journal version `oj`.
- **CELEX** `32016R0679` - sector `3` (legislation), year `2016`, type `R` (regulation), number `0679`.

This is **dictionary-based entity linking** keyed by surface string. It works only because the regex already produced the canonical form: **linker recall is bounded by the canonicalizing power of the recognizer**. "the GDPR" or "Regulation 2016/679" would miss. A realistic linker first normalizes to a structured key `{type: reg, year: 2016, number: 679}` and resolves the key, not the string.

## 17. The ELI Ontology

Namespace `http://data.europa.eu/eli/ontology#`, built on **FRBR** levels:

| FRBR level | ELI class | Meaning in law |
|---|---|---|
| Work | `eli:LegalResource` | the act as an abstract legal object (the GDPR) |
| Expression | `eli:LegalExpression` | a linguistic version (the English GDPR) |
| Manifestation | `eli:Format` | a format (PDF, HTML, Formex XML) |

`eli:LegalResourceSubdivision` (subclass of `LegalResource`) models articles, paragraphs, annexes, chained to the whole with `eli:is_part_of` (inverse `eli:has_part`) - a **mereological hierarchy** that mirrors the provision parser's output:

```text
GDPR            eli:LegalResource
  ▲ is_part_of
Article 46      eli:LegalResourceSubdivision
  ▲ is_part_of
Article 46(1)   eli:LegalResourceSubdivision
```

`eli:is_about` links a resource to its subject matter (`skos:Concept`) - where ELI meets EuroVoc.

### 17.1 Modeling Choices in the Notebook (and Their Limits)

1. **Real URI for the act, local URI for the provision** (`https://example.org/legal-resource/gdpr/article/46/paragraph/1`). Methodologically honest: **never mint URIs in a namespace you do not control**. The node is still typed `eli:LegalResourceSubdivision`, so it stays interoperable.
2. **Flattened hierarchy** - 46(1) links directly to the GDPR, no `Article 46` node. You lose "which paragraphs of Article 46 are interpreted?". Faithful model: one node per parse level, chained with `is_part_of`.
3. **One direction only** - `eli:has_part` is never materialized, so without a reasoner SPARQL over `has_part` returns nothing. Materialize both (as done for `broader`/`narrower`) or use `^eli:is_part_of`.
4. **`dcterms:title` on the work** - in ELI titles are language-specific, so they belong to the **expression** (`eli:title`, domain `eli:LegalExpression`): titles differ across the 24 official languages, the work does not. Local IDs would idiomatically be `eli:id_local`.

## 18. Local SKOS Vocabulary

| Construct | Role | Example |
|---|---|---|
| `skos:Concept` | unit of meaning | *international transfer of personal data* |
| `skos:prefLabel` | canonical label, at most one per language | `"international transfer of personal data"@en` |
| `skos:altLabel` | synonyms, variants | `"cross-border data transfer"@en` |
| `skos:broader` / `skos:narrower` | hierarchy, mutual inverses | *data transfer* -> *data protection* |

SKOS is deliberately weaker than OWL: `skos:broader` is **not** `rdfs:subClassOf` ("more general topic", not set inclusion). EuroVoc is published in SKOS; the notebook builds a **five-concept stand-in** under `example.org` and says plainly these are not EuroVoc URIs.

```text
fundamental rights
   └── protection of personal data
          └── international transfer of personal data
                 └── appropriate safeguards
third country                         (no broader concept)
```

Putting *appropriate safeguards* under *international transfer* is a GDPR-specific choice (Chapter V transfer mechanism); safeguards appear in many regimes. **A vocabulary encodes a point of view** - the hierarchy is an engineering artifact, not a fact.

`broader_chain` walks upward with a `visited` set (SKOS does not forbid cycles) but takes `broader[0]` when there are several parents - silently ignoring **poly-hierarchy**, which EuroVoc has. Fix: return the ancestor DAG, or just use `skos:broader+` in SPARQL.

## 19. Two Extraction Regimes

**Deterministic citations.** Same regexes as Part I (act pattern narrowed to `Regulation (EU)`). Parsed provision: `{article: "46", paragraph: "1", point: None}` - the input that *should* build the provision URI (but see Section 22).

**GLiNER for actors.** Labels `court`, `person`, `organization`, `legal topic`, `legal action`; threshold 0.45. GLiNER is **not** asked to replace the citation regexes: for open actor sets a label-conditioned span model is right; for citations the useful output is the parse and the identifier, and a probabilistic score only adds boundary risk.

### 19.1 The Empty "legal topic" Label

GLiNER returned **no** `legal topic` entities, so the original topic pipeline produced an empty table. Instructive, not accidental:

1. *Legal topic* is an **abstract, non-referential** category. GLiNER's training data is mostly named/concrete mentions; "transfers of personal data to third countries" **describes** a theme rather than **refers** to an individual.
2. Label-conditioned models are **sensitive to label wording** (`"legal concept"`, `"area of law"`...) - label engineering must be evaluated, not tuned by eye.
3. Subject indexing is really **document/passage-level multi-label classification** into EuroVoc descriptors (EURLEX57K, MultiEURLEX benchmarks), not span extraction.

Repair: `find_topic_mentions` - exact occurrences of any SKOS label in the text, i.e. **dictionary-based concept spotting** (same principle as `EntityRuler`, driven by the thesaurus). Independent of GLiNER recall, but with its own limits.

## 20. Topic Normalization - Critical Analysis

### 20.1 The Scoring Function

For mention $s$ and concept $c$ with labels $L(c)$, with $W(\cdot)$ = set of lowercased word tokens:

$$
\text{score}(s, c) = \max_{\ell \in L(c)} \frac{|W(\ell) \cap W(s)|}{|W(\ell)|}
$$

i.e. the **coverage of the label by the mention**; the highest-scoring concept wins.

### 20.2 What Actually Happens

- **Spotting** returns `["third countries", "appropriate safeguards", "safeguards"]`. The phrase carrying the main content, "transfers of personal data", matches **no** label (plural *transfers*, missing *international*, word order).
- **Normalization** maps each to *third country* or *appropriate safeguards* (score 1.0); *appropriate safeguards* appears twice (full label + alt-label "safeguards") - harmless in RDF (a graph is a set) but redundant in the table.
- **Consequence**: the graph asserts `Article 46(1) eli:is_about <third-country>` and `<appropriate-safeguards>` but **not** `<data-transfer>`. The concept the notebook's narrative highlights appears only indirectly, as the broader of *appropriate safeguards*. Lesson: always check the **gap between the story in the markdown and the triples the code produces**.

### 20.3 The Original Fallback Would Have Been Worse

Scoring the mention "transfers of personal data to third countries":

| Concept | Best label | Overlap | Score |
|---|---|---|---|
| third country | "third countries" | {third, countries} | **1.00** |
| protection of personal data | "protection of personal data" | {of, personal, data} | 0.75 |
| international transfer of personal data | "international transfer of personal data" | {of, personal, data} | 0.60 |

It picks the **modifier** (*third country*), not the **head** (*transfer*). Three weaknesses at once:

1. **Short labels win** - the measure is asymmetric and ignores how much of the mention is unexplained. Fix: symmetric measures (Jaccard, Dice, F-measure of label and mention coverage).
2. **No morphology** - *transfers* != *transfer*. Fix: lemmatization / stemming.
3. **Stop words count** - *of* inflates every multi-word label. Fix: stop-word filter or IDF weighting.

### 20.4 No Rejection Threshold

`normalize_topic` always returns something: "Court of Justice" -> *protection of personal data* with score 0.25, purely because of *of*. A normalizer must be able to **abstain** (the **NIL problem** in entity linking): calibrated threshold, below it no link or human review.

### 20.5 Better Approaches (Increasing Sophistication)

1. **Principled lexical**: lemmatize, drop stop words, IDF over the thesaurus, symmetric similarity, rejection threshold.
2. **Dense retrieval**: embed mentions and all SKOS labels (pref + alt, all languages), top-$k$ by cosine, optional cross-encoder rerank (BLINK; SapBERT in biomedicine).
3. **Passage-level indexing**: multi-label classification of the provision into EuroVoc descriptors, trained on already-indexed EUR-Lex documents.

### 20.6 Topic Indexing vs Biomedical Normalization

| | Biomedical normalization | Legal topic indexing |
|---|---|---|
| Question | *What does this mention refer to?* | *What is this resource about?* |
| Linked object | the **mention** (span) | the **resource** (provision, judgment) |
| Relation | identity / reference (`mention -> CUI`) | aboutness (`eli:is_about`) |
| Cardinality | usually one concept per mention | several concepts per resource |
| Evidence | local (span + context) | global (whole passage) |

So the subject of `eli:is_about` is the **provision**; the span is only *evidence*. A complete model records that evidence as provenance (W3C Web Annotation or PROV-O: which span, which component, which score).

## 21. Building the Graph

**Vocabulary reuse + local extension**: `eli:` for legislation, `skos:` for topics, `dcterms:` for identifiers/titles, `rdfs:label` for display, local `ex:` for what ELI lacks (it is a *legislation* ontology - no cases, courts, parties). Production candidates to replace `ex:`: the **ECLI** metadata model, EU **authority tables** (the Court of Justice has one), the **CDM** (Common Data Model behind CELLAR), **schema.org / FOAF / W3C ORG**.

```text
ex:case/C-311-18  a ex:LegalCase ;
    dcterms:identifier "C-311/18" ;
    ex:hasCourt   ex:court/CJEU ;
    ex:hasParty   ex:person/maximillian-schrems , ex:organization/facebook-ireland ;
    ex:interprets <.../gdpr/article/46/paragraph/1> ;
    ex:concerns   <http://data.europa.eu/eli/reg/2016/679/oj> .

<.../gdpr/article/46/paragraph/1>  a eli:LegalResourceSubdivision ;
    eli:is_part_of <http://data.europa.eu/eli/reg/2016/679/oj> ;
    dcterms:identifier "Article 46(1)" ;
    eli:is_about <.../third-country> , <.../appropriate-safeguards> .
```

Links back to Part I:

1. **`ex:hasParty` has no role.** Model roles either as role-specific properties (`ex:hasApplicant`, `ex:hasDefendant`) or by **reifying** the participation as an n-ary relation - the standard RDF qualified-relation pattern, more flexible:

```turtle
ex:case ex:hasParticipation [ ex:party ex:schrems ; ex:role ex:Applicant ] .
```

2. **`ex:interprets` is a genuinely legal relation** - not an NER label, it comes from the verb *interpreted*. Populating it automatically is **relation extraction** (next lectures).

Serialization `g.serialize(format="turtle")`: read it and check that **every triple can be traced** to its origin (text, regex, registry, vocabulary, or hand-written code).

## 22. Extracted vs Hand-Wired

| Graph element | Origin |
|---|---|
| GDPR URI, title, CELEX | registry lookup driven by the regex match (text-driven) |
| Provision URI | **hard-coded** (not built from `parsed_provision`) |
| Provision identifier `"Article 46(1)"` | **hard-coded** literal |
| Case URI and identifier | **hard-coded** (not from `CASE_PATTERN`) |
| Court, persons, organizations | **hard-coded** (GLiNER output unused) |
| `hasParty`, `hasCourt`, `interprets`, `concerns` | **hard-coded** |
| `eli:is_about` topics | spotting + normalization pipeline (text-driven) |

Only **two links** of the chain are driven by text: act grounding and topic annotation. To become a real *text-to-graph* system: mint URIs deterministically from parsed records (case-number slug; article/paragraph/point path), create actor nodes from GLiNER spans with an **entity-resolution** step, produce relations with an RE component carrying provenance and confidence. This is the gap between a **KG schema populated by hand** and a **KG population pipeline**.

## 23. SPARQL

### 23.1 Join Query (Notebook Section 20)

```sparql
SELECT ?caseId ?provisionId ?topicLabel WHERE {
  ?case a ex:LegalCase ; dcterms:identifier ?caseId ; ex:interprets ?provision .
  ?provision dcterms:identifier ?provisionId ; eli:is_about ?topic .
  ?topic skos:prefLabel ?topicLabel .
}
```

Joins **case law + legislation + vocabulary** - impossible over a flat span list. Returns two rows (*third country*, *appropriate safeguards*), for the reasons in Section 20.2.

### 23.2 Hierarchy Query (Notebook Section 21)

`OPTIONAL { ?topic skos:broader ?broader . }` returns the immediate parent or `None` (*third country*). Two extensions:

```sparql
# all ancestors at any depth (SPARQL 1.1 property path)
?topic skos:broader+ ?ancestor .

# hierarchical query expansion: provisions about data protection or anything narrower
?provision eli:is_about/skos:broader* <https://example.org/legal-topic/data-protection> .
```

The second is **hierarchical query expansion**: a provision indexed only with *appropriate safeguards* is found by a search for *data protection* - the practical payoff of SKOS, and how EuroVoc search in EUR-Lex works.

## 24. Live EuroVoc via CELLAR

CELLAR is the Publications Office's semantic repository; public SPARQL endpoint `https://publications.europa.eu/webapi/rdf/sparql`. The optional query looks for concepts whose English `prefLabel` contains *personal data*. Improvements:

1. **Scope to the scheme** (`?concept skos:inScheme <EuroVoc scheme>`) - otherwise every `skos:Concept` of every vocabulary is scanned: slow, timeouts, non-EuroVoc hits.
2. **Search alt-labels** (`skos:prefLabel|skos:altLabel`) - EuroVoc entry terms are a large part of its lexical coverage.
3. **`CONTAINS(LCASE(...))` is a full scan** - Virtuoso's `bif:contains` full-text index is much faster but not portable.
4. **Reproducibility** - results change with EuroVoc versions: pin the version or cache the response.

This is where the local vocabulary gets **aligned** to EuroVoc: replace `example.org` URIs, or assert `skos:exactMatch` / `skos:closeMatch` (keeps local concepts EuroVoc lacks while staying interoperable).

## 25. Beyond NER: LOMO

The **Legal Obligation Metadata Ontology (LOMO)** models interpreted normative content - obligations, permissions, prohibitions, rights, definitions, scope provisions - linked to ELI sources. Research programme:

```text
NER / provision extraction             (what is cited?)
        ↓
relation extraction                    (who interprets / amends / applies what?)
        ↓
normative rule extraction              (what is required, of whom, under what conditions?)
        ↓
formal legal knowledge representation  (deontic modelling, compliance checking)
```

Each step moves from surface to meaning and adds legally **contestable interpretation** - so human-in-the-loop validation and provenance matter *more* as the pipeline matures. Roadmap items (notebook Section 25): authoritative EuroVoc URIs, authority tables, ECLI normalization, CELEX/ELI resolution, multilinguality, `eli:cites` / `amends` / `repeals` / `based_on`, and above all **temporal versioning**: Article *n* "as amended on date *d*" is a different expression; a KG ignoring time makes wrong legal statements.

## 26. Synthesis (Part II)

```text
                    LEGAL TEXT
        ┌────────────────┴────────────────┐
  deterministic rules                GLiNER / learned NER
  case no. · act · provision         court · person · organisation
        │                      dictionary concept spotting (SKOS labels)
        └───────────────┬─────────────────┘
                  normalisation / linking
          ┌─────────────┴─────────────┐
     ELI / CELEX registry         SKOS / EuroVoc
          └─────────────┬─────────────┘
              relation assertion (-> relation extraction)
                        │
              RDF LEGAL KNOWLEDGE GRAPH ──► SPARQL
```

## 27. Part II Exercises (Condensed)

1. Build provision URIs from `parsed_provision`, one node per level, both `is_part_of` and `has_part`; SPARQL for all GDPR subdivisions at any depth.
2. Re-implement `normalize_topic` (lemmas, stop words, Dice, threshold): the key phrase should map to *international transfer of personal data*, "Court of Justice" should be rejected.
3. Dense SKOS-label retriever vs lexical, top-1 on 30 annotated CJEU mentions.
4. GLiNER label engineering (4 wordings x thresholds 0.3-0.6), recall vs a small gold set.
5. N-ary participation with explicit roles (`Applicant`, `Defendant`, `Intervener`, `Referring court`) + query.
6. CELLAR query scoped to EuroVoc with alt-labels; align local concepts via `exactMatch` / `closeMatch`.
7. PROV-O / Web Annotation provenance for every `is_about` triple; query only score >= 0.8.

## 28. Take-Home Messages (Both Parts)

1. Match technique to formality: rules for identifiers and citations, gazetteers for closed sets, learned models for open sets.
2. Recognition, parsing and normalization are distinct steps with distinct failure modes and metrics.
3. Roles are relations between party and proceeding; in RDF, reify them as n-ary participations.
4. Legal references are compositional and nested: `doc.ents` cannot hold them, `doc.spans` can.
5. Strict evaluation on held-out data, plus an error taxonomy whose categories point to remedies.
6. The output representation is as important as the extractor: a graph makes spans connected and queryable.
7. Reuse authoritative identifiers (ELI, CELEX, ECLI, EuroVoc); never mint URIs in namespaces you do not control.
8. Topic indexing is aboutness, not reference - often better framed as classification.
9. Lexical normalization needs morphology, term weighting, symmetric scoring and a rejection option.
10. Distinguish what the pipeline extracts from what it asserts by hand; closing that gap is relation extraction.

## Related

- [[Legal Information Extraction and Knowledge Graphs]] - Lecture 14 theory note: legal object taxonomy, SKOS, SKOS vs OWL, ELI/ECLI/EuroVoc
- [[spaCy Toy NER - Notebook]] - Lecture 11: blank pipelines, character offsets, `doc.ents`
- [[Advanced Named Entity Recognition - Notebook]] - Lecture 12: `SpanCategorizer`, `doc.spans["sc"]`, GLiNER demo
- [[Biomedical Named Entity Recognition and Concept Normalization - Notebook]] - Lecture 13: retrieval-based normalization, Accuracy@1, bi-/cross-encoders
- [[Knowledge Resources for Text Analytics - Notebook]] - Lecture 2: `PhraseMatcher`, `broader_concepts` traversal
- [[Automatic Term Extraction - Notebook]] - Lecture 15: same GDPR domain, statistical term extraction pipeline
- [[RDF Data Model]] - cross-course bridge ([[Graph Databases]]): triples, Turtle, SPARQL
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L14_notebook_Part1]] (AY 2026/2027)
- [[ATA_L14_notebook_Part2]] (AY 2026/2027)
