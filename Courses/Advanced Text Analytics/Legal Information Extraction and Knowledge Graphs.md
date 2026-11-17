---
type: course-note
date: 2026-11-17
tags:
  - course
  - advanced-text-analytics
  - legal-nlp
  - information-extraction
  - skos
  - knowledge-graph
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L14]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 14) on **legal information extraction and its grounding in Semantic Web standards**: why legal IE differs from newswire NER, the three classes of legal objects (ordinary entities, legal references/provisions, contextual roles), the hybrid rules + gazetteers + learned-models architecture, SKOS (labels, hierarchy, mapping properties, integrity rules), SKOS vs OWL, the legal identifier standards (ELI, ECLI, CELEX, EuroVoc), and the text -> spans -> resources -> RDF -> SPARQL pipeline. Created 2026-11-17 from `raw/ATA_L14.md`; a few claims of that source are corrected against the two L14 notebook companions and the W3C SKOS specification (see Section 9). The runnable walkthroughs (regex citation parser, `EntityRuler`, nested `doc.spans`, ELI-like URIs, `rdflib` graph, SPARQL, CELLAR) live in [[Legal Information Extraction and Knowledge Graphs - Notebook]]. Prerequisites: span-based NER and GLiNER from [[Advanced Named Entity Recognition]]; concept normalization from [[Biomedical Named Entity Recognition and Concept Normalization]].

## 1. Why Legal IE Is Different

Classical NER inventories (CoNLL-2003: `PER`, `ORG`, `LOC`, `MISC`) were designed for newswire. Legal texts (statutes, judgments, regulations) contain these entities too, but what a lawyer actually needs lives elsewhere. Legal language has a **dual nature** - one text simultaneously carries:

| Layer | What it is | Example |
|---|---|---|
| **Factual entities** | real-world actors and institutions | *Court of Justice*, *Facebook Ireland* |
| **Formal citation structures** | standardized references to provisions and precedents | *Article 46(1) of Regulation (EU) 2016/679*, `ECLI:EU:C:2020:559` |
| **Normative assertions** | obligations, permissions, prohibitions, rights | "the controller **shall** provide appropriate safeguards" |

Flat token classification fails on this because legal references are **nested and hierarchical**: a point sits inside a paragraph, inside an article, inside an act. Legal IE is therefore organized as a four-part paradigm: **deterministic rules + learned neural models + authoritative resources + task-specific evaluation**.

The end goal is not a list of strings but a queryable graph:

$$
\text{Legal Text} \rightarrow \text{Entity \& Provision Extraction} \rightarrow \text{Resource Grounding} \rightarrow \text{RDF Triplification} \rightarrow \text{Queryable KG}
$$

After grounding, an extraction output is no longer "characters 12-40" but a **persistent legal resource** linked to controlled vocabularies and authoritative identifiers.

## 2. Taxonomy of Legal Objects

| Class | Sub-types | Structural character | Natural technique |
|---|---|---|---|
| **Ordinary named entities** | persons, organizations, courts, locations, dates, amounts | flat spans, standard boundaries | fine-tuned transformer NER, or zero-shot span models (GLiNER) |
| **Legal references and provisions** | acts, treaties, articles, paragraphs, points, recitals | highly nested, hierarchical, internal sub-fields | regular expressions with named groups + nested span representation |
| **Contextual legal roles** | applicant, respondent, appellant, intervener, judge, counsel | not marked on the string - depends on the proceeding | contextual classification / relation extraction |

### 2.1 Legal References Have Internal Structure

A legal reference is a **record**, not just a span:

$$
\text{Legal Reference} = \{\text{Jurisdiction},\ \text{Instrument Type},\ \text{Number},\ \text{Year},\ \text{Article},\ \text{Paragraph},\ \text{Point}\}
$$

Example: *"Article 46(1) of Regulation (EU) 2016/679"* must be split into the primary act (*Regulation 2016/679* = the GDPR) and its subdivision (*Article 46, paragraph 1*). This parsing is what later makes linking to identifiers possible.

### 2.2 Roles Are Relations, Not Types

A corporate body can be a *data controller* in a compliance text, a *respondent* in an administrative proceeding and an *appellant* on appeal. The token string carries no role marker - the role emerges from the procedural context ("the appeal **brought by** X **against** Y"). So a role is a property of the pair (party, proceeding), not of the span: role assignment is close to **relation extraction**. (Formalized in the notebook companion, Part I Section 6.)

### 2.3 Normative Content (LOMO)

Beyond entities, provisions carry **deontic** content. The **Legal Obligation Metadata Ontology (LOMO)** models the interpreted normative content of provisions and links it back to the ELI source. Its core modalities:

1. **Obligations** - mandatory requirements on a legal subject.
2. **Permissions** - authorized or discretionary actions.
3. **Prohibitions** - forbidden actions.
4. **Rights** - entitlements a subject can assert.

(The notebook companion adds *definitions* and *scope provisions*.) Mapping spans to these modalities is the basis for automated **compliance checking**: does an action satisfy the statutory mandate?

## 3. Hybrid Extraction Architecture

No single technique covers legal language. The design principle: **match the inductive bias of the component to the formality of the target**.

### 3.1 Deterministic Rules for Citations

Legislative drafting follows style manuals, so citations have predictable surface templates: `[Act Type] (EU) [Year]/[Number]`, `Article [N]([Paragraph])`. Regexes and small grammars:

- need **no labeled data**;
- reach **high precision** on these formats;
- emit **structured, hierarchical output** directly (named groups), ready for normalization;
- are **auditable** - a rule can be inspected, versioned and justified, which matters a lot in law.

Their weakness: recall drops when conventions change (e.g. pre-2015 EU act numbering) - a form of temporal distribution shift.

### 3.2 Learned Models for Open-Class Entities

Rules are brittle on free text (party names, judicial reasoning). Here learned models take over:

- **BIO token classification** labels each (sub)token; it struggles with long nested noun phrases and cannot represent overlaps (one tag per token).
- **Span-based models**, notably **GLiNER** (Generalist and Lightweight model for Named Entity Recognition), score candidate spans against natural-language **label embeddings**. Two advantages for law:
  1. **Open labels at inference time** - ask for `"data protection authority"` or `"transfer mechanism"` without retraining;
  2. **Span representation** - overlapping spans are representable in principle (but see the caveat in Section 9: default decoding is flat).

### 3.3 Which Component for Which Object

| Object | Best component | Why |
|---|---|---|
| ECLI, case numbers, act numbers, provisions | regex + parser | regular syntax; the useful output is the parse, not just the span |
| Courts, institutions | gazetteer (e.g. spaCy `EntityRuler`) | closed, slowly changing set; gives an ID for free |
| Persons, organizations in open text | learned / generalist NER | open set, low surface regularity |
| Roles | relation extraction on top of recognized entities | role depends on the proceeding, not the string |

## 4. SKOS - Simple Knowledge Organization System

SKOS is a W3C Recommendation for representing **controlled vocabularies, thesauri, taxonomies and subject heading systems** in RDF. It is the layer that removes lexical ambiguity from extracted topics.

### 4.1 Concept vs String

The central idea: **a concept is not its label**. Keyword-based analytics treats terms as strings and suffers from synonymy, polysemy and translation mismatches. In SKOS a `skos:Concept` is an abstract unit of thought identified by a persistent **URI**, carrying:

- one preferred label per language,
- any number of alternative labels (synonyms, acronyms, variants),
- hidden labels (misspellings, obsolete forms - used for search, not display),
- membership in a `skos:ConceptScheme` (the vocabulary boundary, via `skos:inScheme`).

### 4.2 Lexical Labels

| Property | Use | Example for concept *protection of personal data* |
|---|---|---|
| `skos:prefLabel` | canonical display name | `"protection of personal data"@en` |
| `skos:altLabel` | synonyms, abbreviations | `"data protection"@en`, `"data privacy"@en` |
| `skos:hiddenLabel` | typos, legacy forms, index tokens | `"personal data protecton"@en` |

### 4.3 Semantic Relations

- **`skos:broader` / `skos:narrower`** - direct hierarchical links, mutual inverses. `A skos:broader B` means **B is more general than A**. They are **not transitive** by definition, so only immediate links are asserted.
- **`skos:broaderTransitive` / `skos:narrowerTransitive`** - the transitive closure, used to compute ancestors at any depth (in SPARQL usually done with the path `skos:broader+`).
- **`skos:related`** - associative, non-hierarchical link (symmetric).
- **Mapping properties** (between *different* schemes):

| Property | Meaning |
|---|---|
| `skos:exactMatch` | interchangeable concepts (transitive) |
| `skos:closeMatch` | similar enough for some applications, not guaranteed interchangeable |
| `skos:broadMatch` / `skos:narrowMatch` | hierarchical link across schemes |
| `skos:relatedMatch` | associative link across schemes |

### 4.4 Integrity Conditions (Exam-Relevant)

- A resource has **at most one `skos:prefLabel` per language tag**.
- `prefLabel`, `altLabel` and `hiddenLabel` are **pairwise disjoint** (the same literal cannot be both preferred and alternative for one concept).
- `skos:related` is **disjoint with `skos:broaderTransitive`**: two concepts cannot be both hierarchically and associatively linked.
- *Recommended practice* (not a formal integrity condition): no two concepts in the same scheme should share the same `prefLabel` in the same language.

SKOS **allows poly-hierarchy** (several broader concepts) and does **not forbid cycles** - code that walks the hierarchy must handle both.

## 5. SKOS vs OWL

Both are RDF-based but target different abstraction levels. (The canonical treatment of OWL modeling and concept-vs-class punning is in [[OWL SKOS Punning Football Ontology]]; here the focus is the IE use case.)

| Dimension | SKOS | OWL |
|---|---|---|
| Goal | organize vocabularies for indexing and retrieval | formal domain ontologies with logical semantics |
| Core unit | `skos:Concept` (a topic / unit of indexing) | `owl:Class` (a set of individuals) |
| Hierarchy | `skos:broader` - informal "more general topic" | `rdfs:subClassOf` - strict set inclusion with inheritance |
| Labels | rich multilingual pref/alt/hidden labels | `rdfs:label` annotations |
| Reasoning | lightweight (transitive closure, mappings) | Description Logic reasoners: consistency, inferred axioms |
| Constraints | soft, allows partial or subjective hierarchies | disjointness, cardinality, domain/range |
| Typical use | subject indexing, entity normalization, thesaurus mapping | legal reasoning engines, rule checking |

Key exam point: **`skos:broader` is not `rdfs:subClassOf`**. "Appropriate safeguards" being narrower than "international data transfer" says the topic is more specific - not that every safeguard *is a* transfer. SKOS's weakness is a feature for subject vocabularies; SKOS complements OWL rather than replacing it.

## 6. Legal Identifier Standards

| Standard | Identifies | Form / structure | Example |
|---|---|---|---|
| **ELI** (European Legislation Identifier) | legislation and its subdivisions | semantic HTTP URI: type / year / number / version | `http://data.europa.eu/eli/reg/2016/679/oj` |
| **ECLI** (European Case Law Identifier) | judicial decisions | `ECLI` : country : court : year : ordinal (5 colon-separated parts) | `ECLI:EU:C:2020:559` (Schrems II) |
| **CELEX** | EUR-Lex documents | sector + year + doc type + number | `32016R0679` (3 = legislation, R = regulation) |
| **EuroVoc** | subject matter | multilingual EU thesaurus published as a SKOS concept scheme | concepts indexed under thematic domains, e.g. *12 LAW* |

**ELI ontology** (FRBR-inspired): an act is an `eli:LegalResource`; its chapters/articles/paragraphs are `eli:LegalResourceSubdivision` (a subclass) linked with `eli:is_part_of` / `eli:has_part`; `eli:is_about` links a resource to a `skos:Concept` (typically EuroVoc) - this is where legislation and vocabularies meet.

**ECLI** lets case citations from any national or EU court be indexed uniformly; normalizing citations to ECLI enables automated citation networks.

**EuroVoc** is used by the EU Publications Office to index EUR-Lex documents; linking extracted topics to EuroVoc URIs gives **cross-lingual interoperability** (one concept, labels in all official languages).

## 7. End-to-End Legal KG Pipeline

The lecture scenario is cross-border data transfer under the GDPR (Schrems II, Case C-311/18). Four phases:

1. **Span and provision extraction** - GLiNER + rule parsers find provisions, acts, decisions, courts, actors, topics.
2. **Resource grounding** - spans resolved to persistent URIs: the act (ELI), its subdivisions, the decision (ECLI), topic concepts (SKOS/EuroVoc).
3. **RDF triplification** - with `rdflib`, using standard namespaces `rdf:`, `rdfs:`, `dcterms:`, `eli:`, `skos:` plus a local `ex:` for what standards lack (cases, courts, parties).
4. **Persistence and querying** - triples stored, optionally exported to a property graph (Neo4j / PyVis), queried with **SPARQL** for multi-hop questions such as *"which topics are covered by the provisions this case interprets?"*.

Typical triple shapes:

```text
<GDPR>          a eli:LegalResource .
<Art. 46(1)>    a eli:LegalResourceSubdivision ; eli:is_part_of <GDPR> ; eli:is_about <topic> .
<topic>         a skos:Concept ; skos:prefLabel "..."@en ; skos:altLabel "..."@en .
<Case C-311/18> ex:interprets <Art. 46(1)> .
```

A flat list of spans cannot answer a join across case law, legislation and vocabulary - that is the main argument for the graph representation. RDF/SPARQL basics are in [[RDF Data Model]].

## 8. Future Directions

- **Citation networks** - typed relations between acts: `cites`, `amends`, `repeals`, `derogates_from` (ELI provides `eli:cites`, `eli:amends`, `eli:repeals`, `eli:based_on`).
- **Normative reasoning** - combine the KG with deontic models (LOMO) for compliance evaluation.
- **Graph Neural Networks** on the triplified graph for precedent / outcome prediction (speculation as to maturity).

## 9. Source Reconciliation Notes

`raw/ATA_L14.md` (a generated summary report) disagrees with the more careful notebook companions or the SKOS specification on a few points. The versions above are the corrected ones:

| Claim in the theory source | Corrected version | Confidence |
|---|---|---|
| "No two concepts in one scheme may share a prefLabel per language" is an **integrity rule** | the integrity condition is *at most one prefLabel per language per resource*; uniqueness across concepts is only a recommendation | high |
| `skos:related` is disjoint with `skos:broader` / `skos:narrower` | formally disjoint with `skos:broaderTransitive` (which covers direct broader too) | high |
| GLiNER "handles nested entities natively" | span-based, so overlaps are *representable*, but default decoding applies a greedy non-overlap (flat) constraint | high |
| LOMO = "Legal Obligation and Normative Content", four modalities | Legal Obligation Metadata Ontology; modalities also include definitions and scope provisions | high |
| ECLI has four components | four components after the literal `ECLI` prefix - five colon-separated parts in total | high |
| EuroVoc "Domain 36: Information Technology and Data Processing" | domain 36 is *Science*; *information technology and data processing* is a microthesaurus under domain 32 (*Education and communications*) | medium |

## 10. Take-Home Messages

1. Legal IE is broader than `PER/ORG/LOC`: the most valuable objects are identifiers, citations and provisions.
2. Match technique to formality: regular objects -> rules, closed sets -> gazetteers, open sets -> learned models.
3. Legal references are records with internal structure and are nested; flat BIO tagging cannot represent them.
4. Roles are relations between a party and a proceeding, not entity types.
5. SKOS separates concept (URI) from labels and gives a lightweight hierarchy for indexing; OWL gives formal class semantics.
6. ELI, ECLI, CELEX and EuroVoc are the authoritative identifier layer; grounding to them turns spans into linked, queryable resources.

## 11. Suggested Review Questions

1. What are the three layers of legal language, and why does flat NER fail on them?
2. Why are citations a good fit for regexes while party names are not? Give the auditability argument.
3. Why is a legal role not an entity type? How would you model it in RDF?
4. Explain `skos:prefLabel` vs `altLabel` vs `hiddenLabel` and the integrity conditions on them.
5. Why is `skos:broader` not transitive, and how do you get all ancestors anyway?
6. Contrast `skos:broader` with `rdfs:subClassOf` using a legal example.
7. Decompose `ECLI:EU:C:2020:559`, `32016R0679` and `http://data.europa.eu/eli/reg/2016/679/oj`.
8. Which ELI classes and properties model an article of an act and its subject matter?

## Related

- [[Legal Information Extraction and Knowledge Graphs - Notebook]] - companion walkthroughs for both L14 notebooks
- [[Advanced Named Entity Recognition]] - Lecture 12: span classification, GLiNER, nested spans
- [[Biomedical Named Entity Recognition and Concept Normalization]] - Lecture 13: normalization as retrieval, contrasted with legal normalization by construction
- [[Knowledge Resources for Text Analytics]] - Lecture 2: gazetteers, thesauri, ontologies, knowledge graphs
- [[Annotation and Evaluation of Information Extraction]] - strict vs relaxed matching used in the legal evaluation
- [[OWL SKOS Punning Football Ontology]] - cross-course bridge ([[Graph Databases]]): canonical OWL and SKOS modeling
- [[Linked Open Data]] - cross-course bridge: persistent URIs and vocabulary reuse

## Sources

- [[ATA_L14]] (AY 2026/2027)
- W3C SKOS Reference, integrity conditions: https://www.w3.org/TR/skos-reference/ (2009 Recommendation; used to correct Section 4.4)
