---
type: course-note
date: 2026-10-28
tags:
  - course
  - advanced-text-analytics
  - notebook
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L05]]"
ai-first: true
---

## For Future Agent

Notebook companion for [[Knowledge Resources for Text Analytics]] (Lecture 5), extracted 2026-10-28: it collects only the passages documenting what the lecture's Python notebook actually runs (spaCy `PhraseMatcher` demos, the `broader_concepts` traversal, the knowledge-base triple store). The theory of resource types stays in the parent note. Use it when a question is about the concrete code behavior of the demos rather than the concepts.

## 1. Demo Setup

Every code demo runs on spaCy (`en_core_web_sm`) and its `PhraseMatcher`, kept deliberately small so each resource type's *matching behavior* is visible without engineering noise.

## 2. Gazetteer Matching

In the notebook, a `PhraseMatcher` built over the gazetteer's keys tags `BRCA1` and `TP53` correctly in a sample sentence.

## 3. Taxonomy Traversal

The notebook walks the broader/narrower chain with a small `broader_concepts` function returning `['Ischemic heart disease', 'Cardiovascular disease', 'Disease']`.

## 4. Knowledge-Base Triples

The notebook stores triples such as `("BRCA1", "associated_with", "breast cancer")` and answers a query by **filtering on subject and relation**.

## Related

- [[Knowledge Resources for Text Analytics]] - parent theory note (resource types, comparison table, pipeline position)
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L05]] (AY 2026/2027)
