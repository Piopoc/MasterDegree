---
type: course-note
date: 2026-10-10
tags:
  - course
  - graph-databases
  - linked-open-data
course: "[[Graph Databases]]"
sources:
  - "[[LOD]]"
ai-first: true
---

## For Future Agent

Study note for the [[Graph Databases]] course on **Linked Open Data (LOD)** - how Open Data (legal openness) and Linked Data (technical interlinking) combine to turn the Web of Documents into a machine-readable Web of Data. Created 2026-10-10 from [[LOD]], an English briefing on Prof. Gianmaria Silvello's course materials. Key points: the three publication rules and Berners-Lee's four design principles, the URI/URL/resource distinction, HTTP content negotiation, hash vs. slash URI design, and the 5-star deployment schema.

## 1. Open Data: Definition and Licensing

Open Data is data that can be freely **used, reused, and redistributed** by anyone. The only permissible usage conditions:

- **Attribution** - users must credit the original publisher.
- **Sharealike** - derivative works blending open data with other datasets must be published under the same open terms.

To qualify as Open Data, a dataset **must** carry an explicit open license declaring these permissions.

### History and Benefits

- **1942** - Robert K. Merton argued for the scientific necessity of open data (researchers contribute to a collective knowledge pool).
- **1995** - the term "Open Data" first appears in a US scientific-agency document on international scientific-information exchange.
- **2007-present** - technical advances make open-data publishing and consumption viable globally at scale.

Benefits: innovation and commerce (new products/services), personal decision-making and civic participation, governmental efficiency (better services, lower operational costs).

## 2. Open Data vs. Linked Data vs. LOD

| | Open Data | Linked Data | Linked Open Data (LOD) |
|---|---|---|---|
| **What** | Legal openness (license) | Technical interlinking (Web standards) | Both |
| **Can exist alone?** | Yes - e.g. unlinked CSV dumps | Yes - e.g. linked enterprise data behind paywalls | - |
| **Distribution** | Static file dumps or live web services | Static file dumps or live web services | Data designed for interlinking |

## 3. The Semantic Web

- **1956** - Richard H. Richens invents "Semantic Nets" for computer applications.
- **Early 1960s** - Collins, Quillian, and Loftus introduce the formal Semantic Network Model.
- **Tim Berners-Lee** coins **"Semantic Web"**: an extension of the Web where data can be processed directly and indirectly by machines - a transition from *Web of Documents* (hyperlinked pages) to *Web of Data*.

### Three Rules for Data Publication

1. **Use HTTP URIs to name everything** - names must support direct lookup over standard HTTP access.
2. **Provide descriptive information** - on URI lookup, the server returns descriptive data in Web standard languages, specifically **RDF** (see [[RDF Data Model]]).
3. **Include recursive links** - servers embed HTTP URIs to other entities so clients and crawlers can discover related resources recursively.

### Berners-Lee's Four Design Principles of Linked Data

1. Use **URIs** as names for things.
2. Use **HTTP URIs** so people and systems can look up those names.
3. Provide useful information using standards (**RDF**, **SPARQL**) upon lookup.
4. Include **links to other URIs** to facilitate further discovery.

## 4. Identifying Things: URI, URL, Resource

A core architectural requirement: distinguish the *real-world entity*, its *digital representation*, and its *URI identifier* - the "pipe" lesson, *"Ceci n'est pas un chat"*:

- **Real-world object**: a physical cat.
- **Representation**: a photo or web page depicting the cat (not the cat).
- **Identifier**: `http://the.name/of/my/cat` (an abstract identifier, not the cat).

Identifier examples:

| Entity | URI |
|---|---|
| City of Padua | `https://dbpedia.org/page/Padua` |
| Protein B6J853 | `https://www.uniprot.org/uniprot/B6J853` |
| Lab specimen (mouse strain) | `http://ohsu.eagle-i.net/i/0000013c-...` |
| UNICEF | `http://publications.europa.eu/resource/authority/corporate-body/UNICEF` |
| Belgium | `http://publications.europa.eu/resource/authority/country/BEL` |

**DNS guarantee**: HTTP URIs rely on the Domain Name System - a decentralized hierarchical naming system ensuring that any client anywhere resolving a given HTTP URI is directed to the exact same place.

## 5. Content Negotiation (`conneg`)

An HTTP-standard mechanism: clients state their preferred representation format via request headers, so humans and software agents can access the *same URI*:

| Client | Receives |
|---|---|
| Web browser (human) | HTML rendering |
| Software agent / crawler | RDF/XML, Turtle (`.ttl`), JSON-LD |

**Case study - UniProt** `https://www.uniprot.org/uniprot/P43121` (cell-surface glycoprotein MUC18):

- Browser lookup -> rich HTML page (protein function, gene `MCAM`, organism *Homo sapiens*).
- Machine lookup -> redirects to `.../P43121.ttl`, returning pure Turtle RDF.

## 6. URI Design: Hash (`#`) vs. Slash (`/`)

| Dimension | Hash URIs (`#`) | Slash URIs (`/`, 303 redirect) |
|---|---|---|
| Example | `http://www.w3id.org/people#silvello` | `http://www.w3id.org/people/silvello` |
| Mechanism | Fragment identifier: client trims `#...` and requests the base URI | HTTP `303 See Other` redirects resource URI -> location URL with the description |
| Redirect needed | **No** | **Yes** (303) |
| Retrieval | Downloads/caches the whole document with all fragment descriptions | Retrieves only the requested description (modular) |
| Drawback | Whole-file download inefficient for massive datasets | Doubles HTTP calls per lookup; fragments collection retrievals |
| Best for | Small datasets, vocabularies, schemas, ontologies | Large datasets, entity databases, scalable knowledge graphs |

## 7. RDF and SPARQL (overview)

RDF is the W3C standard syntax for modeling Web data: all information decomposes into atomic **triples** $\langle \text{subject}, \text{predicate}, \text{object} \rangle$ (detailed in RDF Data Model). **SPARQL** is the standardized query language for RDF triple stores and graph databases; queries can become highly complex when navigating multi-hop graph patterns, filters, or cross-domain aggregations (e.g. Wikidata award histories, stale biological-ontology tags).

## 8. The 5-Star Deployment Schema

Progressive technical maturity model for publishing Linked Open Data:

| Star | Stage | Requirement | Example formats |
|---|---|---|---|
| $\star$ | 1 | Available on the Web in **whatever format**, under an **open license** | PDF, JPG |
| $\star\star$ | 2 | **Machine-readable structured data** (not scans/images) | XLS, DOC |
| $\star\star\star$ | 3 | + **non-proprietary open formats** (no proprietary software needed) | CSV, XML |
| $\star\star\star\star$ | 4 | + **W3C open standards** (URIs, RDF, SPARQL) identifying entities down to data points | RDF, Turtle |
| $\star\star\star\star\star$ | 5 | + **outbound links to external datasets** for graph context (Linked RDF) | Linked-RDF / LOD |

## 9. Exam Hooks

- Open Data = *license* dimension; Linked Data = *technical* dimension; LOD = intersection - a dataset can be one without the other.
- The three publication rules and the four design principles are overlapping formulations of the same idea - know both lists.
- Hash vs. slash: the exam distinction is *fragment caching* (one request, whole document) vs. *303 redirection* (per-entity, scalable).
- Content negotiation is what makes the *same URI* serve humans and machines.
- The 5-star schema is a ladder: each star is a strict superset of the previous stage.

## Related

- [[RDF Data Model]] - the triple model LOD publishes
- [[LOTUS Semantic Operators]] - knowledge-graph applications
- [[Graph Databases]] - course hub
- [[Database Foundations and Architecture]] - cross-course: URI/data modeling vs. classical DB modeling

## Sources

- [[LOD]] (AY 2026/2027)
