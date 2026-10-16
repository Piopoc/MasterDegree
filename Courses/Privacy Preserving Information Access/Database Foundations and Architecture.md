---
type: course-note
date: 2026-10-09
tags:
  - course
  - ppia
  - databases
course: "[[Privacy Preserving Information Access]]"
sources:
  - "[[Intro]]"
ai-first: true
---

## For Future Agent

Study note for the [[Privacy Preserving Information Access]] course covering database fundamentals, the ANSI/SPARC architecture, the database design process, and the relational model. Created 2026-10-09 from the database-fundamentals portion of [[Intro]] (translated from Italian). Key points: DB/DBMS/UoD definitions, three-level architecture, DBMS-independent vs. dependent design phases, Codd's relational model, and real-world platform case studies.

## 1. Database and DBMS

- **Database (DB):** a permanent, coherent, and integrated collection of related data - facts known, recordable, and endowed with intrinsic meaning - designed for a specific purpose and for well-identified users. It models a well-defined portion of the real world, technically called the **mini-mondo** or **universe of discourse (UoD)**. The collection is kept for long periods, often decades.
- **Database Management System (DBMS):** general-purpose software that lets users create, manage, and update a database. It presides over three fundamental phases:
  1. **Schema definition** - describing data types, structures, and constraints.
  2. **Construction** - storing the data.
  3. **Manipulation** - querying and updating.
  The formal schema description is archived as **metadata** inside the **system catalog**.

### Sharing and concurrency control

Sharing the database among multiple applications and users enables simultaneous access while reducing redundancy and inconsistency. Concurrency control guarantees two key properties:

- **Isolation** - prevents unwanted interference between concurrent accesses.
- **Atomicity** - prevents partial or incomplete transactions from modifying data state.

## 2. ANSI/SPARC Three-Level Architecture

| Level | Function |
|---|---|
| **External** | Defines user schemas, presenting only the data relevant to each application. |
| **Logical** | Provides a global, integrated representation of the whole database, independent of physical storage. |
| **Internal** | Governs the concrete physical representation via underlying data structures and hardware storage units. |

## 3. Database Design Process

Three sequential phases:

1. **Conceptual design** - translates mini-mondo requirements into an abstract schema via the **Entity-Relationship (ER) model**. Rigorously **DBMS Independent**.
2. **Logical design** - produces a relational schema. **DBMS Dependent** (bound to the chosen data model).
3. **Physical design + tuning** - storage structures and access methods. **DBMS Dependent** (bound to hardware).

The DBMS Independent / DBMS Dependent demarcation is methodologically central to the software development lifecycle: only the conceptual level survives a change of platform (open question).

## 4. The Relational Model

- Introduced in **1970 by E. F. Codd at IBM**, with the primary goal of promoting **data independence**.
- Founded on the mathematical concept of a **relation**, represented intuitively via **tables**.
- **SQL** (*Structured Query Language*) is the standard language in DBMSs for defining, manipulating, and querying tables.

## 5. Case Studies: Why Large Platforms Mix Engines

Commercial platforms combine traditional relational DBMSs with distributed engines, Big Data architectures, or in-memory caching layers for architectural and performance reasons (figures as reported in the source):

| Platform | Stack | Figures reported in the lecture material |
|---|---|---|
| **Last.fm** | PostgreSQL (catalog) + Hadoop | 60+ billion *scrobbles* |
| **Facebook** | MySQL + Memcached | 800M users; 60M queries/s; 1.5B reads/s; 4-5 ms latency |
| **Twitter/X** | MySQL | 140M users; 5,000 tweets/s; 3M writes/s |
| **Skype** | PostgreSQL | call and chat records |
| **Public Interest Registry** | PostgreSQL | 10M+ `.org` domains |
| **McAfee / Trend Micro** | PostgreSQL | quarantine, anti-phishing and URL-filtering logs |

Confidence: `stated` for the stack choices; the performance figures are lecture-reported historical numbers - verify before reuse.

## 6. Glossary (this note's terms)

| Term | Definition |
|---|---|
| **Database (DB)** | Permanent, coherent, integrated collection of related data representing a mini-mondo for identified users over long periods. |
| **DBMS** | General-purpose software to create, manage, update a database: schema definition, construction, manipulation. |
| **Mini-Mondo / UoD** | The specific portion of reality modeled and stored in the database. |
| **System Catalog (Metadata)** | DBMS component holding the formal schema description (structures, types, constraints). |
| **External / Logical / Internal schema** | The three ANSI/SPARC levels: user view / global integrated view / physical representation. |
| **ER Model** | DBMS-independent conceptual graphical model of the mini-mondo. |
| **Relational Model** | 1970 Codd (IBM) logical model based on mathematical relations, represented as tables. |
| **SQL** | Standard relational language for defining, manipulating, and querying tables. |
| **Atomicity and Isolation** | Concurrency-control properties: no partial transactions; no interference between parallel transactions. |

## Related

- [[SQL Injection and Defenses]] - how SQL inputs can be abused
- [[Data Security and Privacy Fundamentals]] - privacy obligations on the stored data
- [[RDF Data Model]] - cross-course: the graph data model family databases belong to

## Sources

- [[Intro]] (AY 2026/2027)
