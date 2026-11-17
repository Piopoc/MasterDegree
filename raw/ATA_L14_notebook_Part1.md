# Advanced Text Analytics — Lecture 14, Part I
# Hybrid Legal Named Entity Recognition: Citations, Provisions, Parties and Roles

*Lecture notes accompanying the notebook `ATA_L14_Legal_NER_companion.ipynb`*

---

## 0. Purpose and learning objectives

These notes are about one methodological claim. **In the legal domain, information extraction is naturally hybrid.** No single family of techniques covers legal language well. Hand-written rules, curated resources (gazetteers, registries) and learned models each work best on a different class of object. A well-designed system gives each class to the component that suits it.

The companion notebook demonstrates this on one short, hand-crafted EU-law sentence. The sentence is deliberately small, because the subject of study is the *mechanics* of extraction, not benchmark performance.

At the end of this unit, the student should be able to:

1. Explain why legal text contains a large share of *quasi-formal* objects (identifiers, citations, provision references), and why such objects suit deterministic extraction.
2. Distinguish between **recognition** (locating and typing a span) and **parsing** (recovering the internal structure of the span).
3. Explain why **legal roles** (appellant, respondent, …) are *relational, context-dependent* properties rather than intrinsic entity types, and why role assignment is therefore close to relation extraction.
4. Represent **nested (overlapping) legal references**, and explain why the flat `doc.ents` layer of spaCy cannot hold them.
5. Design a strict entity-level evaluation. Evaluate structured parsing separately from span recognition. Categorise errors so that each category points to a specific remedy.

---

## 1. Why legal NER is different

Classical NER inventories (CoNLL-2003: `PER`, `ORG`, `LOC`, `MISC`) were designed for newswire. Legal documents contain these categories too: judges, parties, ministries, courts. But the information that matters most to a lawyer is mostly elsewhere:

| Object class | Example | Surface regularity | Natural technique |
|---|---|---|---|
| Case-law identifier (ECLI) | `ECLI:EU:C:2020:559` | Very high, standardised syntax | Regular expression + parser |
| Case number | `Case C-311/18` | High, court-specific convention | Regular expression |
| Legislative act | `Regulation (EU) 2016/679` | High, but varies over time | Regular expression + registry |
| Provision reference | `Article 6(1)(f)` | High, hierarchical | Regular expression with named groups |
| Court / institution | `Court of Justice` | Closed set, multi-word | Gazetteer (`EntityRuler`) |
| Person / organisation | `Smith`, `Facebook Ireland` | Open set, low regularity | Statistical / generalist NER |
| Procedural role | *appellant*, *respondent* | Not a surface property at all | Contextual classification / relation extraction |

The table shows a **spectrum of formality**. At one end, an ECLI is in effect a piece of a formal language embedded in prose: a grammar can describe it exactly. At the other end, a party's procedural role is not marked on the string at all. It comes from the syntactic and pragmatic context ("the appeal *brought by* X *against* Y").

A sound design principle follows: **match the inductive bias of the component to the formality of the target**. A neural model can learn to recognise ECLIs. But that would spend annotated data and model capacity on something a 60-character regular expression does deterministically and verifiably. On the other side, any rule system will be brittle at recognising unseen party names in free text.

> **Remark (verifiability).** Legal applications value *auditability* highly. A regular expression that extracts `Article 6(1)(f)` can be inspected, versioned and justified. A model prediction often cannot. This is one non-technical reason why rule components survive in legal IE even in the era of large pretrained models.

---

## 2. The running example

```text
In Case C-311/18, the Court of Justice dismissed the appeal brought by Smith
against the Ministry of Justice under Article 6(1)(f) of Regulation (EU) 2016/679.
The judgment is identified as ECLI:EU:C:2020:559.
```

This one sentence contains every object class in the table above. The notebook says plainly that the sentence is *pedagogical*. Students should know that `C-311/18` and `ECLI:EU:C:2020:559` really identify the *Schrems II* judgment of the Court of Justice (16 July 2020). That case was a **preliminary reference** from the Irish High Court, not an appeal, and the parties were the Data Protection Commissioner, Facebook Ireland and Maximillian Schrems. The example thus mixes real identifiers with fictitious procedural content. This makes a useful point: in a preliminary-reference procedure, the parties are "applicant / defendant *in the main proceedings*", not "appellant / respondent". **The role inventory itself depends on the procedure type**, a point we return to in §8.

---

## 3. Deterministic extraction with regular expressions

### 3.1 ECLI

```python
ECLI_PATTERN = re.compile(r"\bECLI:[A-Z]{2}:[A-Z0-9]+:\d{4}:[A-Z0-9.]+\b")
```

The European Case Law Identifier has **five colon-separated components**:

```text
ECLI : <country code or "EU"> : <court code> : <year> : <ordinal number>
 (1)            (2)                  (3)          (4)            (5)
```

The regex mirrors this structure one-to-one:

- `[A-Z]{2}` — the country component (`EU`, `IT`, `DE`, …);
- `[A-Z0-9]+` — the court code (`C` = Court of Justice, `T` = General Court);
- `\d{4}` — the year of decision;
- `[A-Z0-9.]+` — the ordinal number, which may contain dots in national ECLIs.

**A subtle point about the trailing `\b`.** In the example the ECLI is followed by a sentence-final full stop: `…:2020:559.` The last character class allows `.`, so the greedy quantifier first consumes `559.`. The word boundary `\b` then fails, because both `.` and end-of-string are non-word positions. The engine backtracks one character, and `\b` succeeds between `9` and `.`. The match is the correct `ECLI:EU:C:2020:559`. The behaviour is correct, but it happens *by accident of backtracking*. A more explicit formulation forbids a trailing dot: `[A-Z0-9]+(?:\.[A-Z0-9]+)*`. Students should learn to reason about regexes at this level of detail. Legal punctuation (dots, slashes, parentheses) is exactly where naive patterns break.

### 3.2 Case numbers

```python
CASE_PATTERN = re.compile(r"\bCase\s+[A-Z]-\d+/\d+\b")
```

EU case numbers have the form `<court letter>-<serial>/<two-digit year>`. `C-` is used for the Court of Justice and `T-` for the General Court. Two design decisions should be noted:

1. The keyword `Case` is **included in the span**. This is an annotation-guideline decision, not a technical one. Many corpora would annotate only `C-311/18`. Whatever the choice, it must be consistent with the gold standard, otherwise strict evaluation (§10) counts a boundary error.
2. Joined cases (`Joined Cases C-293/12 and C-594/12`) and appeal suffixes (`C-123/21 P`) are not covered. This is acceptable for a teaching pattern but not for production.

### 3.3 Legislative acts

```python
EU_ACT_PATTERN = re.compile(
    r"\b(?:Regulation|Directive|Decision)\s+\((?:EU|EC|EEC)\)\s+\d{4}/\d+\b"
)
```

This pattern encodes the **post-2015 unified numbering scheme** of EU acts: `Type (Domain) YYYY/N`, for example `Regulation (EU) 2016/679`. Before 2015, citation conventions differed by act type and over time:

| Period / type | Typical form |
|---|---|
| Post-2015, all acts | `Regulation (EU) 2016/679`, `Directive (EU) 2019/790` |
| Pre-2015 directives | `Directive 95/46/EC` (year/number/domain, no parentheses) |
| Pre-2015 regulations | `Regulation (EC) No 45/2001` (number/year, with `No`) |

So the pattern has high precision on modern citations, and its recall drops sharply on historical ones. Section 16 of the notebook shows this: `Directive (EC) 95/46` is *not* matched because `\d{4}` requires a four-digit year. (The canonical form of that act is in fact `Directive 95/46/EC`, which the pattern would not match either.) This is a textbook case of **distribution shift along the temporal axis**. Legal drafting conventions change, and an extractor developed on recent texts fails silently on older ones.

### 3.4 Provision references and named groups

```python
PROVISION_PATTERN = re.compile(
    r"\bArticle\s+"
    r"(?P<article>\d+[A-Za-z]?)"
    r"(?:\((?P<paragraph>\d+)\))?"
    r"(?:\((?P<point>[a-z])\))?"
)
```

This is the most instructive pattern in the notebook, because it does **recognition and parsing at the same time**. The named groups `article`, `paragraph` and `point` turn the regex from a span detector into a shallow *structural parser*:

```text
Article 6(1)(f)   →   { article: "6", paragraph: "1", point: "f" }
```

Points worth noting:

- `\d+[A-Za-z]?` covers inserted articles such as `Article 8a`, which are common in amended legislation.
- The optional groups are **nested left to right**: a point is recognised only after a paragraph. This reflects how EU provisions are actually written. But the pattern would also accept `Article 6(f)`, attaching `(f)` to the *point* slot without a paragraph. Whether that is right depends on the drafting conventions of the document.
- Not covered: ranges (`Articles 12 to 14`), enumerations (`Article 6(1)(a) and (f)`), sub-points (`(i)`, `(ii)`), and the *first/second subparagraph* convention. Each needs its own grammar extension. Enumerations in particular raise a **coordination-ellipsis** problem: `(f)` in `6(1)(a) and (f)` means `6(1)(f)`, so the extractor must reconstruct material that is not in the text.

---

## 4. Recognition versus parsing

The notebook separates two operations that are often conflated:

```text
RECOGNITION                     PARSING
"there is an ECLI at            "this ECLI has country=EU, court=C,
 characters 151–169"             year=2020, ordinal=559"
```

`parse_ecli` performs the second step by splitting on colons. It also checks the result: it requires exactly five parts and the literal prefix `ECLI`. This validation step matters. Recognition and parsing can fail **independently**:

- a span can be recognised correctly and parsed wrongly (e.g. a court code containing a colon in a malformed source);
- a parser can be correct while the recogniser has the wrong boundaries.

So the two steps need **separate evaluation metrics** (§10–11). Parsing turns a string into a *record*. That record is what makes linking to authoritative resources possible later (Part II of this lecture).

---

## 5. A common representation for heterogeneous extractors

`regex_entities` normalises the output of all patterns into a single schema:

```python
{"text": ..., "label": ..., "start": ..., "end": ..., "source": "regex"}
```

The schema is the **integration contract** of the hybrid system. Three of its fields deserve a comment:

- **Character offsets** (`start`, `end`), not token indices. Offsets do not depend on tokenisation, so components with different tokenisers (a regex, a spaCy pipeline, a subword-based transformer) can be compared and merged. Offsets are the lingua franca of span annotation (cf. brat standoff, the W3C Web Annotation model).
- **`source`** records provenance. In a production system provenance is essential for debugging, for confidence calibration, and for legal auditability ("why does the system believe this is a provision?").
- The output is **sorted by `(start, end)`**. This deterministic ordering makes later merging and overlap detection simpler.

---

## 6. Gazetteers with spaCy's `EntityRuler`

```python
nlp = spacy.blank("en")
ruler = nlp.add_pipe("entity_ruler", config={"overwrite_ents": True})
ruler.add_patterns([
    {"label": "COURT",        "pattern": "Court of Justice",    "id": "COURT_CJEU"},
    {"label": "ORGANIZATION", "pattern": "Ministry of Justice", "id": "ORG_MINISTRY_JUSTICE"},
    {"label": "PERSON",       "pattern": "Smith",               "id": "PERSON_SMITH"},
])
```

Technical observations:

1. **`spacy.blank("en")`** gives a pipeline with only a tokeniser: no tagger, parser or statistical NER. The `EntityRuler` is therefore the *only* source of `doc.ents`. This is intentional, because it isolates the contribution of the gazetteer.
2. **String patterns** are phrase patterns. Internally they are tokenised and matched with a `PhraseMatcher`, which is efficient (roughly linear in document length, whatever the gazetteer size) but matches exact token sequences. `court of justice` in lower case would not match unless `phrase_matcher_attr: "LOWER"` is set.
3. **`overwrite_ents=True`** lets ruler matches override entities predicted by earlier components. Here it has no effect because there are none. In a pipeline that also contains a statistical `ner` component, the order of the ruler relative to `ner` and this flag together set the **precedence policy** between rules and model. That is an architectural decision.
4. **The `id` field** fills `ent.ent_id_`. As the notebook says, this is a *minimal normalisation layer*. Different surface forms (`Court of Justice`, `CJEU`, `the Court`) can share one identifier, so recognition becomes the first step towards **entity linking**.

A gazetteer is the right tool for a **closed, slowly changing, authoritative set**: EU institutions, national supreme courts, agencies. It is the wrong tool for parties, which form an open set. Including `Smith` in the gazetteer is a pedagogical shortcut. In a realistic system persons would come from a statistical or generalist NER component (§13).

---

## 7. Merging sources

Section 8 of the notebook simply concatenates the regex and ruler outputs and sorts them. Here this works because the two sources produce **disjoint** spans. In general a merge policy has to answer three questions:

- **Exact duplicates**: the same span and label from two sources. Keep one and union the provenance.
- **Conflicting labels on the same span**: e.g. `Court of Justice` as `ORG` from a statistical model and `COURT` from the gazetteer. Resolve by source priority, by specificity (the more specific label wins), or by confidence.
- **Overlapping, non-identical spans**: e.g. `Article 6(1)(f)` and `Article 6(1)(f) of Regulation (EU) 2016/679`. Decide whether the downstream representation is *flat* (keep one span by a longest-match or priority rule; cf. `spacy.util.filter_spans`) or *nested* (keep both, §9).

These choices are rarely discussed in tutorials, but they have a large effect on measured performance.

---

## 8. Legal roles are contextual, not intrinsic

This is conceptually the most important part of the notebook.

```text
Smith                    type = PERSON          role = APPELLANT
Ministry of Justice      type = ORGANIZATION    role = RESPONDENT
```

The **entity type** is a property of the referent: Smith is a person in every document. The **role** is a property of a *relation* between the referent and a *particular proceeding*. The same ministry may be a respondent in one case, an intervener in another and an applicant in a third. Formally, a role is a binary relation

$$
\text{role} : \text{Party} \times \text{Proceeding} \rightarrow \{\text{APPELLANT}, \text{RESPONDENT}, \text{INTERVENER}, \dots\}
$$

and not a unary predicate over spans. This is why the notebook says role assignment is "already close to relation extraction". The case is implicit here because the document is about only one case. Once a document mentions several proceedings (a judgment citing earlier case law, for instance), the case argument must be made explicit.

### 8.1 The toy role assigner

```python
("APPELLANT",  re.compile(r"appeal\s+brought\s+by\s+(?P<name>[A-Z][a-z]+)")),
("RESPONDENT", re.compile(r"against\s+the\s+(?P<name>Ministry of Justice)")),
```

These are **lexico-syntactic trigger patterns**, in the tradition of Hearst-style patterns for relation extraction. The cue phrase (*appeal brought by*, *against*) marks the relation, and the captured group gives the argument. They have two different weaknesses:

- The appellant pattern captures a single capitalised token, `[A-Z][a-z]+`. It would truncate `Maximillian Schrems` to `Maximillian` and miss `McDonald's` entirely: a **boundary error by construction**.
- The respondent pattern hard-codes the name of the respondent. It is in effect a gazetteer entry disguised as a contextual rule, and it does not generalise.

A more principled rule-based design captures the role argument **by reference to an already recognised entity**. For example, a spaCy `Matcher` or `DependencyMatcher` pattern could require an `ENT_TYPE` in `{PERSON, ORGANIZATION}` after the trigger, or follow the dependency arc `brought —agent→ by —pobj→ X`. Recognition and role assignment then become cleanly *factorised*: the first decides *what* the entity is, the second decides *what it does in this case*.

### 8.2 Joining type and role (notebook §11)

The join is done on the exact `(start, end)` key:

```python
typed_by_span = {(ent["start"], ent["end"]): ent for ent in spacy_result}
```

It works in the example because the role patterns capture *exactly* the same character spans as the gazetteer. In general this is fragile. If one component includes a determiner (`the Ministry of Justice`) and the other does not, the join silently returns `None`. More robust strategies are **overlap-based joins** (maximum character overlap, or an IoU threshold) or, better, the design of §8.1, where the role component works directly on recognised entities and no join is needed.

---

## 9. Nested legal references

```text
Article 6(1)(f) of Regulation (EU) 2016/679
└─────┬──────┘    └──────────┬───────────┘
   PROVISION              LEGAL_ACT
└───────────────────┬──────────────────────┘
              LEGAL_REFERENCE
```

The three spans are all legitimate and all useful: the provision on its own, the act on its own, and the complete reference that fixes *which* Article 6 is meant. Nesting is pervasive in legal text. References are *compositional*: they are built from smaller referring expressions by structural connectives (*of*, *in conjunction with*, *as amended by*).

### 9.1 Why `doc.ents` cannot hold them

The `doc.ents` layer in spaCy is a **flat, non-overlapping segmentation**. Every token carries one IOB tag (`B-`, `I-`, `O`), so a token cannot belong to two entities. This is the representational limit of the **sequence-labelling formulation of NER** (BIO/BILOU tagging with a CRF or softmax head) discussed in Lecture 12. Any nested structure must be flattened, and information is lost.

Solutions in the literature fall into three families:

1. **Span-based NER**: enumerate candidate spans up to a maximum length and classify each one independently (e.g. span classification over pooled transformer representations). Overlaps are allowed by construction. GLiNER (§13) belongs to this family.
2. **Layered / stacked sequence labelling**: one tagging layer per nesting level.
3. **Hypergraph or parsing-based approaches**: treat nested mentions as constituents of a tree.

### 9.2 `doc.spans` and `SpanGroup`

```python
nested_doc.spans["legal_refs"] = legal_spans
```

`doc.spans` is a dictionary of named `SpanGroup`s. Unlike `doc.ents`, a `SpanGroup` may contain **arbitrarily overlapping spans**. It is also the storage format used by spaCy's trainable `SpanCategorizer` (`spancat`). So the notebook's representation can serve directly as training data for a learned nested-span model, a useful bridge from rules to learning.

**Technical caveat on `alignment_mode="expand"`.** `Doc.char_span` converts character offsets to token boundaries. With `"strict"` (the default) it returns `None` if the offsets do not fall exactly on token boundaries. With `"expand"` it widens the span to the nearest enclosing token boundaries. This matters here, because spaCy's English tokeniser treats parentheses with affix rules and may split a string like `6(1)(f)` at positions that do not match the regex boundaries. `"expand"` makes the conversion robust, **but it can silently enlarge a span**. Students should compare `"strict"`, `"contract"` and `"expand"` on such strings and inspect the tokenisation with `[t.text for t in nested_doc]`. A boundary mismatch between the character level and the token level is a classic, hard-to-see source of evaluation noise.

---

## 10. From recognition to identifiers

Notebook §§14–15 link the composite reference to a structured record and then to an **ELI-like URI**:

```text
"Article 6(1)(f) of Regulation (EU) 2016/679"
        ↓ parse_legal_reference
{article: 6, paragraph: 1, point: f, legal_act: "Regulation (EU) 2016/679"}
        ↓ toy_eli_from_reference
https://eur-lex.europa.eu/eli/reg/2016/679/art_6/par_1/oj
```

This is the legal analogue of **entity normalisation** in biomedical NLP (mapping a disease mention to a UMLS/MeSH concept). There is one important difference. In biomedicine normalisation is usually *retrieval* over a large lexicon of synonyms. In law, much of it can be done by *construction*, because the identifier scheme is compositional and deterministic.

Three critical remarks, which the notebook partly anticipates by calling the function "not a general EUR-Lex resolver":

1. **The point `(f)` is dropped.** The function adds `art_` and `par_` segments but no point segment. So two distinct legal references, `6(1)(a)` and `6(1)(f)`, map to the same URI. **Normalisation has lost information.** This is exactly the kind of error that span-level F1 cannot detect.
2. **URI form.** The canonical ELI of the GDPR is `http://data.europa.eu/eli/reg/2016/679/oj`. The `eur-lex.europa.eu/eli/...` form is a resolvable presentation URL, not the canonical identifier. Also, the published EU ELI does not guarantee resolvable URIs for every subdivision. A constructed URI should therefore be treated as a *key*, not as a dereferenceable link, unless it has been checked against the resolver.
3. **Act-type coverage.** Only `Regulation (EU)` is handled. Directives (`dir`) and decisions (`dec`) would need their own branches.

Part II of the lecture takes this step further, from identifiers to a knowledge graph.

---

## 11. Evaluation

### 11.1 Strict entity-level P/R/F1

An entity counts as correct only if **start, end and label all match**:

$$
P = \frac{|G \cap \hat{Y}|}{|\hat{Y}|}, \qquad
R = \frac{|G \cap \hat{Y}|}{|G|}, \qquad
F_1 = \frac{2PR}{P+R},
$$

where $G$ and $\hat{Y}$ are sets of `(start, end, label)` triples. This is the CoNLL convention, also implemented by `seqeval` in strict mode.

On the running example the regex system reaches $P = R = F_1 = 1.0$: four gold entities, four predictions, all exact matches. This result must be read critically. **The rules were written while looking at this sentence, and are evaluated on the same sentence.** It is a measurement on the development set, not a generalisation estimate. A credible protocol needs a held-out test set from a different time period, different courts and ideally a different drafting tradition.

Two further remarks:

- **Strictness is a design choice.** Under the strict criterion, `C-311/18` against a gold `Case C-311/18` is a false positive *and* a false negative. Relaxed variants report this as partial credit: *type match with overlap*, *exact boundary regardless of type*, or the four-way scheme of SemEval-2013 Task 9 (strict, exact, partial, type). In legal IE, where downstream linking needs exact boundaries, strict evaluation is usually the right primary metric. Relaxed metrics are useful for diagnosis.
- **Nested gold standards.** If the gold standard has `LEGAL_REFERENCE` spans and the system is flat, the system is penalised by construction. The evaluation set-up must match the representation.

### 11.2 Field-level parsing accuracy

Notebook §19 evaluates the parsed provision *field by field* (article, paragraph, point). This confirms the point of §4: **recognition and parsing are separate tasks with separate metrics**. A useful finer metric is *exact-record accuracy*, the share of references whose fields are *all* correct. A single wrong field (paragraph 2 instead of 1) changes the legal meaning completely, so per-field averages can hide errors that matter.

---

## 12. Error taxonomy and remedies

The notebook proposes a classification of errors. The table below adds the diagnostic signal and the typical remedy for each class:

| Error class | Example | Diagnostic signal | Typical remedy |
|---|---|---|---|
| **Boundary** | `Regulation` vs `Regulation (EU) 2016/679` | Overlap ≠ exact match | Extend patterns; revise annotation guidelines |
| **Type** | `Court of Justice` → `ORGANIZATION` | Exact span, wrong label | Gazetteer precedence; finer label set |
| **Role** | Appellant tagged as respondent | Correct entity, wrong relation | Better trigger patterns; dependency-based rules; supervised relation classifier |
| **Citation parsing** | `6(2)(f)` instead of `6(1)(f)` | Correct span, wrong fields | Fix grammar or named groups; field-level tests |
| **Linking / normalisation** | Right act, wrong URI | Correct record, wrong identifier | Registry lookup; resolver validation |
| **Nested-entity** | Only the outer span is found | Missing inner or outer layer | Span-based representation (`doc.spans`, `spancat`) |

One remark on the notebook's example: the pair `Article 6(2)(f)` vs `Article 6(1)(f)` is labelled a *citation parsing* error. Strictly speaking, if the *surface string* predicted is `6(2)(f)` while the text says `6(1)(f)`, the fault is in recognition (or upstream OCR), not in parsing. A true parsing error leaves the string correct and assigns a field wrongly. This distinction is not pedantic. **The remedy depends on the category**, so the categorisation must be exact.

---

## 13. Open-label NER with GLiNER (optional component)

GLiNER (Zaratiana et al., NAACL 2024) is a **span-based, label-conditioned** NER model. A bidirectional transformer encoder jointly encodes the input text and a list of natural-language label descriptions (`"court"`, `"legal provision"`, …). Each candidate span is then scored against each label embedding, and a threshold (here 0.5) selects predictions. Two properties make it relevant to legal IE:

1. **The label set is given at inference time.** No retraining is needed to ask for `"respondent"` or `"legal act"`. This suits a domain where label inventories differ by jurisdiction and application.
2. **It is span-based**, so it can in principle produce overlapping spans (§9.1). The default decoding, however, usually applies a greedy non-overlap constraint unless configured otherwise.

The notebook's final questions are well chosen. The point of comparison is not "which system finds more spans" but **which component is best for which object class**:

- Identifiers and citations: regex. GLiNER can find them, but cannot *parse* them, and the threshold adds uncertainty where none is needed.
- Courts and institutions: gazetteer, for reliability and built-in normalisation (`ent_id_`).
- Persons and organisations in open text: learned NER, which generalises to unseen names.
- Roles: neither alone. Asking GLiNER for the label `"appellant"` conflates type and relation (§8). It may work on simple sentences, but the model has no explicit notion of the proceeding the role refers to.

---

## 14. Synthesis: the reference architecture

```text
legal document
      │
      ├── regex / grammars ──────────► ECLI, case numbers, act numbers, provisions
      │                                   │
      │                                   └─► structural parsing (named groups)
      ├── gazetteers / EntityRuler ──► courts, institutions  (+ resource IDs)
      ├── learned / generalist NER ──► persons, organisations, open categories
      │
      ▼
 span integration (offsets, provenance, merge policy, flat vs nested)
      │
      ▼
 role assignment (contextual; relation to the proceeding)
      │
      ▼
 normalisation (records → identifiers → URIs)
      │
      ▼
 structured legal information   ──►  Part II: knowledge graph
```

### Take-home messages

1. Legal NER is broader than `PER/ORG/LOC`. The most valuable objects are identifiers, citations and provisions.
2. The **formality of the target** should determine the technique. Highly regular objects go to rules, closed sets to gazetteers, open sets to learned models.
3. **Recognition, parsing and normalisation** are distinct steps, with distinct failure modes and distinct metrics.
4. **Roles are relations**, not types. Modelling them as relations is both conceptually correct and practically more robust.
5. Legal references are **compositional and nested**. Flat sequence labelling cannot represent them; span-based representations can.
6. Evaluation must be strict, done on held-out data, and paired with an **error taxonomy** that points to remedies.

---

## 15. Exercises

1. **Robust ECLI.** Rewrite `ECLI_PATTERN` so it does not depend on backtracking to exclude a trailing full stop. Test it on national ECLIs such as `ECLI:NL:HR:2019:1234` and `ECLI:IT:CASS:2020:12345CIV`.
2. **Historical acts.** Extend `EU_ACT_PATTERN` to cover `Directive 95/46/EC` and `Regulation (EC) No 45/2001`. Then parse each match into a normalised record `{type, domain, year, number}`. Note that number and year are *in different orders* in the two conventions.
3. **Enumerations.** Design an extractor for `Article 6(1)(a) and (f)` that outputs *two* fully specified provision records. Which problem from computational linguistics are you solving?
4. **Entity-anchored roles.** Re-implement the role assigner with a spaCy `Matcher` that requires the role argument to be an existing entity of type `PERSON` or `ORGANIZATION`. Compare its robustness with the regex version on paraphrases ("Smith's appeal against the Ministry", "the Ministry, respondent in the appeal brought by Smith").
5. **Alignment.** Print the tokenisation of the running example with `spacy.blank("en")`. Then convert every nested span with `alignment_mode` set to `"strict"`, `"contract"` and `"expand"`, and report which spans change or disappear.
6. **Evaluation.** Build a 20-sentence held-out test set from real EUR-Lex judgments. Report strict and partial P/R/F1, per label, and classify every error according to §12.
7. **Lossless normalisation.** Fix `toy_eli_from_reference` so that distinct provisions never collapse to the same identifier. Support directives and decisions.
