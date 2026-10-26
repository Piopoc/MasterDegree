---
type: course-note
date: 2026-10-10
tags:
  - course
  - graph-databases
  - rdf
course: "[[Graph Databases]]"
sources:
  - "[[RDF]]"
ai-first: true
---

## For Future Agent

Study note for the [[Graph Databases]] course on the **Resource Description Framework (RDF)** - the W3C data model underlying the Semantic Web: triples, URI-based identity resolution, CURIEs/namespaces, literals and typing, blank nodes, reification (RDF-star), named graphs/datasets, and concrete serializations. Created 2026-10-10 from [[RDF]], an English briefing on Prof. Gianmaria Silvello's course materials. Key points: the graph-merging rule ("same URI iff same node"), the subject/predicate/object position rules, and N-Triples as the canonical lowest-level syntax.

## 1. Purpose and the Triple

RDF (*Resource Description Framework*) manages distributed data by connecting Web documents to the real-world "things" they describe.

- **Resource**: anything someone may want to talk about or describe.
- **Statement**: every statement is a **triple** - the fundamental building block of the model:

$$\boxed{\text{triple} = \langle \text{subject}, \text{predicate}, \text{object} \rangle}$$

| Subject | Predicate | Object |
|---|---|---|
| `Francis Ford Coppola` | `directed` | `The Godfather` |
| `Woody Allen` | `actedIn` | `Manhattan` |
| `Manhattan` | `setIn` | `New York City` |

### Position rules

1. **Subject** - a URI (resource) or a blank node.
2. **Predicate** - must be a **URI** representing a property/relationship.
3. **Object** - a URI, a literal, a blank node, or (RDF 1.2) a **triple term**.

## 2. Identity Resolution and Graph Merging

Data integration reduces to: *"When is a node in one graph the same node as a node in another graph?"*

- **URI**: a global, web-wide identifier; a **URL** is a specific kind of URI (its syntax lets it be *dereferenced* - protocol, server, path locate a resource).
- **Graph-merging rule**: a node from one graph merges with a node from another **if and only if they share the exact same URI**.

```
[Server 1: https://dbpedia.org/page/Francis_Ford_Coppola]
                    │ (same URI)
[Server 2: https://dbpedia.org/page/Francis_Ford_Coppola]
                    ▼
          [Merged Graph Node]
```

## 3. CURIEs and Namespaces

**CURIE** (Compact URI) = scoping mechanism avoiding verbose URI strings: `prefix:reference` separated by a colon.

```
curie     := [[ prefix ] ':'] reference
prefix    := NCName
reference := irelative-ref (as defined in IRI)
```

The prefix may be omitted (with or without the colon). A **namespace** is a set of names uniquely identifying objects, preventing naming collisions (hierarchized as built-in, global, local in Python/XML).

| Prefix | Usage | Namespace IRI |
|---|---|---|
| `xsd:` | XML Schema datatypes | `http://www.w3.org/2001/XMLSchema#` |
| `rdf:` | RDF syntax identifiers | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` |
| `rdfs:` | RDF Schema | `http://www.w3.org/2000/01/rdf-schema#` |
| `skos:` | Simple Knowledge Organization System | `http://www.w3.org/2004/02/skos/core#` |
| `owl:` | Web Ontology Language | `http://www.w3.org/2002/07/owl#` |

An RDF **graph** is a set of triples, visualized as a directed node-arc-node graph: `(subject) -[predicate]-> (object)`.

## 4. Literals and Datatypes

A literal can appear **only as an Object** - never as subject or predicate:

| Kind | Example | Notes |
|---|---|---|
| Simple literal | `"Annie Hall"` | defaults to `^^xsd:string` |
| Language-tagged | `"Io e Annie"@it` | datatype `rdf:langString` |
| Typed literal | `"1978"^^xsd:gYear`, `"91"^^xsd:integer` | XML Schema 1.1 types; value via lexical-to-value mapping |
| Direction-tagged (RDF 1.2) | `"Io e Annie"@it--ltr` | datatype `rdf:dirLangString`, direction `ltr`/`rtl` |

### Elementary typing: `rdf:type`

Asserts a resource is an instance of a class; abbreviable as **`a`** in Turtle:

```turtle
dbpedia:AnnieHall a schema:Movie .
```

## 5. Blank Nodes

Complex structures (e.g. a postal address with street, number, city, country) often need no global URI for intermediate nodes.

- **Definition**: an *anonymous resource* without a URI - a structural connector, locally referencable only.
- **Notation**: `_:name` (e.g. `_:silvelloAddress`).
- **Scope**: identifiers are local to a single graph serialization; not part of the abstract RDF graph. They express *existence* without asserting an unknown IRI.
- **Position**: blank nodes may appear only as **subject** or **object**.

```turtle
:G_Silvello       :hasAddress       _:silvelloAddress .
_:silvelloAddress :street           "Via Aldo Rossi" .
_:silvelloAddress :number           12 .
_:silvelloAddress :city             :Padova .
_:silvelloAddress :country          :Italy .
```

## 6. Reification: Metadata About Statements

Statements often carry metadata (validity, license, price, provenance), e.g. *"Irrational Man is streamed by Amazon Prime for 1.99 EUR on June 4, 2021"*:

1. **Conventional reification** - treats the statement as an instance of a triple; complex, avoid when plain event modeling works.
2. **Event modeling** - model the event/transaction as the primary resource (e.g. `:stream/e34`).
3. **RDF 1.2 / RDF-star** - attach metadata to a *reifier* linked directly to a triple via **`rdf:reifies`** (modern approach).

## 7. Graphs and Datasets (RDF 1.1+)

- **Named graph**: an RDF graph paired with an explicit name (IRI or blank node).
- **RDF dataset**: exactly **one unnamed default graph** (may be empty) + **zero or more named graphs**, each with a graph name.
- **SPARQL integration**: graph names let queries target specific subsets. SPARQL 1.1 allowed only IRIs as graph names; **SPARQL 1.2 also allows blank nodes**.
- Terminology:
  - **Ground graph** - no blank nodes.
  - **Subgraph** - a subset of a graph's triples.
  - **Graph instance** - obtained by substituting blank nodes with literals, IRIs, or other blank nodes via a mapping $M$.

## 8. Serialization Formats

| Format | Character |
|---|---|
| **N-Triples** | line-by-line raw triples (canonical lowest level) |
| **Turtle / N3** | compact, human-readable; prefixes and shortcuts |
| **JSON-LD** | JSON-based, for web integration |
| **RDF/XML** | XML syntax for legacy toolchains |

### N-Triples rules

1. Every URI fully unabbreviated, in angle brackets `<...>`.
2. Strict `Subject Predicate Object` ordering.
3. Every statement terminated by a period `.`.

```text
<http://w3id.org/stream/e34> <http://purl.org/comm/product> <http://dbpedia.org/page/Irrational_Man> .
<http://w3id.org/stream/e34> <http://purl.org/dc/terms/date> "2021-06-04"^^<http://www.w3.org/2001/XMLSchema#date> .
<http://dbpedia.org/page/Netflix> <http://purl.org/comm/stream> <http://w3id.org/stream/e298> .
```

## 9. Exam Hooks

- The **iff-URI graph-merging rule** is the one-sentence answer to "how does RDF integrate distributed data?".
- Position rules are strictly asymmetric: literals only as objects; predicates always URIs; blank nodes only subject/object.
- CURIE = prefix + colon + reference - the notational bridge between verbose URIs and readable serializations.
- Know which feature arrived when: RDF 1.1 (datasets/named graphs), RDF 1.2 / RDF-star (`rdf:reifies`, direction tags, blank-node graph names).
- N-Triples is the reference serialization for the abstract model; Turtle is its human-friendly abbreviation.

## Related

- [[Linked Open Data]] - publication principles and the 5-star schema built on RDF
- [[Graph Databases]] - course hub
- [[LOTUS Semantic Operators]] - knowledge-graph query layer
- [[Database Foundations and Architecture]] - cross-course: the database models this course extends
- [[OWL SKOS Punning Football Ontology]] - merged lecture note covering OWL, SKOS, punning, football ontology

## Sources

- [[RDF]] (AY 2026/2027)
