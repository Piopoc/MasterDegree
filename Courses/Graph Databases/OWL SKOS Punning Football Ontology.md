---
type: course-note
date: 2026-10-25
tags:
  - course
  - graph-databases
course: "[[Graph Databases]]"
sources:
  - "[[lezione_persa_GDB]]"
ai-first: true
---

## For Future Agent

Single study note for the [[Graph Databases]] course covering the whole lecture transcription [[lezione_persa_GDB]] (English source, AY 2026/2027). It merges four topics that were previously split into separate notes, per user preference: OWL ontology foundations (`hasValue`, existential vs universal quantifiers, ADA 90), class vs instance modeling (classism, minimality, medical/histology example, data property vs subclass), SKOS (concept schemes, broader/narrower/related, labels, concept vs class, medical application), punning (TBox/ABox mixing), and the football ontology case study (FIFA + Transfer Market, competency questions, Appearance class, position taxonomy). Exam-oriented. Reworked from the raw lecture transcription left immutable in `raw/`.

## 1. OWL Ontology Foundations

### 1.1 Conformant Apps and the `hasValue` Constraint

The lecture opens with a software-compliance example:

- A **desktop app** is a subclass of a **conformant app**.
- A **conformant app** is an app conforming to a legislation.
- The legislation in question is **ADA 90** (Americans with Disabilities Act, 1990).
- If Excel is a desktop app, a reasoner infers it is a conformant app and therefore conforms to ADA 90.

In OWL this is expressed with the **`hasValue` constraint**: build an equivalent class of the conformant class and restrict the `conformsTo` property with `hasValue ADA-90`, where `ADA-90` is exactly one value. Conformance therefore means: the `conformsTo` property must have at least one value, and that value must be the specific legislation (ADA-90).

### 1.2 Existential vs Universal Quantifiers

`hasValue` is a specific case of the **existential quantifier**. The contrast:

| Quantifier | OWL construct | Meaning for `conformsTo` |
|---|---|---|
| **Existential** | `someValuesFrom` / `hasValue` | At least one value must exist on the property for an element of the class. |
| **Universal** | `allValuesFrom` | All values of the property are restricted to a given set. |

With `hasValue`, other conformed-to values may exist in principle, but one specific value (here ADA-90) is mandatory. If new legislation arrives (e.g. ADA-2019) you could technically add a second value, because `hasValue` accepts a set — but the professor advises aiming for exactly one value; the more values you add, the more purely "existential" the constraint becomes.

### 1.3 ADA 90 Compliance Example (Manchester syntax)

```
Individual: ADA-90
Type: Legislation

Class: ConformantApp
EquivalentTo: App and (conformsTo hasValue ADA-90)

Class: DesktopApp
SubClassOf: ConformantApp

Individual: Excel
Type: DesktopApp
```

A reasoner infers: **Excel is a ConformantApp** and **Excel conformsTo ADA-90**.

## 2. Class vs Instance Modeling

### 2.1 The "Classism" Problem

**Classism** is modeling everything with classes. The professor warns that when every concept is a class, instantiation forces you to create a new URI for every individual, which:

- creates too many individuals for the same concept,
- wastes resources,
- adds unnecessary complexity,
- produces redundant individuals referring to the exact same thing.

### 2.2 Minimality Principle

Model with the **minimum number of concepts** and generate the **minimum number of individuals**. Using many classes is not "wrong by definition," but correctness and minimality both matter; the right choice depends on the application requirements.

### 2.3 Medical/Histology Domain Example

Reality to model: a tissue is withdrawn from a patient, located in an anatomical location (lungs, pancreas, ...), and analyzed for a disease outcome (cancer, diabetes, polyp, ...).

**First attempt (class-based / classism):**
```
Class: TissueWithdrawnPatient
Class: AnatomicalLocation
  SubClassOf: Lung
    SubClassOf: UpperLung
    SubClassOf: LowerLung
Class: Disease
  SubClassOf: Polyp
  SubClassOf: LungCancer
  SubClassOf: LungCarcinoma
ObjectProperty: locatedIn
ObjectProperty: hasOutcome
```

This works but creates redundant individuals: one "lower lung" individual per patient and one "polyp" individual per patient, although the lower lung and the polyp are the same concept for everyone. Unless you store the specific cells from a specific polyp, you do not need a separate individual per patient.

### 2.4 Data Properties vs Subclasses

To model severity (good polyp / bad polyp / very bad polyp) there are two options:

**Option A — subclasses:**
```
Class: BadPolyp SubClassOf: Polyp
Class: GoodPolyp SubClassOf: Polyp
Class: VeryBadPolyp SubClassOf: Polyp
```

**Option B — data property:**
```
Class: Polyp
DataProperty: severity  (values: good, bad, very bad)
```

Which is correct depends on the application:
- **Merging graphs on the web** about polyps: model "good polyp" as a class, so you can merge all good polyps across datasets.
- **Managing a hospital:** you do not need per-patient good/bad polyp classes; put all polyps in one class and differentiate with a `severity` data property.

There is no always-applicable rule — only guidance to check which choice fits the use case.

### 2.5 When to Subclass vs When to Add Properties

Before creating a subclass, ask:

1. **Can you imagine an individual for this class?** If not, the class itself should probably be an individual.
2. **Can you imagine properties for the class?**
3. **Can you imagine specific values for those properties?**

A subclass should justify itself through useful instantiable individuals or specific distinguishing properties. Example: a literary author "Elisabetta Montos" may have specific properties general authors lack; "literary authors" vs "screenwriters" are overlapping sets.

## 3. General Ontology Modeling Guidelines

### 3.1 Questions to Ask Before Creating a Subclass

The professor's checklist:
1. Can I imagine an individual for this class? (If no, the class itself should be an individual.)
2. Can I imagine properties for the class?
3. Can I imagine specific values for those properties?

A subclass exists only if justified by instantiable individuals or specific properties that distinguish it from the superclass.

### 3.2 ER Modeling vs Ontology Modeling

Database professors initially treated ontology as "just a different formality" for Entity-Relationship modeling. It is not — especially at scale:

| Aspect | ER Modeling | Ontology Modeling |
|---|---|---|
| Generalization/specialization | Restructured in the logical model (tables merged or split) | No restructuring; no logical model |
| Conceptual → physical | 1:1 mapping does not hold; you restructure | Conceptual level is 1:1 with the physical level |
| Scale | Works well for small/medium models | Key differences emerge at very large scale |

What you model conceptually is not 1:1 with instances — that is the big difference.

## 4. SKOS — Simple Knowledge Organization System

### 4.1 What Is SKOS

**SKOS** = **Simple Knowledge Organization System**.

- A **data sharing standard** originating around 2009.
- Bridges different communities and practices (library science, information science, ...).
- Goal: connect communities through a shared organization of knowledge.
- The SKOS data model is formally defined as a **full ontology** — the highest level of expressiveness within its scope.
- **However, SKOS is not a formal knowledge-representation language.** It is a vocabulary of terms; each term has a specific meaning, and you assemble those terms into constructs.
- The underlying data model is **RDF** — always the same triple structure (see [[RDF Data Model]]).
- Compared to RDFS → OWL, SKOS adds more terms and more axioms, increasing complexity.

SKOS does not model only a hierarchy: it is a **graph** (typically a **thesaurus**) because you can relate siblings, not only parents and children.

### 4.2 Concept Scheme and Concepts

SKOS views a knowledge organization as a **concept scheme** comprising a set of **concepts**:

| Term | Meaning | Example |
|---|---|---|
| **Concept Scheme** | An organization of knowledge | Linnaean taxonomy of life; video-game genres |
| **Concept** | A unit within a scheme | "mammal"; "roguelike"; "action game" |

Example concept schemes:
- **Linnaeus taxonomy** — animal, plant, mammal, reptile, cow, ...
- **Video game anthology** — genres (platform, roguelike, Castlevania), platforms (PlayStation 5, PlayStation 4, Xbox, Switch).
- **Movie genres** — some derived from video-game genres.

Concepts can be labeled with lexical strings (one or more), assigned notations, linked to other concepts, and mapped across schemes (the same concept can appear in both a video-game scheme and a movie scheme).

### 4.3 skos:broader / skos:narrower

The two main **semantic (vertical)** properties:

- `skos:broader` — from a concept to its broader (parent) concept.
- `skos:narrower` — from a concept to its narrower (child) concept.

**Direction is counterintuitive:** "sheep `skos:broader` livestock" means sheep's broader concept is livestock, i.e. **livestock is broader than sheep** (livestock is the parent). Equivalently, "livestock `skos:narrower` sheep" means sheep is narrower than livestock (sheep is the child).

So:
- `A skos:broader B` → B is broader than A (B is the parent).
- `B skos:narrower A` → A is narrower than B (A is the child).

**Transitivity:** `skos:broader` and `skos:narrower` are **not transitive by default**. Transitive versions exist (`skos:broaderTransitive`, `skos:narrowerTransitive`) and are used only when you want inference over a chain. Naturally the relations are transitive (a cow is narrower than a mammal, a mammal narrower than an animal, so a cow narrower than an animal), but SKOS separates the plain property from the transitive one so you do not force inference everywhere.

### 4.4 skos:related

A **horizontal** relationship between concepts:
- **Symmetric**: if two concepts are related, they are related both ways.
- Example: "roguelike" related to "procedural generation" — associated, not parent-child.

### 4.5 skos:prefLabel / skos:altLabel / skos:inScheme

| Property | Meaning | Example |
|---|---|---|
| `skos:prefLabel` | Preferred label (plain literal, often language-tagged) | "love" @en |
| `skos:altLabel` | Alternative label | "adoration" @en |
| `skos:inScheme` | The concept scheme the concept belongs to | concept A in scheme S |

Lecture example:
- Individual `A` is of type `skos:Concept`.
- `A` has `skos:prefLabel` "love" @en.
- `A` has `skos:altLabel` "adoration" @en.
- `A skos:inScheme S`.
- `A skos:broader D`.
- `B` has `skos:prefLabel` "emotion"; "emotion" is broader than "love".

### 4.6 Concept vs Class vs Resource vs Thing

SKOS was built around **concept** because earlier RDF/RDFS vocabularies lacked a dedicated term for it:

| Term | Characteristic |
|---|---|
| **Class** | Has individuals/members; defined by extension (its members). |
| **Concept** | Has a **definition**; defined by meaning, not by members. |
| **Resource** | Everything in RDF is a resource (`rdfs:Resource` is the top). |
| **Thing / `owl:Thing`** | Everything is a subclass or sub-element of `owl:Thing`. |

A concept differs from a class, a resource, and a thing. **Key distinction:** a class is set-based (defined by its members); a concept has a definition independent of its instances. Example: the concept "roguelike" is a genre definition; the game "Dead Cells" is an instance of that concept — calling "Dead Cells" a class would be wrong.

### 4.7 SKOS Is Not a Formal Knowledge-Representation Language

Per the specification, SKOS is powerful and widely used (often without users realizing they are using RDF). You can use SKOS terminology (term, subordinate, superordinate) without knowing RDF. Used side by side with OWL, SKOS is not another language: it is the same language (RDF), just a different vocabulary.

### 4.8 SKOS Examples

**Taxonomy of life (Linnaeus):**
```
animal skos:broader livingThing
plant skos:broader livingThing
mammal skos:broader animal
reptile skos:broader animal
cow skos:broader mammal
```

**Video games:**
```
roguelike skos:broader videoGameGenre
platform skos:broader videoGameGenre
castlevania skos:broader videoGameGenre
deadCells skos:inScheme videoGameGenres
deadCells skos:broader roguelike
playstation5 skos:broader playstation
playstation5pro skos:broader playstation5
xbox skos:broader console
switch skos:broader console
```

### 4.9 SKOS Applied to Medical Modeling

The professor reuses the medical histology scenario (tissue withdrawn from a patient, located in an anatomical location, with a disease outcome) to show how SKOS avoids classism.

Instead of creating a separate class individual for every patient's lung and polyp, anatomical locations and diseases are modeled as **SKOS concepts**:

- `lung` is both an `AnatomicalLocation` and a `skos:Concept`.
- `upper lung` is both an `AnatomicalLocation` and a `skos:Concept`; `upper lung skos:broader lung`.
- `lower lung` is both an `AnatomicalLocation` and a `skos:Concept`; `lower lung skos:broader lung`.
- `lung cancer` is a `skos:Concept` and a `Disease`.
- `polyp` is a `skos:Concept` and a `Disease`.
- `lung carcinoma skos:broader lungCancer` (hierarchy).

Concepts carry labels (`skos:prefLabel` "lung" @en, "polmone" @it, ...) and broader/narrower relations. You can now point to the **same concept** (`lung`) for every patient instead of creating an individual `XYZ-34` per patient:

```
John a Patient.
TissueX a Tissue.
TissueX withdrawnFrom John.
TissueX locatedIn lung.
TissueX hasDisease polyp.
```

You do **not** need to instantiate a "polyp of patient John" — `polyp` is a shared concept. You only need a specific individual if you later store the actual cells from that specific polyp.

**Design pattern:** use classes for real, concrete entities (patients, tissues in jars); use SKOS concepts for shared, repeatable concepts (anatomical locations, diseases).

## 5. Punning

### 5.1 What Is Punning

**Punning** is using an **individual** as a **schema element** — implicitly mixing instances (ABox) with classes and properties (TBox). The professor encountered it in a large project in 2017, used it, then disliked it, and developed a different solution (the SKOS-based approach presented in Section 4).

### 5.2 TBox vs ABox Mixing

- **TBox** (Terminology Box): classes, properties, axioms — the schema.
- **ABox** (Assertion Box): individuals, assertions — the data.

You cannot normally cross the TBox/ABox boundary. Except via `rdf:type`, you cannot use `type` in certain contexts. Punning is the workaround: an individual that is actually used as a schema element because you always refer to it as if it were a class.

The professor's assessment: "It's a bad practice, I really don't like it, but in general it is not bad. But it's possible." This was the only way he had seen it written; he could not find it in any paper and is writing a chapter to document it.

### 5.3 Why It Is Considered Bad Practice

- You mix instances and schema implicitly — it is not evident.
- You have an individual used as a schema element.
- You cannot refer directly to the class because you cannot cross the TBox/ABox boundary (except via `rdf:type`).

## 6. Football Ontology Case Study

### 6.1 Requirements and Data Sources

The professor builds a complete football ontology for RDF ingestion and graph-database population.

**Data sources (open data, not yet linked data):**
- **FIFA video game database** (CSV: player characteristics, overall value, potential, ...) — data about 4–5 years old, so players are not current.
- **Transfer Market** website (player economic value, statistics).

**Goal:** transform open data into **linked data** by merging the two sources.

**Modeled domain:** football players with performance attributes, abilities, economic value and wage; club and league; games, teams, time, results, outcomes; single-player performances (goals, assists); assists as key passages; yellow and red cards.

**Competency questions** (queries the ontology should answer):
- Who are the top players by economic value and overall performance?
- Who received more red cards in the season?

> When you model an ontology, you need a big set of competency questions.

### 6.2 Classes and Properties

**Class: Person** (superclass)
- Data properties: first name, last name, birthday, height, weight, nationality (also a characteristic of a manager).

**Class: Player** (subclass of Person)
- Data properties: position, overall FIFA value, growth/potential, economic value, annual wage, contract value, player feature.

**Class: TransferMarketProfile** (separate class — the profile of the player, not the player itself). You could use a data property with the Transfer Market URL, but a separate class is cleaner if the profile is its own entity.

**Class: Club**
- Data properties: nationality/country.
- Object property: competes in a League.

**Class: League**
- Data properties: nationality.

**Relationships:**
- Player **plays for** a Club (one club at a time — a player at AC Milan does not borrow Real Madrid's properties).
- Club **competes in** a League.
- League has a nationality.

### 6.3 The Appearance Class (Many-to-Many Relationship)

Player, Club, and Game are all in many-to-many relationships, so an **intermediate class** is needed: **Appearance**.

> A player does not care about the game in general. He cares about the game while they were playing it. That's the appearance.

Example: Roberto Baggio scored 100 goals in 200 appearances, not in 200 games. A player may have 20 goals in 30 games, but summed minutes may be less than 90 × 30 because appearances were short (e.g. 20 minutes × 30).

**Appearance properties:** goals, assists, red cards, yellow cards, minutes played (implicitly via the difference between games and actual playing time).

**Game properties:** match day, home club goals, away club goals, home club, away club.

Example: Milan vs Juventus, Champions League final, May 2005, 0–0 (decided on penalties).

### 6.4 Position Taxonomy Discussion

The professor discusses modeling positions as a taxonomy:

- **Macro taxonomy:** goalkeeper, defender, midfield, attack.
- **Micro taxonomy:** central midfield, left midfield, right midfield.

**Key question:** do we need classes for positions?
- Players can be left midfield **and** left defender (multiple classes).
- Different macro-positions may have different data properties (attributes differ for midfielders vs defenders vs strikers).
- Same positions may have the same properties (left midfield and right midfield share properties, differing only by foot).
- **Object properties** are key to the subclass decision; most data properties are common to all players.

The professor keeps it simple and does not fully flesh out the taxonomy, noting that object properties drive the subclass decision.

## Exam Hooks

- `hasValue` forces exactly one value on a property; it is a specific case of the existential quantifier.
- Existential = at least one value must exist; universal = all values restricted to a set.
- Classism = modeling everything with classes, creating redundant individuals; avoid when not needed.
- Minimality = minimum concepts and minimum individuals.
- Subclass when you need specific individuals/properties; data property when values differentiate instances.
- Before subclassing: can you imagine an individual, properties, specific values?
- SKOS = data-sharing standard, RDF-based, not a formal KR language.
- `skos:broader` / `skos:narrower`: counterintuitive direction; not transitive by default; use `*Transitive` for inference.
- `skos:related`: horizontal and symmetric.
- Concept (definition-based) vs class (member-based).
- Design pattern: classes for concrete entities, SKOS concepts for shared repeatable concepts.
- Punning = individual used as a schema element; mixes TBox and ABox; possible but bad practice.
- Appearance = intermediate class for player–club–game many-to-many; captures goals/assists/cards/minutes per appearance.
- Object properties, not data properties, drive the position-subclass decision.

## Related

- [[Graph Databases]] - course hub
- [[RDF Data Model]] - RDF triples are the substrate OWL/SKOS axioms are built on
- [[Linked Open Data]] - OWL/SKOS vocabularies are published as LOD
- [[Database Foundations and Architecture]] - cross-course: database models this course extends

## Sources

- [[lezione_persa_GDB]] (AY 2026/2027)
