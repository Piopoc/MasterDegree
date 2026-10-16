# Lecture 5: Knowledge Resources for Text Analytics

Source notebook: `ATA_L05_Knowledge_Resources_for_Text_Analytics.ipynb` (Advanced Text Analytics).

## Where this lecture sits

Lecture 4 showed how to encode explicit knowledge with rules and patterns. This lecture asks a prior question: where does the knowledge inside those rules actually come from? When a system flags `breast cancer`, `myocardial infarction`, or `BRCA1`, the expression has to come from somewhere. The lecture walks through the main kinds of knowledge resources we can hand to a text analytics system, what each one encodes, and what each one lets us do. The path runs from plain name lists all the way to knowledge graphs, with biomedical and legal examples along the way.

The code cells stay small on purpose. Everything runs on spaCy (`en_core_web_sm`) and its `PhraseMatcher`, which is enough to show matching behavior for each resource type.

## Lists and gazetteers

The simplest resource is a list of names: `BRCA1`, `BRCA2`, `TP53`, `EGFR`. It works for surface matching, but the list says nothing about what those strings are. A gazetteer closes that gap by pairing each name with a category, so `BRCA1 → GENE`. In the notebook, a `PhraseMatcher` built over the gazetteer's keys tags "BRCA1" and "TP53" correctly in a sample sentence.

Membership in a gazetteer is evidence, not interpretation. The lecture's example is `Washington`, which can denote a person, a city, a state, or an institution. The resource narrows the possibilities; context decides among them.

## Terminologies: from strings to concepts

A terminology resource organizes entries around concepts rather than strings. Concept `C001` carries a preferred term ("myocardial infarction"), synonyms ("heart attack"), abbreviations ("MI"), and a category. Several surface forms can point to the same concept.

This is where the lecture separates two operations that are easy to confuse: mention recognition and concept normalization. In the demo, "heart attack" and "myocardial infarction" are both found in a sentence, and both map to `C001`, which normalizes back to "myocardial infarction". Finding the mention is one step; deciding what it refers to is the next one.

A controlled vocabulary works along the same lines. It selects the authorized terms for indexing and retrieval, so "heart attack", "cardiac infarction", and "MI" all collapse into one preferred form. The aim is to reduce uncontrolled lexical variation.

## Taxonomies and thesauri: hierarchy and relations

A taxonomy adds a broader/narrower hierarchy over concepts: myocardial infarction is a kind of ischemic heart disease, which is a kind of cardiovascular disease, which is a kind of disease. The notebook walks up this chain with a small `broader_concepts` function that returns `['Ischemic heart disease', 'Cardiovascular disease', 'Disease']`. Hierarchies support query expansion, classification, aggregation, semantic search, and hierarchical evaluation.

A thesaurus keeps that hierarchy and adds associative links: preferred term, alternative term, broader term, narrower term, related term. The example entry for "data protection" links to "privacy" and "personal data" as related concepts.

Resources at this level often separate two dimensions. The linguistic side records how a concept is expressed in English, Italian, or French. The conceptual side records what the concept is: a `C001` identifier with its broader relation. Labels in three languages attach to the same node.

## Ontologies: formal domain models

An ontology explicitly represents classes, relations, properties, and constraints. The example defines `Gene` and `Disease` classes plus an `associated_with` relation with domain `Gene` and range `Disease`. A small validation function then accepts "BRCA1 associated_with breast cancer" (Gene, Disease) and rejects "European Commission associated_with breast cancer" (Organization, Disease).

Worth stressing: an ontology does not extract the relation from text by itself. It supplies constraints used for validation, filtering, interpretation, and enrichment.

The lecture contrasts taxonomy and ontology briefly. A taxonomy is mainly hierarchy with broader/narrower links, meant for classification. An ontology is a domain model with typed relations and constraints. Real resources often blur the boundary.

## Knowledge bases and knowledge graphs

The teaching distinction: the ontology is the conceptual schema, and the knowledge base is the facts stated using some schema. The notebook stores triples like `("BRCA1", "associated_with", "breast cancer")` and answers a query by filtering on subject and relation. Viewed as a graph, entities and classes become nodes while assertions become edges. That view supports linking, enrichment, validation, search, question answering, and reasoning.

## The full family side by side

The lecture's comparison table, condensed:

| Resource | Main unit | Typical structure | Use in text analytics |
|---|---|---|---|
| Gazetteer | string/name | list + category | mention detection |
| Terminology | concept + designations | preferred terms, synonyms | normalization |
| Controlled vocabulary | authorized terms/concepts | controlled labels | indexing and search |
| Taxonomy | concepts/classes | hierarchy | generalization |
| Thesaurus | concepts + terms | broader/narrower/related | expansion, indexing |
| Ontology | formal classes | typed relations + constraints | validation, reasoning |
| Knowledge base / KG | entities + facts | graph of assertions | linking, enrichment |

These are typical emphases rather than hard categories. Real resources combine several of them.

## Two worked examples

Biomedical: "BRCA1 mutations are associated with increased breast cancer risk." A pattern finds a gene-like string; the gazetteer says BRCA1 is a known gene name; the terminology maps "breast cancer" to disease concept `D001`; the taxonomy places it under cancer and disease; the knowledge base already holds the association between BRCA1 and breast cancer.

Legal: "The Court applied Article 6 of Regulation (EU) 2016/679." A pattern recognizes the article reference; the gazetteer recognizes institutions and court names; a controlled vocabulary normalizes subject concepts; the ontology types legal resources and their relations; the knowledge base links to the specific legal resource.

The point of the pair is that no single resource answers everything. Each contributes a different kind of information about the same sentence.

## Where resources enter the pipeline

In a pipeline sketch from the lecture, text goes through NLP to mentions, mentions are checked against gazetteers and terminologies (or an NER model), then normalized to a concept or entity identifier, then consulted against ontology and knowledge base for enrichment, validation, and relations.

The lecture is blunt about limits. Resources raise problems of coverage, ambiguity, granularity, domain specificity, multilinguality, versioning, licensing, mapping between resources, and the mismatch between how text is written and the canonical terminology. A resource constrains and enriches interpretation; it does not replace contextual analysis.

The closing position is therefore additive. Rules produce candidate mentions, gazetteers and terminologies classify and normalize them, models and LLMs handle what lists cannot, and ontologies and knowledge bases validate and enrich the result. No single source of knowledge has to replace the others.

## Exercises and take-home messages

Two mini exercises ask what each resource type could contribute to one biomedical sentence and one legal sentence. The notebook ships empty dictionaries as slots to fill in (there is a separate solutions notebook for the answers).

The synthesis reduces each resource to a single question:

- Gazetteer: is this string a known member of a category?
- Terminology: which concept does this expression designate?
- Taxonomy: what broader or narrower concept does it belong to?
- Thesaurus: what related concepts and alternative labels exist?
- Ontology: what kinds of entities and relations are formally defined?
- Knowledge base: what facts are known about this particular entity or concept?

Five take-home messages close the lecture. Knowledge resources encode information that training examples alone may not recover. Different resources represent different kinds of knowledge: names, concepts, hierarchies, relations, or facts. Strings and terms are not the same thing as concepts. Resources can support recognition, normalization, expansion, validation, reasoning, and enrichment. And resources, rules, learned models, and LLMs are complementary components of a single text analytics system.
