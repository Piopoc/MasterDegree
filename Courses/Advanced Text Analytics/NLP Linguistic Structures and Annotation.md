---
type: course-note
date: 2026-10-09
tags:
  - course
  - advanced-text-analytics
  - nlp
course: "[[Advanced Text Analytics]]"
sources:
  - "[[NLP]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course covering linguistic structures and computational annotation in NLP. Created 2026-10-09 from [[NLP]] (already in English; restructured for study use). Use it when working on tokenization, POS tagging, parsing, or any SpaCy-style annotation pipeline, or when explaining why bag-of-words is insufficient. Key points: word vs. token, the annotation pipeline (lemma, morphology, POS, dependency, entities), the phrase/clause hierarchy, and dependency vs. constituency grammar.

## 1. Words vs. Tokens

A fundamental distinction between the theoretical linguistic unit (**Word**) and the practical processing unit (**Token**):

| Aspect | Word | Token |
|---|---|---|
| Nature | Linguistic unit | Computational unit |
| Defined by | Grammar and lexicon | Tokenizer rules |
| Examples | "do", "not", "don't" | "Don", "'", "t" *(tokenizer dependent)* |
| Stability | Fixed in language | Variable across NLP systems |
| Relevance | Linguistic analysis | Text preprocessing / model input |

### Major syntactic categories of words

- **Noun (N)** - depicts an object or entity (living or non-living).
- **Verb (V)** - actions, states, or occurrences (incl. auxiliary and transitive sub-categories).
- **Adjective (Adj)** - qualifies other words, typically nouns/noun phrases.
- **Adverb (Adv)** - modifies nouns, verbs, adjectives, or other adverbs.

## 2. Why Structure Matters: The Bag-of-Words Failure

Reordering the same word set entirely alters meaning:

- _"The dog bites the man"_ vs. _"The man bites the dog"_ - identical word bags, opposing actions.

Word order dictates functional roles, so NLP needs **annotation layers over the surface text** that preserve it without altering the original string.

## 3. The Annotation Pipeline

```
[Raw Text] ──▶ [Tokenization] ──▶ [Linguistic Annotation]
                                         │
    ┌────────────────┬───────────────────┼───────────────────┬────────────────┐
    ▼                ▼                   ▼                   ▼                ▼
[Lemma]        [POS Tagging]       [Morphology]        [Dependency]      [Entities]
```

### Key layers

1. **Lemmatization** - reduce inflected forms to the canonical base:
   - _mutations_ -> `mutation`, _applied_ -> `apply`, _courts_ -> `court`
2. **Morphological features** - grammatical properties of a token: for _**applied**_: Lemma = `apply`, Tense = `Past`, VerbForm = `Fin`.
3. **POS tagging & dependency parsing** - syntactic roles and grammatical relations.

### Worked example

Input: _"BRCA1 mutations increase breast cancer risk."_

- **BRCA1:** `PROPN` (proper noun)
- **mutations:** `NOUN` -> *nsubj* (nominal subject) of `increase`
- **increase:** `VERB`
- **breast / cancer / risk:** `NOUN`; **risk** -> *obj* (direct object) of `increase`

## 4. Structural Hierarchy of Language

$$\text{Sentence} \rightarrow \text{Clause} \rightarrow \text{Phrase} \rightarrow \text{Word}$$

### Phrases

A **phrase** is a group of words functioning as a single constituent; it does **not** contain both a subject and a predicate, and every phrase has a **head word** determining its category:

| Phrase type | Head | Examples |
|---|---|---|
| **NP** (noun phrase) | noun | _dessert_, _the lazy dog_, _the big red ball_ |
| **VP** (verb phrase) | verb | _is running_, _could have gone_ |
| **AdjP** | adjective | _very intelligent_, _extremely happy_ |
| **AdvP** | adverb | _quite slowly_, _as often as possible_ |
| **PP** (prepositional phrase) | preposition | _in the garden_, _through the park_ |

### Clauses

A **clause** contains both a subject and a predicate:

1. **Independent (main) clause** - complete thought, stands alone: _"She smiled."_
2. **Dependent (subordinate) clause** - needs an independent clause: _"because it rained"_

### Phrase vs. clause

| Feature | Phrase | Clause |
|---|---|---|
| Subject + predicate? | No | Yes |
| Complete thought? | No | Often (if independent) |
| Sentence role | Constituent unit | Sentence or major section of one |
| Samples | _the old house_, _in the morning_ | _she left early_, _because it rained_ |

## 5. Computational Grammar Frameworks

### Dependency Grammar

Models syntax strictly through **word-to-word relationships**, not phrase groupings:

- **Core principle:** every word depends on another word, with exactly one exception.
- **Root:** the single word with no parent dependency (typically the main verb).
- **Dependency arcs:** directed, labeled relations - `nsubj`, `obj`, `amod`, `det`, `prep`.

### Constituency Grammar (Phrase Structure Grammar)

Models syntax as nested, hierarchically ordered **constituents**:

- Sentences are built from phrase components via formal **phrase structure rules**.
- Words -> lexical categories -> heads of larger phrases (NP, VP, PP) -> clauses -> sentences.
- Output: a **nested constituency tree**, not point-to-point word dependencies.

### When to use which

- Dependency: direct relations between words - compact, word-order aware, standard in modern pipelines (SpaCy uses dependency parses).
- Constituency: hierarchical grouping - better for grammatical-theoretic analysis of phrase structure.

## Related

- [[LOTUS Semantic Operators]] - how such annotations feed LLM-based data processing (cross-course)
- [[Advanced Text Analytics]] - course hub

## Sources

- [[NLP]] (AY 2026/2027)
