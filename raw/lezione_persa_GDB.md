# Graph Databases — Lecture Notes

**Course:** Graph Databases  
**Institution:** University of Padova — Master's Degree in Computer Engineering  
**Professor:** Italian professor teaching in English  
**Source:** Lecture transcription (`audio.txt`)  
**Language:** English  

---

## Table of Contents

1. [Overview](#overview)
2. [OWL Ontology Foundations](#owl-ontology-foundations)
   - [Conformant Apps and the `hasValue` Constraint](#conformant-apps-and-the-hasvalue-constraint)
   - [Existential vs Universal Quantifiers](#existential-vs-universal-quantifiers)
   - [ADA 90 Compliance Example](#ada-90-compliance-example)
3. [Class vs Instance Modeling](#class-vs-instance-modeling)
   - [The "Classism" Problem](#the-classism-problem)
   - [Minimality Principle](#minimality-principle)
   - [Medical/Histology Domain Example](#medical-histology-domain-example)
   - [Data Properties vs Subclasses](#data-properties-vs-subclasses)
   - [When to Subclass vs When to Add Properties](#when-to-subclass-vs-when-to-add-properties)
4. [SKOS — Simple Knowledge Organization System](#skos---simple-knowledge-organization-system)
   - [What Is SKOS](#what-is-skos)
   - [Concept Scheme and Concepts](#concept-scheme-and-concepts)
   - [skos:broader / skos:narrower](#skosbroader--skosnarrower)
   - [skos:related](#skosrelated)
   - [skos:prefLabel / skos:altLabel / skos:inScheme](#skospreflabel--skosaltlabel--skosinscheme)
   - [Concept vs Class vs Resource vs Thing](#concept-vs-class-vs-resource-vs-thing)
   - [SKOS Is Not a Formal Knowledge Representation Language](#skos-is-not-a-formal-knowledge-representation-language)
   - [SKOS Examples](#skos-examples)
5. [SKOS Applied to Medical Modeling](#skos-applied-to-medical-modeling)
   - [Avoiding Classism with Concepts](#avoiding-classism-with-concepts)
   - [Anatomical Locations as SKOS Concepts](#anatomical-locations-as-skos-concepts)
   - [Disease Concepts Shared Across Patients](#disease-concepts-shared-across-patients)
6. [Punning](#punning)
   - [What Is Punning](#what-is-punning)
   - [TBox vs ABox Mixing](#tbox-vs-abox-mixing)
   - [Why It Is Considered Bad Practice](#why-it-is-considered-bad-practice)
7. [Soccer/Football Ontology Case Study](#soccerfootball-ontology-case-study)
   - [Requirements and Data Sources](#requirements-and-data-sources)
   - [Competency Questions](#competency-questions)
   - [Classes and Properties](#classes-and-properties)
   - [The Appearance Class (Many-to-Many Relationship)](#the-appearance-class-many-to-many-relationship)
   - [Position Taxonomy Discussion](#position-taxonomy-discussion)
8. [General Ontology Modeling Guidelines](#general-ontology-modeling-guidelines)
   - [Questions to Ask Before Creating a Subclass](#questions-to-ask-before-creating-a-subclass)
   - [ER Modeling vs Ontology Modeling](#er-modeling-vs-ontology-modeling)
9. [Exam and Grading Advice](#exam-and-grading-advice)
10. [Summary and Key Takeaways](#summary-and-key-takeaways)

---

## 1. Overview

This lecture covers the modeling aspects of graph databases and semantic web technologies. The professor moves through several connected topics:

- **OWL ontology constraints** (`hasValue`, existential/universal quantifiers)
- **Class vs instance modeling** — when to model something as a class vs as an individual/concept, and the pitfalls of "classism"
- **SKOS** (Simple Knowledge Organization System) — a lightweight vocabulary for organizing knowledge that sits on top of RDF
- **Punning** — using an individual as a schema element (mixing TBox and ABox)
- **A complete soccer/football ontology** — a practical case study built from FIFA video game data and Transfer Market data

The teaching style is conversational, with many examples drawn from medical/histology domains and video games. The professor emphasizes that there is rarely one "correct" model — rather, there are trade-offs depending on the application requirements.

---

## 2. OWL Ontology Foundations

### 2.1 Conformant Apps and the `hasValue` Constraint

The lecture opens with an example about software compliance:

- A **desktop app** is a subclass of a **conformant app**.
- A **conformant app** is an app conforming to a legislation.
- The specific legislation to conform to is **ADA 90** (Americans with Disabilities Act, 1990).
- If Excel is a desktop app, then by inference it must be a conformant app, and therefore it must conform to ADA 90.

To express this in OWL, the professor uses the **`hasValue` constraint**:

> You create an equivalent class to the conformant class and you restrict the `conformsTo` property with `hasValue S-value` where S-value is exactly one value (in this case, ADA-90).

This means: when you are conformant, you must have exactly one value on the `conformsTo` property, and that value must be the specific legislation (e.g., ADA-90). You are forcing this constraint.

**Key point:** `hasValue` is a specific case of the **existential quantifier**.

### 2.2 Existential vs Universal Quantifiers

| Quantifier | Meaning |
|------------|---------|
| **Existential** (`someValuesFrom` / `hasValue`) | There **must exist** some value on the class legislation for the property `conformsTo` for an element in the conforming class. At least one value must exist. |
| **Universal** (`allValuesFrom`) | You may have `conformsTo` only going to specific values — it restricts all values to a set. |

With `hasValue`, there might be other values conformed to it, but there **must be one** — and in this case, specifically ADA-90.

**Note on multiple values:** If new legislation arrives (e.g., ADA-2019), you could in principle put two values because `hasValue` accepts a set. However, the professor advises that usually you want only one value — the more values you add, the more "existential" it becomes. A good use case is to have exactly one specific value.

### 2.3 ADA 90 Compliance Example

```text
Individual: ADA-90
Type: Legislation

Class: ConformantApp
EquivalentTo: App and (conformsTo hasValue ADA-90)

Class: DesktopApp
SubClassOf: ConformantApp

Individual: Excel
Type: DesktopApp
```

From this, a reasoner infers:
- Excel is a ConformantApp
- Excel conformsTo ADA-90

---

## 3. Class vs Instance Modeling

### 3.1 The "Classism" Problem

**Classism** is the practice of modeling *everything* with classes. The professor warns against this:

> Classism is when you model everything with a class. You see, they are all classes.

When you model everything with a class, and then you need to instantiate it, everything you represent is instantiated by creating new URIs. This leads to:

- Creating too many individuals for the same concept
- Wasted resources
- Unnecessary complexity
- Redundant individuals referring to the exact same thing

### 3.2 Minimality Principle

The professor advocates for modeling with the **minimum number of concepts** while also generating the **minimal number of individuals** as a good practice:

> There is also this concept that you have to model with the minimum number of concepts but also generating the minimal number of individuals as a good practice.

A model is not "wrong by definition" just because it uses many classes — but correctness and minimality matter. The right choice depends on the specific requirements.

### 3.3 Medical/Histology Domain Example

The professor uses a medical histology analysis scenario as a running example throughout the lecture.

**The reality to model:**
- A tissue is withdrawn from a patient (invasive procedure).
- The tissue is located in an anatomical location (lungs, pancreas, etc.).
- The tissue is analyzed to see if there is cancer or not.
- The tissue has an outcome: a disease (cancer, diabetes, polyp, etc.).

**First modeling attempt (class-based / classism):**

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

This model works — you can instantiate:
- Tissue X withdrawn from person
- Tissue X located in anatomical location XYZ-34 (lower lung)
- Tissue X outcome = disease ASPX-49 (polyp)

**But it creates problems:**

| Individual | What it represents | Problem |
|------------|-------------------|---------|
| Tissue X | Concrete tissue from patient 1 | Needed — real object in a jar |
| Tissue Y | Concrete tissue from patient 2 | Needed — real object |
| Person | The patient | Needed |
| XYZ-34 | Lower lung (patient 1's) | Redundant — same concept as patient 2's lower lung |
| XYZ-45 | Lower lung (patient 2's) | Redundant duplicate |
| ASPX-49 | Polyp (patient 1's tissue) | Redundant — polyp is a general concept |
| ABX-37 | Polyp/carcinoma (patient 2's tissue) | Redundant duplicate |

You create **4 individuals** (2 locations + 2 disease outcomes) that refer to exactly the same concepts. The lower lung is the same concept for everyone. The polyp is the same concept for everyone. Unless you actually store the specific cells from a specific polyp, you do not need a separate individual per patient.

### 3.4 Data Properties vs Subclasses

The professor discusses whether to model severity levels (e.g., bad polyp, good polyp, very bad polyp) as **subclasses** or as a **data property**.

**Option A — Subclasses:**
```
Class: BadPolyp SubClassOf: Polyp
Class: GoodPolyp SubClassOf: Polyp
Class: VeryBadPolyp SubClassOf: Polyp
```

**Option B — Data property:**
```
Class: Polyp
DataProperty: severity (values: good, bad, very bad)
```

**Which is correct?** It depends on the application:

- **If you are merging graphs on the web** discussing polyps: you probably need the concept of a "good polyp" as a class, because you need to merge all good polyps out there.
- **If you are managing a hospital:** you probably do not need classes for specific individuals' good/bad polyps. You ask: what is the role of these individuals? What properties differentiate good from bad? The answer is just belonging to that class. So you put all polyps in a higher class and differentiate with a property.

> Voilà. Right? So this is the kind of reasoning. As you see, there is not a rule that always applies. But there is guidance that you can check to think what is the best choice.

### 3.5 When to Subclass vs When to Add Properties

Before creating a subclass, ask:

1. **Can you imagine an individual for this class?** If not, the class itself should probably be an individual.
2. **Can you imagine properties for the class?**
3. **Can you imagine specific values for those properties?**

A subclass should have specific properties or useful individuals. For example:
- "Elisabetta Montos" (a literary author from a given period) may have specific properties that general authors do not.
- Literary authors vs screenwriters are two different sets that may overlap.

**The reasoning pattern:** always think about *why* you are creating a subclass, whether it makes sense, whether you will use it, whether there are individuals, and whether there are specific properties — *then* create the subclass.

---

## 4. SKOS — Simple Knowledge Organization System

### 4.1 What Is SKOS

**SKOS** = **Simple Knowledge Organization System**.

- It is a **data sharing standard** (originated around 2009).
- It breaches several different fields of knowledge, technology, and practice.
- Goal: provide a bridge between different communities or practices (e.g., library science, information science).
- The SKOS data model is formally defined as a **full ontology** — it is at the highest level of expressiveness within its scope.
- **However, SKOS is not a formal knowledge representation language.** It is a vocabulary with terms; each term has a specific meaning, and you use those terms to build constructs.
- The underlying data model is **RDF** — always the same triple structure.
- Compared to RDFS → OWL, SKOS adds more terms and more axioms, increasing complexity.

### 4.2 Concept Scheme and Concepts

SKOS views a knowledge organization as a **concept scheme** comprising a set of **concepts**.

| Term | Meaning |
|------|---------|
| **Concept Scheme** | An organization of knowledge (e.g., Linnaean taxonomy of life, video game genres, movie genres). |
| **Concept** | A unit within a scheme (e.g., "mammal", "roguelike", "action game"). |

Examples of concept schemes:
- **Linnaeus taxonomy** — organizing life (animal, plant, mammal, reptile, cow, etc.)
- **Video game anthology** — genres (platform, roguelike, Castlevania), platforms (PlayStation 5, PlayStation 4, Xbox, Switch)
- **Movie genres** — connected to video game genres (some movie genres derived from video games)

Concepts can be:
- Labeled with lexical strings (one or more labels)
- Assigned one or more notations
- Linked to other concepts
- Mapped across different schemes (e.g., the same concept appears in both video games and movies schemes)

SKOS does not model only a hierarchy — it is a **graph** (typically called a **thesaurus**) because you may have relationships between siblings, not only up-and-down hierarchies.

### 4.3 skos:broader / skos:narrower

These are the two main semantic (vertical) properties.

- `skos:broader` — goes from a concept to its broader (parent) concept.
- `skos:narrower` — goes from a concept to its narrower (child) concept.

**Direction is counterintuitive:**

> We say: sheep `skos:broader` livestock. So it means that sheep has broader concept livestock. Livestock `skos:narrower` sheep.

So:
- `A skos:broader B` → A's broader concept is B → **B is broader than A** (B is the parent).
- `B skos:narrower A` → B's narrower concept is A → **A is narrower than B** (A is the child).

The professor notes this is tricky and counterintuitive — you must pick which direction you want and remember how to reason about it.

**Transitivity:**
- `skos:broader` and `skos:narrower` are **NOT transitive** by default.
- There are transitive versions: `skos:broaderTransitive` and `skos:narrowerTransitive`.
- You use the transitive version **only when you want to perform inference on that chain**.
- In meaning, narrower/broader are naturally transitive (a mammal is narrower than an animal, a cow is narrower than a mammal, therefore a cow is narrower than an animal). But SKOS separates the simple property from the transitive one so you don't force inference everywhere.

### 4.4 skos:related

- A **horizontal** relationship between concepts.
- **Symmetric**: if two concepts are related, they are related in both ways.
- Example: "roguelike" related to "procedural generation" (not parent-child, just associated).

### 4.5 skos:prefLabel / skos:altLabel / skos:inScheme

| Property | Meaning | Example |
|----------|---------|---------|
| `skos:prefLabel` | Preferred label (plain literal, often with language tag) | "love" @en |
| `skos:altLabel` | Alternative label | "adoration" @en |
| `skos:inScheme` | The concept scheme this concept belongs to | concept A in scheme S |

Example from the lecture:
- Individual `A` is of RDF type `skos:Concept`.
- `A` has `skos:prefLabel` "love" (plain literal, English).
- `A` has `skos:altLabel` "adoration" (plain literal, English).
- `A skos:inScheme S`.
- `A skos:broader D`.
- `B skos:prefLabel` "emotion"; `B skos:broader` relationship makes "emotion" broader than "love".

### 4.6 Concept vs Class vs Resource vs Thing

SKOS was built around the idea of **concept** because earlier RDF/RDFS vocabularies lacked a dedicated term for it:

| Term | Characteristic |
|------|---------------|
| **Class** | Has individuals/members belonging to it. Defined by extension (its members). |
| **Concept** | Has a **definition**. A concept is not defined by its members but by its meaning/definition. |
| **Resource** | Everything in RDF is a resource (`rdfs:Resource` is the top). |
| **Thing / `owl:Thing`** | Everything is a subclass or sub-element of `owl:Thing`. |

A concept is different from a class, a resource, and a thing. SKOS fills this gap by providing `skos:Concept`.

**Important distinction:** A class is set-based (defined by its members). A concept has a definition independent of its instances. For example:
- The concept "roguelike" is a genre definition.
- The instance "Dead Cells" is an instance of that concept.
- But saying "Dead Cells is a class" would be wrong — Dead Cells is an individual/game, not a class of games.

### 4.7 SKOS Is Not a Formal Knowledge Representation Language

The professor quotes the specification:

> SKOS is so powerful and big and used by people all the time without them knowing that they are using RDF that it is an independent thing and it is actually because SKOS is a representation of knowledge. But you can use it without knowing RDF.

You can use SKOS terminology (term, subordinate, superordinate) without knowing RDF. When used side by side with OWL, it is not another language — it is the same language (RDF), just a different vocabulary.

### 4.8 SKOS Examples

**Example 1 — Taxonomy of Life (Linnaeus):**
```
animal skos:broader livingThing
plant skos:broader livingThing
mammal skos:broader animal
reptile skos:broader animal
cow skos:broader mammal
```

**Example 2 — Video Games:**
```
roguelike skos:broader videoGameGenre
platform skos:broader videoGameGenre
castlevania skos:broader videoGameGenre  (mixture of castle + exploration games from the 90s)
deadCells skos:inScheme videoGameGenres
deadCells skos:broader roguelike  (or: deadCells is an instance of roguelike)
playstation5 skos:broader playstation
playstation5pro skos:broader playstation5
xbox skos:broader console
switch skos:broader console
```

---

## 5. SKOS Applied to Medical Modeling

### 5.1 Avoiding Classism with Concepts

The professor shows how to use SKOS to model the same medical situation without classism:

> Tissue withdrawn patient, tissue located in an anatomical location, tissue has disease. Because that part we like, this means that we instantiate always a tissue and always a patient.

But instead of creating separate class individuals for every patient's lung and polyp, you use **SKOS concepts** for anatomical locations and diseases.

### 5.2 Anatomical Locations as SKOS Concepts

You have a **head class** (the most general class for the organizational system). On one hand: anatomical location; on the other hand: disease.

Anatomical location individuals (lung, upper lung, lower lung) are also declared as `skos:Concept`:

- `lung` is both an `AnatomicalLocation` and a `skos:Concept`.
- `upper lung` is both an `AnatomicalLocation` and a `skos:Concept`.
- `upper lung skos:broader lung`.
- `lower lung skos:broader lung`.

This gives you all the properties of a concept:
- **Labels**: `skos:prefLabel` "lung" @en, "polmone" @it, etc.
- **Broader/narrower relationships** for organizing knowledge.

You can now point to the **same concept** (`lung`) for every patient, rather than creating a new individual `XYZ-34` per patient.

### 5.3 Disease Concepts Shared Across Patients

Similarly, disease concepts (lung cancer, polyp, carcinoma) are SKOS concepts:

- `lung cancer` is a `skos:Concept` and a `Disease`.
- `polyp` is a `skos:Concept` and a `Disease`.
- `lung carcinoma skos:broader lungCancer` (or similar hierarchy).

When modeling:
```
John a Patient.
TissueX a Tissue.
TissueX withdrawnFrom John.
TissueX locatedIn lung.
TissueX hasDisease polyp.
```

You do **not** need to instantiate a specific "polyp of patient John" — `polyp` is a shared concept. You only need a specific individual if you later store the actual cells from that specific polyp.

**This is a design pattern:** use classes for real, concrete entities (patients, tissues in jars), and use SKOS concepts for shared, repeatable concepts (anatomical locations, diseases).

---

## 6. Punning

### 6.1 What Is Punning

**Punning** is when you use an **individual** as a **schema element** — mixing instances (ABox) with classes/properties (TBox) implicitly.

The professor encountered this in a big project in 2017. He used it, then disliked it, then developed a different solution (the SKOS-based approach presented in the lecture).

### 6.2 TBox vs ABox Mixing

- **TBox** (Terminology Box): classes, properties, axioms — the schema.
- **ABox** (Assertion Box): individuals, assertions — the data.

You cannot normally cross the line between TBox and ABox. If not by using `rdf:type`, you cannot use `type` in certain contexts. So you do punning: an individual that is actually used as a schema element because you always refer to it as if it were a class.

> It's a bad practice, I really don't like it, but in general it is not bad. But it's possible.

### 6.3 Why It Is Considered Bad Practice

- You are mixing instances and schema implicitly — it is not evident.
- You have an individual used as a schema element.
- You cannot refer directly to the class because you cannot cross the TBox/ABox boundary (except via `rdf:type`).

The professor notes: this is the only way he had seen written to do it, but he couldn't find it in any paper. He is writing a chapter so it gets documented. He invites students to tell him if they find a source with this pattern.

---

## 7. Soccer/Football Ontology Case Study

### 7.1 Requirements and Data Sources

The professor introduces a complete football/soccer ontology used for RDF ingestion and graph database population.

**Data sources (open data, not linked data yet):**
- **FIFA video game database** (CSV file with player characteristics, overall value, potential, etc.) — data is about 4–5 years old, so players are not current.
- **Transfer Market** website (player economic value, statistics).

**Goal:** Transform open data into **linked data** by merging the two sources.

**What is modeled:**
- Football players with performance attributes, abilities, economic value, wage, etc.
- Club and league of the club.
- Games, teams, time, results, outcomes.
- Performances of single players: goals, assists.
- **Assists**: key passages — when you make a pass and the other player scores after that pass.
- Yellow and red cards.

**Competency questions** (example queries the ontology should answer):
- Who are the top players in terms of economic value and overall performance indicator?
- Who received more red cards in the [current] season?

> When you model an ontology, you need a big set of competency questions. Here is a very simple example.

### 7.2 Classes and Properties

**Class: Person** (superclass)
- Data properties: first name, last name, birthday, height, weight, nationality (also characteristic of a manager).

**Class: Player** (subclass of Person)
- Data properties: position, overall FIFA value, growth/potential, economic value, annual wage, contract value, player feature.

**Class: TransferMarketProfile** (separate class — it is the profile of the player, not the player itself)
- You could alternatively use a data property with the Transfer Market URL, but a separate class is cleaner if you need the profile as its own entity.

**Class: Club**
- Data properties: nationality/country.
- Object property: competes in a League.

**Class: League**
- Data properties: nationality.

**Relationships:**
- Player **plays for** a Club (one club at a time — a player may play for AC Milan, but properties of Real Madrid are not borrowed).
- Club **competes in** a League.
- League has a nationality.

### 7.3 The Appearance Class (Many-to-Many Relationship)

Player, Club, and Game are all in many-to-many relationships, so an **intermediate class** is needed: **Appearance**.

**Why Appearance?**
> A player does not care about the game in general. He cares about the game while they were playing the game. That's the appearance.

Example: Roberto Baggio scored 100 goals in 200 appearances, not in 200 games. A player may have 20 goals in 30 games, but if you sum the minutes played, it may be less than 90 × 30 because appearances were short (e.g., 20 minutes × 30).

**Appearance properties:**
- Goals
- Assists
- Red cards
- Yellow cards
- Minutes played (implicitly, via the difference between games and actual playing time)

**Game properties:**
- Match day
- Home club goals
- Away club goals
- Home club
- Away club

Example: Milan vs Juventus, date May 2005 (Champions League final), home goals 0, away goals 0 (decided on penalties).

### 7.4 Position Taxonomy Discussion

The professor discusses whether to model positions as a taxonomy:

**Macro taxonomy:** goalkeeper, defender, midfield, attack  
**Micro taxonomy:** central midfield, left midfield, right midfield

**Key question:** Do we need classes for positions?

- Players can be left midfield **and** left defender (multiple classes).
- Different macro-positions may have different data properties (e.g., attributes differ for midfielders vs defenders vs strikers).
- Same positions may have the same properties (left midfield and right midfield have same properties, just different foot).
- **Object properties** are key to deciding whether to create subclasses. Different object properties are really the key; most data properties are common to all players.

The professor keeps it simple and does not fully flesh out the position taxonomy, noting that object properties drive the subclass decision.

---

## 8. General Ontology Modeling Guidelines

### 8.1 Questions to Ask Before Creating a Subclass

The professor recommends this checklist when considering a new subclass:

1. **Can I imagine an individual for this class?** (Yes/No — if no, the class itself should be an individual.)
2. **Can I imagine properties for the class?**
3. **Can I imagine specific values for those properties?**

A subclass should justify its existence through:
- Useful individuals that can be instantiated, **or**
- Specific properties that distinguish it from the superclass.

### 8.2 ER Modeling vs Ontology Modeling

The professor worked with database professors who were experts in Entity-Relationship (ER) modeling. They initially thought ontology was "just a different formality" for ER modeling.

**It is not — especially at scale.**

| Aspect | ER Modeling | Ontology Modeling |
|--------|-------------|-------------------|
| Generalization/specialization | Restructured in logical model (tables merged or split) | No restructuring — no logical model |
| Conceptual → physical | 1:1 mapping does not hold; you restructure | Conceptual level is 1:1 with physical level |
| Scale | Works well for small/medium models | Key differences emerge at very large scale |

> What you are modeling at the conceptual level is 1 to 1 what you find at the physical level. Actually not really at the instances — so it's a big difference.

---

## 9. Exam and Grading Advice

The professor shares advice about the course exam and grading (Italian system: 18–30, with 30 being the maximum):

- **Goal of the exam:** Check if you understood what was said, not to take you to a difficult place.
- **Professor's approach:** He gives you the types of questions during the course so that if you practiced everything he said, you know exactly what to expect — though not the specific content.
- **Grades:**
  - **30/30:** Excellent (often research-oriented students).
  - **26:** Perfect grade.
  - **24:** Acceptable.
  - **Below 24:** Fine — don't worry too much.
- **Research vs grades:** The best students are often those in research, not necessarily those with all 30s. A student with 30s everywhere may be "plain" — good at everything but not differentiated. The professor prefers students who are very good at what they like (e.g., graph databases, generative AI, search engines) even if their GPA is 26–28.
- **Advice:** Know what you are good at and what you really like to do. Specialize. In this society, we need specialization.

---

## 10. Summary and Key Takeaways

| Topic | Key Takeaway |
|-------|-------------|
| `hasValue` constraint | Forces exactly one value on a property; specific case of existential quantifier. |
| Existential vs universal | Existential = at least one value must exist; Universal = all values restricted to a set. |
| Classism | Modeling everything with classes creates redundant individuals; avoid when not needed. |
| Minimality | Model with minimum concepts and minimum individuals. |
| Subclass vs property | Subclass when you need specific individuals/properties; data property when values differentiate instances. |
| SKOS | Lightweight knowledge organization on top of RDF; concept scheme + concepts; not a formal KR language. |
| skos:broader/narrower | Direction is counterintuitive (`A broader B` means B is broader than A); not transitive by default; use `*Transitive` for inference. |
| skos:related | Horizontal, symmetric relationship between concepts. |
| Concept vs class | A concept has a definition; a class is defined by its members. SKOS fills the gap RDF/RDFS lacked. |
| Punning | Using an individual as a schema element; mixes TBox and ABox; possible but bad practice. |
| Appearance class | Intermediate class for many-to-many (player–club–game); captures minutes/goals/assists/cards per appearance. |
| Modeling advice | Ask: can you imagine an individual? properties? specific values? Then decide on subclass. |

---

*These notes were generated from the lecture transcription `audio.txt`. If any section needs clarification or expansion, please let me know.*
