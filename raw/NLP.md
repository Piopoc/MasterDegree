# Linguistic Structures and Computational Annotations in NLP

## Executive Summary

Natural Language Processing (NLP) relies on transforming raw text into structured linguistic annotations without destroying the original surface text. While basic text processing treats language as an unordered collection of items, true linguistic understanding requires modeling word order, morphological properties, and hierarchical syntactic structures.

This briefing document details the core concepts of computational linguistics as applied in annotation pipelines (such as SpaCy). Key takeaways include:

- **Tokens vs. Words:** "Words" are fixed linguistic units defined by grammar and lexicon, whereas "Tokens" are computational units derived through tokenizer rules that vary across NLP systems.
- **The Power of Structure:** Text cannot be treated purely as a "bag of words." Reordering the same set of words entirely alters meaning (e.g., _"The dog bites the man"_ vs. _"The man bites the dog"_).
- **Annotation Pipelines:** NLP pipelines attach sequential layers of information to tokens—including tokenization, lemmatization, Part-of-Speech (POS) tagging, morphological analysis, dependency relations, and named entities.
- **Hierarchical Organization:** Language follows a strict structural hierarchy: **Sentence** **\rightarrow** **Clause** **\rightarrow** **Phrase** **\rightarrow** **Word**.
- **Grammar Frameworks:** Computational syntax is broadly categorized into **Dependency Grammar** (which maps direct head-dependent relationships between words branching from a single root) and **Constituency Grammar** (which organizes sentences into nested phrase structures based on phrase rules).

## 1. Fundamental Units: Words vs. Tokens

In computational linguistics, a fundamental distinction is made between a theoretical linguistic unit (**Word**) and a practical processing unit (**Token**).

|   |   |   |
|---|---|---|
|Aspect|Word|Token|
|**Nature**|Linguistic unit|Computational unit|
|**Defined By**|Grammar and lexicon|Tokenizer rules|
|**Examples**|“do”, “not”, “don’t”|“Don”, “’”, “t” _(tokenizer dependent)_|
|**Stability**|Fixed in language|Variable across NLP systems|
|**Relevance**|Used in linguistic analysis|Used in text preprocessing and model input|

While words are the smallest independent units in a language that carry inherent meaning, tokenizers split text into operational chunks for machine processing.

### Major Syntactic Categories of Words

Words are categorized into major syntactic classes that define their grammatical role:

- **Noun (N):** Depicts an object or entity (living or non-living).
- **Verb (V):** Describes actions, states, or occurrences. Sub-categories include auxiliary and transitive verbs.
- **Adjective (Adj):** Describes or qualifies other words, typically nouns and noun phrases.
- **Adverb (Adv):** Acts as a modifier for nouns, verbs, adjectives, or other adverbs.

## 2. Linguistic Annotation Pipelines and Syntax

Order and structure are essential to natural language processing. A "bag of words" model fails to capture semantic meaning because word order dictates functional roles. For example, _"The dog bites the man"_ and _"The man bites the dog"_ share identical word bags but convey opposing actions.

To preserve context, an NLP pipeline applies normalized linguistic layers on top of raw text without altering the surface string.

```
[Raw Text] ──▶ [Tokenization] ──▶ [Linguistic Annotation]
                                         │
    ┌────────────────┬───────────────────┼───────────────────┬────────────────┐
    ▼                ▼                   ▼                   ▼                ▼
[Lemma]        [POS Tagging]       [Morphology]        [Dependency]      [Entities]
```

### Key Annotation Layers

1. **Lemmatization:** Reduces inflected forms of a word to its base canonical form (lemma).
    - _mutations_ \rightarrow `mutation`
    - _applied_ \rightarrow `apply`
    - _courts_ \rightarrow `court`
2. **Morphological Features:** Identifies specific grammatical properties of a tokenized term.
    - For the term _**applied**_: Lemma = `apply`, Tense = `Past`, VerbForm = `Fin`.
3. **Part-of-Speech (POS) & Dependency Parsing:** Identifies syntactic roles and grammatical relationships between terms.

#### Annotation Example

For the input sentence: _"BRCA1 mutations increase breast cancer risk."_

- **BRCA1:** `PROPN` (Proper Noun)
- **mutations:** `NOUN` \rightarrow _nsubj_ (nominal subject) of `increase`
- **increase:** `VERB`
- **breast:** `NOUN`
- **cancer:** `NOUN`
- **risk:** `NOUN` \rightarrow _obj_ (direct object) of `increase`

## 3. Structural Hierarchy of Language

Language constituents exist in a nested hierarchy: **Sentence** **\rightarrow** **Clause** **\rightarrow** **Phrase** **\rightarrow** **Word**.

```
Sentence
  └── Clause
        └── Phrase
              └── Word
```

### Phrases

A **phrase** is a group of words that functions as a single constituent or part of speech within a sentence. A phrase **does not** contain both a subject and a predicate. Every phrase contains a **head word** that determines its syntactic category.

- **Noun Phrase (NP):** Head word is a noun. Functions as a subject or object.
    - _Examples:_ _"dessert"_, _"the lazy dog"_, _"the brown fox"_, _"the big red ball"_.
- **Verb Phrase (VP):** Head word is a verb. Expresses an action or state.
    - _Examples:_ _"is running"_, _"could have gone"_, _"is running fast"_.
- **Adjective Phrase (AdjP / ADJP):** Head word is an adjective. Modifies nouns or pronouns; positioned before or after the target word.
    - _Examples:_ _"very intelligent"_, _"Too quick"_, _"extremely happy"_.
- **Adverb Phrase (AdvP / ADVP):** Head word is an adverb. Modifies verbs, adjectives, or other adverbs with further detail.
    - _Examples:_ _"quite slowly"_, _"pretty soon"_, _"as often as possible"_.
- **Prepositional Phrase (PP):** Head word is a preposition. Provides spatial or temporal context.
    - _Examples:_ _"in the garden"_, _"on the table"_, _"through the park"_.

### Clauses

A **clause** is a group of related words that **does** contain both a subject and a predicate (or a predicate containing a verb/verb phrase with an object). Clauses can stand alone or be joined to form complex sentences.

1. **Independent (Main) Clause:** Expresses a complete thought and can stand alone as a sentence.
    - _Examples:_ _"She smiled."_, _"she left early"_
2. **Dependent (Subordinate) Clause:** Cannot stand alone as a sentence; relies on an independent clause to complete its meaning.
    - _Examples:_ _"because she was happy"_, _"because it rained"_

### Comparative Analysis: Phrase vs. Clause

|   |   |   |
|---|---|---|
|Feature|Phrase|Clause|
|**Contains Subject + Predicate?**|No|Yes|
|**Expresses a Complete Thought?**|No|Often (if Independent)|
|**Sentence Role**|Acts as a constituent unit|Acts as a sentence or a major structural section of one|
|**Sample Constructions**|_the old house_, _in the morning_|_she left early_, _because it rained_|

## 4. Computational Grammar Frameworks

Grammar frameworks define the formal rules used to position words, phrases, and clauses when constructing sentences. Computational linguistics primarily relies on two major grammatical paradigms:

### 1. Dependency Grammar

Dependency grammar models syntactic structure strictly through **word-to-word relationships**. It prioritizes direct dependencies between words rather than constituent phrase groupings.

- **Core Principle:** Every word in a sentence depends on another word, with exactly **one exception**.
- **The Root:** The single word in a sentence that has no parent dependency is designated as the **root** of the sentence (typically the main verb).
- **Dependency Arcs:** Directed, labeled relations link head words to their dependents (e.g., `nsubj` for nominal subject, `obj` for object, `amod` for adjectival modifier, `det` for determiner, `prep` for prepositional modifier).

### 2. Constituency Grammar (Phrase Structure Grammar)

Constituency grammar models sentence structure by breaking text down into nested, hierarchically ordered **constituents** (phrases and sub-phrases).

- **Core Principle:** Sentences are constructed from phrase components built according to formal **phrase structure rules**.
- **Organization:** Words belong to lexical categories, which form the head words of larger phrases (NPs, VPs, PPs), which in turn combine to form clauses and sentences.
- **Output:** Generates a nested constituency tree reflecting the grammatical hierarchy rather than direct point-to-point word dependencies.