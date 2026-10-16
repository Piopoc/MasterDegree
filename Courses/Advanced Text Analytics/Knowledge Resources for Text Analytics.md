---
type: course-note
date: 2026-10-16
tags:
  - course
  - advanced-text-analytics
  - knowledge-resources
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L05]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 5) on **knowledge resources for text analytics** - the external knowledge (lists, gazetteers, terminologies, controlled vocabularies, taxonomies, thesauri, ontologies, knowledge bases/graphs) that a system consults to recognize, normalize, validate, and enrich mentions in text. Created 2026-10-16 from [[ATA_L05]] (English NotebookLM export). Use it when the question is "where does the knowledge inside the rules come from" or "which resource type does what to a mention". Key points: the string-vs-concept distinction, mention recognition vs concept normalization, the family comparison table, and the additive position (resources do not replace rules, models, or LLMs).

## 1. Where This Lecture Sits

Lecture 4 built explicit knowledge as **rules and patterns**. This lecture asks the prior question: *where does the knowledge inside those rules come from?* When a system flags `breast cancer`, `myocardial infarction`, or `BRCA1`, the expression has to originate somewhere.

The lecture walks the ladder of knowledge resources - from plain name lists to full knowledge graphs - with biomedical and legal examples. Every code demo runs on spaCy (`en_core_web_sm`) and its `PhraseMatcher`, kept deliberately small so each resource type's *matching behavior* is visible without engineering noise.

The organizing idea is that different resources encode **different kinds of knowledge**: names, concepts, hierarchies, relations, or facts. The lecture's tension is that of *Mention Recognition* versus *Concept Normalization* (see Section 3): finding a string in text is not the same as deciding what it refers to.

## 2. Lists and Gazetteers

The simplest resource is a **list of names**: `BRCA1`, `BRCA2`, `TP53`, `EGFR`. It supports surface matching only - the list says nothing about *what* those strings are.

A **gazetteer** closes that gap by pairing each name with a category:

| Surface form | Category |
|---|---|
| `BRCA1` | `GENE` |
| `TP53` | `GENE` |
| `breast cancer` | `DISEASE` |

In the notebook, a `PhraseMatcher` built over the gazetteer's keys tags `BRCA1` and `TP53` correctly in a sample sentence.

The critical analytical caveat from the lecture: **membership in a gazetteer is evidence, not interpretation.** The canonical example is `Washington`, which can denote a person, a city, a state, or an institution. The resource *narrows the possibilities*; the surrounding context decides among them. A gazetteer answer answers one question only: *is this string a known member of a category?*

## 3. Terminologies: From Strings to Concepts

A **terminology** organizes entries around **concepts rather than strings**. Concept `C001` carries:

- a **preferred term** - "myocardial infarction";
- **synonyms** - "heart attack";
- **abbreviations** - "MI";
- a **category**.

Several surface forms thus point to the *same* concept. This is where the lecture separates two operations that are easy to conflate:

1. **Mention recognition** - find the expression in the text ("heart attack" and "myocardial infarction" are both found in a sentence).
2. **Concept normalization** - decide which concept it designates (both map to `C001`, which normalizes back to the preferred form "myocardial infarction").

Finding the mention is one step; deciding what it refers to is the next.

A **controlled vocabulary** works along the same lines, but from the retrieval side: it selects the **authorized terms** for indexing and retrieval, so "heart attack", "cardiac infarction", and "MI" all collapse into one preferred form. Its aim is to reduce *uncontrolled lexical variation*.

## 4. Taxonomies and Thesauri: Hierarchy and Relations

A **taxonomy** adds a broader/narrower **hierarchy** over concepts:

$$\text{Myocardial infarction} \;\subset\; \text{Ischemic heart disease} \;\subset\; \text{Cardiovascular disease} \;\subset\; \text{Disease}$$

The notebook walks this chain with a small `broader_concepts` function returning `['Ischemic heart disease', 'Cardiovascular disease', 'Disease']`. Hierarchies support query expansion, classification, aggregation, semantic search, and hierarchical evaluation.

A **thesaurus** keeps that hierarchy and adds **associative links**. Its standard relation set is:

| Relation | Meaning |
|---|---|
| Preferred term | the authorized label |
| Alternative term | synonym / non-preferred label |
| Broader term (BT) | the parent concept |
| Narrower term (NT) | the child concept |
| Related term (RT) | associative (not hierarchical) link |

Example: the entry for "data protection" links to "privacy" and "personal data" as *related* concepts.

Resources at this level often separate **two dimensions** - a point worth internalizing for multilingual work:

- the **linguistic side**: how a concept is expressed in English, Italian, or French;
- the **conceptual side**: what the concept *is* - a `C001` identifier with its broader relation.

Three language labels attach to the same concept node.

## 5. Ontologies: Formal Domain Models

An **ontology** explicitly represents **classes, relations, properties, and constraints**. The lecture's example defines:

- classes `Gene` and `Disease`;
- a relation `associated_with` with **domain** `Gene` and **range** `Disease`.

A small validation function then **accepts** `"BRCA1 associated_with breast cancer"` (Gene, Disease) and **rejects** `"European Commission associated_with breast cancer"` (Organization, Disease) - because the domain constraint is violated.

Worth stressing: **an ontology does not extract relations from text by itself.** It supplies *constraints* used for validation, filtering, interpretation, and enrichment. Extraction is done by rules, patterns, or models; the ontology judges the result.

### Taxonomy vs. ontology

| | Taxonomy | Ontology |
|---|---|---|
| Core content | broader/narrower hierarchy | typed relations + constraints |
| Main purpose | classification | formal domain model, reasoning |
| Unit | concept in a tree | classes, properties, individuals |

Real resources often blur the boundary - the lecture treats this as a contrast of emphasis, not a hard wall.

## 6. Knowledge Bases and Knowledge Graphs

The teaching distinction:

- the **ontology** is the *conceptual schema* (what kinds of things and relations exist);
- the **knowledge base** is the *facts* stated using some schema.

The notebook stores triples such as `("BRCA1", "associated_with", "breast cancer")` and answers a query by **filtering on subject and relation**. Viewed as a graph, entities and classes become **nodes** while assertions become **edges** - the same triple model that underpins the Semantic Web graph of [[RDF Data Model]] (cross-course).

That graph view supports linking, enrichment, validation, search, question answering, and reasoning. The knowledge base answers the last and most specific question of the whole ladder: *what facts are known about this particular entity or concept?*

## 7. The Full Family Side by Side

The lecture's comparison table, condensed:

| Resource | Main unit | Typical structure | Use in text analytics |
|---|---|---|---|
| Gazetteer | string / name | list + category | mention detection |
| Terminology | concept + designations | preferred terms, synonyms | normalization |
| Controlled vocabulary | authorized terms / concepts | controlled labels | indexing and search |
| Taxonomy | concepts / classes | hierarchy | generalization |
| Thesaurus | concepts + terms | broader / narrower / related | expansion, indexing |
| Ontology | formal classes | typed relations + constraints | validation, reasoning |
| Knowledge base / KG | entities + facts | graph of assertions | linking, enrichment |

These are **typical emphases rather than hard categories** - real resources combine several of them.

## 8. Two Worked Examples

The lecture's pedagogical point is that **no single resource answers everything**; each contributes a different *kind* of information about the same sentence.

### 8.1 Biomedical

Sentence: *"BRCA1 mutations are associated with increased breast cancer risk."*

| Resource | Contribution |
|---|---|
| Pattern | finds a gene-like string |
| Gazetteer | says `BRCA1` is a known gene name |
| Terminology | maps "breast cancer" to disease concept `D001` |
| Taxonomy | places it under cancer and disease |
| Knowledge base | already holds the association BRCA1 - breast cancer |

### 8.2 Legal

Sentence: *"The Court applied Article 6 of Regulation (EU) 2016/679."*

| Resource | Contribution |
|---|---|
| Pattern | recognizes the article reference |
| Gazetteer | recognizes institutions and court names |
| Controlled vocabulary | normalizes subject concepts |
| Ontology | types legal resources and their relations |
| Knowledge base | links to the specific legal resource |

## 9. Where Resources Enter the Pipeline

Pipeline sketch from the lecture:

```text
[Raw text] -> [NLP] -> [Mentions] -> [gazetteers / terminologies / NER model]
           -> [normalize to concept or entity id]
           -> [ontology + knowledge base] -> enrichment, validation, relations
```

The NLP and mention layer is a direct application of the annotation pipeline in [[NLP Linguistic Structures and Annotation]] (course hub note): tokenization and linguistic annotation produce the mentions that every downstream resource then classifies, normalizes, or validates.

## 10. Limits and the Additive Position

The lecture is blunt about **limits**. Resources raise problems of:

- coverage, ambiguity, granularity;
- domain specificity and multilinguality;
- versioning and licensing;
- mapping between resources;
- the mismatch between how text is written and the canonical terminology.

**A resource constrains and enriches interpretation; it does not replace contextual analysis.**

The closing position is therefore **additive**:

1. Rules produce candidate mentions;
2. gazetteers and terminologies classify and normalize them;
3. models and LLMs handle what lists cannot;
4. ontologies and knowledge bases validate and enrich the result.

No single source of knowledge has to replace the others. This is the same complementary stance that lets LLM-based semantic operators such as [[LOTUS Semantic Operators]] (cross-course) operate *over* structured and text sources rather than instead of them.

## 11. Synthesis and Take-Home Messages

Each resource reduces to a single diagnostic question:

| Resource | Question it answers |
|---|---|
| Gazetteer | Is this string a known member of a category? |
| Terminology | Which concept does this expression designate? |
| Taxonomy | What broader or narrower concept does it belong to? |
| Thesaurus | What related concepts and alternative labels exist? |
| Ontology | What kinds of entities and relations are formally defined? |
| Knowledge base | What facts are known about this specific entity or concept? |

Take-home messages:

1. Knowledge resources encode information that **training examples alone may not recover**.
2. Different resources represent different kinds of knowledge: names, concepts, hierarchies, relations, or facts.
3. **Strings and terms are not the same thing as concepts** - surface form is not meaning.
4. Resources can support recognition, normalization, expansion, validation, reasoning, and enrichment.
5. Resources, rules, learned models, and LLMs are **complementary components** of one text analytics system.

## Related

- [[NLP Linguistic Structures and Annotation]] - the annotation pipeline that produces the mentions these resources consume
- [[Linked Open Data]] - how ontologies and knowledge graphs are published on the Web (cross-course)
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L05]] (AY 2026/2027)
