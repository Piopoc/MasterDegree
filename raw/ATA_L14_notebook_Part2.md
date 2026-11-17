# Advanced Text Analytics — Lecture 14, Part II
# From Legal Entities to a Legal Knowledge Graph: GLiNER, ELI, SKOS and EuroVoc

*Lecture notes accompanying the notebook `ATA_L14_Realistic_Legal_KG_GLiNER_ELI_EuroVoc.ipynb`*

---

## 0. Purpose and learning objectives

Part I treated legal IE as a problem of *recognition and parsing*: finding spans, typing them, and recovering their internal structure. Part II takes the next step and asks **what the output of extraction should become**. The notebook's answer is a small **legal knowledge graph (KG)** in RDF, built on established semantic-web vocabularies:

```text
text → spans → normalised resources → typed relations → queryable graph
```

At the end of this unit, the student should be able to:

1. Explain the division of labour between **open-label extraction** (GLiNER) and **deterministic citation parsing**, and why they complement each other rather than compete.
2. Describe the core of the **ELI ontology** (European Legislation Identifier): the classes `eli:LegalResource` and `eli:LegalResourceSubdivision` and the properties `eli:is_part_of`, `eli:has_part` and `eli:is_about`. Place them in the FRBR-inspired model ELI is based on.
3. Model a controlled legal vocabulary in **SKOS**, and explain how it differs from a flat label list (preferred and alternative labels, `broader`/`narrower`).
4. Distinguish **topic indexing** (annotating a legal resource with a subject concept) from **entity normalisation** in the biomedical sense.
5. Build, serialise (Turtle) and query (SPARQL) a small RDF graph with `rdflib`, and query an authoritative endpoint (CELLAR).
6. **Critically assess** the pipeline: identify where it is truly data-driven and where it is hand-wired, and where lexical normalisation fails.

---

## 1. The scenario

```text
In Case C-311/18, the Court of Justice examined a dispute involving Maximillian
Schrems and Facebook Ireland. The Court interpreted Article 46(1) of Regulation
(EU) 2016/679 in relation to transfers of personal data to third countries and
the requirement to provide appropriate safeguards.
```

The paragraph is a simplified version of the *Schrems II* judgment (C-311/18, ECLI:EU:C:2020:559). In that judgment the Court examined, among other things, the conditions under which personal data may be transferred to a third country on the basis of **appropriate safeguards** under Article 46 GDPR (in particular standard contractual clauses). The legal content is therefore realistic. In Part I the scenario was a stylised appeal; here it is genuine.

The objects to be represented are of four kinds:

| Kind | Instances | Natural source of knowledge |
|---|---|---|
| Legislation | GDPR, Article 46(1) | Authoritative registries (ELI, CELEX) |
| Case law | Case C-311/18 | Court registries (case number, ECLI) |
| Actors | Court of Justice, Schrems, Facebook Ireland | Authority tables / NER |
| Subject matter | data transfers, third countries, safeguards | Controlled vocabularies (EuroVoc) |

Each kind has a **different authority** behind its identifiers. Designing a legal KG therefore largely comes down to deciding *whose identifiers to reuse for what*.

---

## 2. Authoritative identifiers: ELI and CELEX

```python
LEGAL_RESOURCE_REGISTRY = {
    "Regulation (EU) 2016/679": {
        "title": "General Data Protection Regulation",
        "eli_uri": "http://data.europa.eu/eli/reg/2016/679/oj",
        "celex": "32016R0679", ...
    }
}
```

The GDPR has two real identifiers:

- **ELI** `http://data.europa.eu/eli/reg/2016/679/oj`: a *semantic*, human-readable HTTP URI. Its segments encode the act type (`reg`), year, number and the version published in the Official Journal (`oj`). ELI is a cross-European standard, adopted by the EU and several Member States, for identifying legislation with persistent URIs.
- **CELEX** `32016R0679`: the EUR-Lex document number. Its parts are *sector* (`3` = legislation), *year* (`2016`), *document type* (`R` = regulation) and *number* (`0679`).

The registry is a dictionary keyed by the **surface string** of the citation. This is the simplest form of **dictionary-based entity linking**: an exact-match lookup from mention to resource. It works because §3 of Part I has already brought the mention into canonical form. Note the dependence: *the linker's recall is bounded by the canonicalising power of the recogniser*. If the text said "the GDPR" or "Regulation 2016/679", the lookup would fail. A realistic linker therefore normalises the mention to a structured key first (`{type: reg, year: 2016, number: 679}`) and then resolves the key, not the raw string.

---

## 3. The ELI ontology

### 3.1 Conceptual model

The ELI ontology (namespace `http://data.europa.eu/eli/ontology#`) follows the **FRBR** model (*Functional Requirements for Bibliographic Records*), which separates levels of abstraction:

| FRBR level | ELI class | Meaning in law |
|---|---|---|
| Work | `eli:LegalResource` | The act as an abstract legal object (the GDPR) |
| Expression | `eli:LegalExpression` | A linguistic realisation (the English text of the GDPR) |
| Manifestation | `eli:Format` | A physical/digital format (PDF, HTML, Formex XML) |

`eli:LegalResourceSubdivision` is a subclass of `eli:LegalResource`. It represents structural units (articles, paragraphs, annexes). They are linked to the whole by `eli:is_part_of` and its inverse `eli:has_part`. The resulting **mereological hierarchy** is the natural model of the internal structure recovered by the provision parser in Part I:

```text
eli:LegalResource                   GDPR
      ▲ eli:is_part_of
eli:LegalResourceSubdivision        Article 46
      ▲ eli:is_part_of
eli:LegalResourceSubdivision        Article 46(1)
```

`eli:is_about` links a legal resource to its **subject matter**, expressed as a `skos:Concept`. This property is the point where ELI and controlled vocabularies such as EuroVoc meet.

### 3.2 Modelling choices in the notebook

1. **Real URI for the act, local URI for the provision.** The GDPR node uses its canonical ELI. The node for Article 46(1) uses `https://example.org/legal-resource/gdpr/article/46/paragraph/1`. This is methodologically honest: one should not *mint* URIs in a namespace one does not control, because that claims an authority one does not have. The node is still typed with the real class `eli:LegalResourceSubdivision`, so it is semantically interoperable while staying clear about where its identifier comes from.
2. **A flattened hierarchy.** Article 46(1) is linked *directly* to the GDPR. There is no intermediate `Article 46` node. This is acceptable for a toy graph, but it loses the ability to ask "which paragraphs of Article 46 are interpreted in case law?". A faithful model would create one node per level of the parse (`article`, `paragraph`, `point`) and chain them with `eli:is_part_of`.
3. **Only one direction.** Only `eli:is_part_of` is asserted. `eli:has_part` is listed among the terms used but never materialised. Without an OWL reasoner, SPARQL queries over `has_part` will return nothing. One should either materialise both directions (as the notebook does for `skos:broader`/`skos:narrower`) or use SPARQL property paths (`^eli:is_part_of`).
4. **`dcterms:title` vs `eli:title`.** The notebook attaches `dcterms:title` to the `LegalResource`. In the ELI model, titles are language-specific and therefore properties of the *expression* (`eli:title`, whose domain is `eli:LegalExpression`). Likewise the local identifier would more idiomatically be `eli:id_local`. These simplifications are reasonable for teaching, but in an exam answer students should be able to say *why* the title belongs at the expression level: titles differ between the 24 official languages, and the work does not.

---

## 4. SKOS: controlled vocabularies as graphs

### 4.1 Why SKOS

**SKOS** (*Simple Knowledge Organization System*, a W3C Recommendation) is the standard RDF model for thesauri, classification schemes and taxonomies. It is deliberately *weaker* than OWL. Its concepts are units of indexing, not classes with formal semantics, and `skos:broader` is **not** `rdfs:subClassOf`. This weakness is a feature: it fits subject vocabularies, where "broader" often means "more general topic" rather than a strict set inclusion.

The constructs used by the notebook:

| Construct | Role | Example |
|---|---|---|
| `skos:Concept` | Unit of meaning | *international transfer of personal data* |
| `skos:prefLabel` | Canonical label (at most one per language) | `"international transfer of personal data"@en` |
| `skos:altLabel` | Synonyms, variants, abbreviations | `"cross-border data transfer"@en` |
| `skos:broader` / `skos:narrower` | Hierarchical relation (mutual inverses) | *data transfer* → *data protection* |

**EuroVoc**, the multilingual thesaurus maintained by the EU Publications Office, is published in SKOS. It is used to index EU documents in EUR-Lex. The notebook builds a five-concept local stand-in under `example.org`, and says plainly that these are *not* EuroVoc URIs. Again, this is correct practice.

### 4.2 The local hierarchy

```text
fundamental rights
   └── protection of personal data
          └── international transfer of personal data
                 └── appropriate safeguards
third country                         (no broader concept)
```

Placing *appropriate safeguards* as narrower than *international transfer* is a domain-specific modelling choice. Within the GDPR, appropriate safeguards are a transfer mechanism (Chapter V). In a general legal thesaurus the concept could reasonably sit elsewhere, because safeguards appear in many regimes. This is a reminder that **a vocabulary encodes a point of view**. The hierarchy is an *engineering artefact*, not a fact about the world.

`broader_chain` walks the hierarchy upward. It keeps a `visited` set to prevent infinite loops on cyclic data (SKOS does not forbid cycles, and real thesauri sometimes contain them), and it takes `broader[0]` when a concept has several parents. That second choice is a silent simplification: SKOS allows **poly-hierarchy**, and EuroVoc contains it. A complete implementation would return a DAG of ancestors, or simply use the SPARQL property path `skos:broader+`.

---

## 5. Two extraction regimes

### 5.1 Deterministic citation extraction

The citation regexes (case number, legal act, provision) are the same as in Part I, with one change: `LEGAL_ACT_PATTERN` is narrowed to `Regulation (EU)`. The parsed provision `{article: "46", paragraph: "1", point: None}` is the structural input to the ELI hierarchy. In principle it is what *should* produce the provision URI (see §8).

### 5.2 Open-label extraction with GLiNER

GLiNER is asked for `court`, `person`, `organization`, `legal topic` and `legal action`, with threshold 0.45. The design statement in the notebook is the important part: **GLiNER is not asked to replace the citation regexes**. The reasoning is as follows.

- For **actors**, the label is a coarse semantic type and the set is open. A generalist, label-conditioned span model is the right tool.
- For **citations**, the model can find the span, but the useful output is the *parse* and the *identifier*. A probabilistic span score adds nothing there except the risk of boundary errors.

### 5.3 An empirical observation: the "legal topic" label

The notebook's own added comment reports that **GLiNER returned no entities labelled `legal topic`**. The original topic pipeline, which relied on that label, therefore produced an empty table. The result is instructive, not accidental:

1. *Legal topic* is an **abstract, non-referential category**. GLiNER's training data (synthetic NER annotations over web text) consists mainly of *named* or at least *concrete* mentions. Subject-matter descriptors such as "transfers of personal data to third countries" are noun phrases that **describe** a theme rather than **refer to** an individual. Most NER models are not trained to treat these as entities.
2. Label-conditioned models are **sensitive to label wording**. `"legal topic"`, `"legal concept"`, `"area of law"` or `"data protection concept"` can give very different recall. Prompt-like label engineering is part of using such models, and it should be evaluated systematically, not tuned by eye.
3. More fundamentally, subject indexing is usually a **document- or passage-level classification** task (multi-label classification into EuroVoc descriptors, as in the EURLEX57K / MultiEURLEX benchmarks), not a span-extraction task. The notebook's difficulty is a symptom of applying a span-level tool to a problem that is largely about aboutness.

The repair the notebook adds, `find_topic_mentions`, scans the text for exact occurrences of any SKOS label. This is **dictionary-based concept spotting**, the same principle as `EntityRuler`, but driven by the thesaurus. It is a sensible fallback, and it makes topic detection independent of GLiNER's recall. As the next section shows, however, it has its own strong limits.

---

## 6. Topic normalisation: a critical analysis

### 6.1 The scoring function

For a surface string $s$ and a concept $c$ with labels $L(c)$, `normalize_topic` computes

$$
\text{score}(s, c) = \max_{\ell \in L(c)} \frac{|W(\ell) \cap W(s)|}{|W(\ell)|},
$$

where $W(\cdot)$ is the set of lower-cased word tokens. This is the **coverage of the label by the mention**: the share of label words that appear in the surface string. The concept with the highest score is returned.

### 6.2 What actually happens on the running example

Running the notebook's code on the scenario sentence gives the following.

**Step 1: spotting.** `find_topic_mentions` returns

```text
["third countries", "appropriate safeguards", "safeguards"]
```

The phrase that carries the *main* legal content of the sentence, "transfers of personal data", matches **no** label. The labels are `international transfer of personal data` and `data transfer`. The plural *transfers*, the missing *international*, and the word order all prevent an exact substring match.

**Step 2: normalisation.** Each mention maps to *third country* or *appropriate safeguards* with score 1.0. *appropriate safeguards* appears twice (from the full label and from the alt-label "safeguards"). This is harmless in the RDF graph, because a graph is a *set* of triples, but it is redundant in the tabular output.

**Consequence.** The graph asserts

```text
Article 46(1)  eli:is_about  <third-country>
Article 46(1)  eli:is_about  <appropriate-safeguards>
```

but **not** `eli:is_about <data-transfer>`. The concept the notebook's narrative highlights ("Article 46(1) is about international transfer of personal data", §§14, 22) appears in the graph only *indirectly*, as the `skos:broader` of *appropriate safeguards*. The broader-topics query in §21 happens to show it for that reason. Students should notice the gap between the **story told in the markdown** and the **triples actually produced by the code**. Checking this kind of gap is an essential part of evaluating any IE pipeline.

### 6.3 What would have happened with the original fallback

Had the hand-written fallback been used (`RUN_GLINER = False`), the mention "transfers of personal data to third countries" would have been scored as follows:

| Concept | Best label | Overlap | Score |
|---|---|---|---|
| third country | "third countries" | {third, countries} | **1.00** |
| protection of personal data | "protection of personal data" | {of, personal, data} | 0.75 |
| international transfer of personal data | "international transfer of personal data" | {of, personal, data} | 0.60 |

The phrase would be normalised to **third country**, the *modifier*, not the *head* of the noun phrase. Three weaknesses of the scorer appear together:

1. **Normalisation by label length favours short labels.** A two-word label that is fully covered beats a five-word label that is mostly covered. The measure is asymmetric: it ignores how much of the *mention* is left unexplained. Symmetric measures (Jaccard, Dice, or an F-measure combining label coverage and mention coverage) penalise this.
2. **No morphological normalisation.** *transfers* ≠ *transfer*, so the correct concept loses its most informative word. Lemmatisation or stemming would fix this directly.
3. **Stop words count as evidence.** *of* contributes to the overlap. Without a stop-word filter or term weighting (IDF), function words inflate the scores for every multi-word label.

### 6.4 No rejection threshold

`normalize_topic` *always* returns the best concept, however low its score. In our test, the string "Court of Justice" maps to *protection of personal data* with score 0.25, purely because of the word *of*. A normalisation component must be able to **abstain** (the NIL problem in entity linking). The standard approach is a calibrated threshold below which the mention is not linked, or is flagged for human review.

### 6.5 Better approaches

In increasing order of sophistication:

1. **Lexical, but principled:** lemmatise, remove stop words, weight terms by IDF over the thesaurus, use a symmetric similarity, and apply a rejection threshold.
2. **Dense retrieval:** embed mentions and all SKOS labels (pref + alt, in all languages) with a sentence encoder, retrieve the top-*k* concepts by cosine similarity, and optionally re-rank with a cross-encoder. This is the bi-encoder / cross-encoder paradigm of modern entity linking (cf. BLINK; SapBERT in biomedicine).
3. **Passage-level indexing:** treat *aboutness* as multi-label classification of the provision or paragraph into EuroVoc descriptors, trained on EUR-Lex documents, which are already indexed by the Publications Office.

### 6.6 Topic indexing versus biomedical normalisation

The notebook stresses that topic normalisation "is not the same as biomedical concept normalisation". The distinction can be made precise:

| | Biomedical normalisation | Legal topic indexing |
|---|---|---|
| Question answered | *What does this mention refer to?* | *What is this resource about?* |
| Linked object | The **mention** (a span) | The **resource** (a provision, a judgment) |
| Relation | Identity / reference (`mention → CUI`) | Aboutness (`eli:is_about`) |
| Cardinality | Typically one concept per mention | Several concepts per resource |
| Evidence | Local (the span and its context) | Global (the whole passage) |

So in the KG the subject of `eli:is_about` is the *provision*, not the text span. The span is only evidence for the annotation. A complete model would keep that evidence: annotation provenance using the W3C Web Annotation or PROV-O vocabularies (*which span, which component, which score* produced the triple).

---

## 7. Building the graph

### 7.1 Vocabulary reuse and local extension

The graph combines:

- **standard vocabularies** where they fit: `eli:` for legislation, `skos:` for topics, `dcterms:` for identifiers and titles, `rdfs:label` for display labels;
- a **local namespace** `ex:` for what ELI does not cover. ELI is a *legislation* ontology and has no classes for cases, courts or parties.

This applies a basic principle of ontology engineering, quoted in the notebook: *reuse standard vocabularies where they fit, extend locally where they do not*. In a production system the local classes would themselves be replaced or aligned. Candidates include the **ECLI** metadata model for case law, the **EU authority tables** of the Publications Office for institutions (the Court of Justice has an authority-table URI), the **CDM** (*Common Data Model*, the ontology behind CELLAR), and **schema.org / FOAF / W3C ORG** for persons and organisations.

### 7.2 The resulting structure

```text
ex:case/C-311-18  a ex:LegalCase ;
    dcterms:identifier "C-311/18" ;
    ex:hasCourt   ex:court/CJEU ;
    ex:hasParty   ex:person/maximillian-schrems , ex:organization/facebook-ireland ;
    ex:interprets <…/gdpr/article/46/paragraph/1> ;
    ex:concerns   <http://data.europa.eu/eli/reg/2016/679/oj> .

<…/gdpr/article/46/paragraph/1>  a eli:LegalResourceSubdivision ;
    eli:is_part_of <http://data.europa.eu/eli/reg/2016/679/oj> ;
    dcterms:identifier "Article 46(1)" ;
    eli:is_about <…/third-country> , <…/appropriate-safeguards> .
```

Two observations link back to Part I:

1. **`ex:hasParty` has no role.** Part I argued that roles are relations between a party and a proceeding. The KG is the natural place to model them properly. Either use role-specific properties (`ex:hasApplicant`, `ex:hasDefendant`), or *reify* the participation as an n-ary relation (`ex:case ex:hasParticipation [ ex:party ex:schrems ; ex:role ex:Applicant ]`). The second option is more flexible and is the standard RDF pattern for qualified relations.
2. **`ex:interprets` is a genuinely legal relation.** It is not stated by any NER label. It comes from the predicate *interpreted* in the text. Populating it automatically is a **relation extraction** problem, the subject of the next lectures.

### 7.3 Serialisation

The Turtle serialisation (`g.serialize(format="turtle")`) is the human-readable view of the graph. Students should read it and check that every triple can be traced to its source: text, regex, registry, vocabulary, or hand-written code.

---

## 8. Which parts are extracted, and which are hand-wired?

The notebook is honest about being a teaching pipeline. Even so, it is a valuable exercise to separate what is **data-driven** from what is **hard-coded**:

| Graph element | Origin in the notebook |
|---|---|
| GDPR URI, title, CELEX | Registry lookup driven by the regex match ✔ |
| Provision URI | **Hard-coded** string (not built from `parsed_provision`) |
| Provision identifier `"Article 46(1)"` | **Hard-coded** literal (not taken from the match) |
| Case URI and identifier | **Hard-coded** (not taken from `CASE_PATTERN`) |
| Court, persons, organisations | **Hard-coded** URIs and labels (GLiNER output not used) |
| `hasParty`, `hasCourt`, `interprets`, `concerns` | **Hard-coded** assertions |
| `eli:is_about` topics | Driven by the spotting and normalisation pipeline ✔ |

So the text drives only two links in the chain: act grounding and topic annotation. Turning the notebook into a genuine *text-to-graph* system means replacing every hard-coded element with a function of the extraction output:

- URIs minted deterministically from parsed records (e.g. a slug of the case number; a path built from article/paragraph/point);
- actor nodes created from GLiNER spans and deduplicated by an entity-resolution step;
- relations produced by a relation-extraction component, with provenance and confidence.

This is the gap between a **knowledge graph *schema* populated by hand** and a **knowledge graph *population* pipeline**.

---

## 9. Querying the graph

### 9.1 Join query (notebook §20)

```sparql
SELECT ?caseId ?provisionId ?topicLabel WHERE {
  ?case a ex:LegalCase ; dcterms:identifier ?caseId ; ex:interprets ?provision .
  ?provision dcterms:identifier ?provisionId ; eli:is_about ?topic .
  ?topic skos:prefLabel ?topicLabel .
}
```

This query **joins three kinds of knowledge**: case law (`ex:interprets`), legislation (`dcterms:identifier` of the provision), and the vocabulary (`skos:prefLabel`). A flat list of spans cannot answer it. That is the main argument for the graph representation. On the actual data it returns two rows, *third country* and *appropriate safeguards*, for the reasons given in §6.2.

### 9.2 Hierarchy query (notebook §21)

The `OPTIONAL { ?topic skos:broader ?broader . … }` pattern returns each topic with its *immediate* broader concept, and `None` where there is none (*third country*). Two extensions are worth practising:

```sparql
# all ancestors, at any depth (SPARQL 1.1 property path)
?topic skos:broader+ ?ancestor .

# retrieval with query expansion: provisions about data protection or anything narrower
?provision eli:is_about/skos:broader* <https://example.org/legal-topic/data-protection> .
```

The second pattern is **hierarchical query expansion**. A provision indexed only with *appropriate safeguards* is retrieved by a search for *data protection*. This is the practical payoff of SKOS for legal information retrieval, and it is how EuroVoc-based search works in EUR-Lex.

---

## 10. Live EuroVoc via CELLAR

CELLAR is the Publications Office's semantic repository. Its public SPARQL endpoint (`https://publications.europa.eu/webapi/rdf/sparql`) exposes EU legal metadata and the EU controlled vocabularies, EuroVoc included. The optional query looks for SKOS concepts whose English `prefLabel` contains *personal data*.

Methodological comments:

1. **Scope the query to a scheme.** As written, the query scans *every* `skos:Concept` in the endpoint, across all vocabularies and authority tables. That is slow, may hit server timeouts, and may return concepts from schemes other than EuroVoc. Adding `?concept skos:inScheme <EuroVoc scheme URI>` makes it faster and more precise.
2. **Search alt-labels too.** Restricting the search to `skos:prefLabel` misses entry terms (non-preferred terms). In EuroVoc these are a large part of the lexical coverage. A union over `skos:prefLabel | skos:altLabel` is the usual pattern.
3. **`CONTAINS(LCASE(...))` is a full scan.** Endpoints based on Virtuoso usually provide a full-text index (`bif:contains`), which is much faster. This is not portable, however.
4. **Reproducibility.** Results "depend on the current EuroVoc data", as the notebook warns. For a reproducible experiment, either fix the version of the vocabulary (EuroVoc is versioned) or cache the response alongside the notebook.

This is the step where the local teaching vocabulary would be **aligned** with the authoritative one, either by replacing the `example.org` URIs or by asserting mappings such as `skos:exactMatch` and `skos:closeMatch`. Alignment through SKOS mapping properties keeps local concepts that EuroVoc lacks, while still making the graph interoperable.

---

## 11. Beyond NER: LOMO and normative content

The notebook ends with a pointer to the **Legal Obligation Metadata Ontology (LOMO)**, which models the *interpreted normative content* of provisions: obligations, permissions, prohibitions, rights, definitions and scope provisions, linked back to their ELI sources. This sets out the research programme that the present pipeline is the first step of:

```text
NER / provision extraction        (what is cited?)
        ↓
relation extraction               (who interprets / amends / applies what?)
        ↓
normative rule extraction         (what does the provision require, of whom, under what conditions?)
        ↓
formal legal knowledge representation  (deontic modelling, compliance checking)
```

Each step moves from *surface* to *meaning*. Each also increases the share of interpretation that is legally contestable. That is why human-in-the-loop validation and explicit provenance become *more* important, not less, as the pipeline matures.

The roadmap in notebook §25 lists natural extensions: authoritative EuroVoc URIs, authority tables, ECLI normalisation, CELEX/ELI resolution, multilinguality, temporal versioning (consolidated versions, points in time), and the ELI relations `eli:cites`, `eli:amends`, `eli:repeals` and `eli:based_on`. Of these, **temporal versioning** deserves particular emphasis. A legal provision is not a fixed object: Article *n* "as amended on date *d*" is a different expression. A KG that ignores time will make incorrect legal statements.

---

## 12. Synthesis

```text
                    LEGAL TEXT
                         │
        ┌────────────────┴────────────────┐
  deterministic rules                GLiNER / learned NER
  case no. · act · provision         court · person · organisation
        │                                 │
        │                      dictionary concept spotting (SKOS labels)
        │                                 │
        └───────────────┬─────────────────┘
                  normalisation / linking
          ┌─────────────┴─────────────┐
     ELI / CELEX registry         SKOS / EuroVoc
          └─────────────┬─────────────┘
              relation assertion (→ relation extraction)
                        │
              RDF LEGAL KNOWLEDGE GRAPH  ──►  SPARQL
```

### Take-home messages

1. **The representation of the output is a design decision as important as the extractor.** A graph turns spans into connected, queryable knowledge.
2. **Reuse authoritative identifiers** (ELI, CELEX, ECLI, EuroVoc). Never mint URIs in namespaces you do not control. Extend locally where standards stop.
3. **ELI models legislation as a FRBR-style hierarchy of resources and subdivisions.** The structure recovered by provision parsing maps directly onto it.
4. **SKOS turns a label list into a navigable concept graph.** Hierarchical relations support query expansion and more general indexing.
5. **Topic indexing is about aboutness, not reference.** It attaches concepts to resources, and it is often better framed as classification than as span extraction.
6. **Lexical normalisation needs morphology, term weighting, symmetric scoring and a rejection option.** Without them it can confidently map a mention to the wrong concept, as the running example shows.
7. **Distinguish what the pipeline extracts from what it asserts by hand.** The step from a hand-populated schema to an automatic population pipeline is relation extraction, the subject of the following lectures.

---

## 13. Exercises

1. **Faithful hierarchy.** Build the provision URIs *from* `parsed_provision`, creating one `eli:LegalResourceSubdivision` node per level (article → paragraph → point). Materialise both `eli:is_part_of` and `eli:has_part`. Write a SPARQL query listing all subdivisions of the GDPR at any depth.
2. **Better topic normalisation.** Re-implement `normalize_topic` with lemmatisation, stop-word removal, a symmetric Dice score and a rejection threshold. Show that "transfers of personal data to third countries" now maps to *international transfer of personal data*, and that "Court of Justice" is rejected.
3. **Dense linking.** Replace the lexical matcher with a sentence-embedding retriever over all SKOS labels. Compare top-1 accuracy with the lexical version on 30 hand-annotated mentions from real CJEU judgments.
4. **Label engineering.** Run GLiNER with alternative wordings of the topic label (`"legal concept"`, `"area of law"`, `"data protection concept"`, `"legal subject matter"`) and thresholds 0.3–0.6. Report recall against a small gold set, and discuss whether span extraction is the right formulation for this task.
5. **Qualified roles.** Remodel `ex:hasParty` as an n-ary participation with an explicit role (`Applicant`, `Defendant`, `Intervener`, `Referring court`). Write a query returning every party with its role in Case C-311/18.
6. **EuroVoc alignment.** Run the CELLAR query restricted to the EuroVoc scheme and including `skos:altLabel`. Align each local concept with the closest EuroVoc concept using `skos:exactMatch` or `skos:closeMatch`, and justify each choice.
7. **Provenance.** For every `eli:is_about` triple, record which text span, which component (spotter / GLiNER) and which score produced it, using PROV-O or the Web Annotation vocabulary. Write a query that returns only annotations with score ≥ 0.8.
