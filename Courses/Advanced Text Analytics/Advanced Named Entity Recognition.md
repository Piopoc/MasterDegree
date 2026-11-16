---
type: course-note
date: 2026-11-10
tags:
  - course
  - advanced-text-analytics
  - ner
  - span-based-ner
  - zero-shot-ner
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L12]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 12) on **advanced NER: span-based extraction, open-domain models, and generalist frameworks**. Created 2026-11-10 from `raw/ATA_L12.md`. Use it when a question involves the move from token-level BIO sequence labeling to span classification, how span vectors are pooled from token hidden states, why candidate enumeration creates structural class imbalance, how GLiNER turns the entity type into an input embedding, how UniversalNER distills teacher-LLM supervision into a small student, what hard zero-shot evaluation requires (ZeroNER), or how to pick among the paradigms in the selection table. Prerequisite: the token-level formulation this lecture relaxes lives in [[Supervised Named Entity Recognition - Notebook]] and [[spaCy Toy NER - Notebook]] (Lecture 11). The concrete walkthroughs (spaCy `SpanCategorizer` training, span enumeration math, toy span classifier, cosine label matching, optional GLiNER demo, threshold sweep) live in [[Advanced Named Entity Recognition - Notebook]].

## 1. From Token Classification to Span Classification

A standard NER model is a token classifier: the Transformer emits a hidden vector $h_i$ per token and a classification head projects $h_i$ directly onto a BIO tag. This baseline rests on two strict assumptions:

1. **Token-level BIO decoding** - every token receives one tag, and entity boundaries are inferred from tag transitions.
2. **Fixed output inventory** - the set of entity classes is a fixed output layer decided at training time; adding a type means retraining.

Lecture 12 relaxes both. The prediction unit becomes a contiguous token interval $(i, j)$, so the model scores candidate spans directly instead of reconstructing boundaries from transitions:

```text
token classification:   h_i                 -> BIO tag per token
span classification:    g(i, j)             -> entity type / NONE
open / generalist:      g(i, j) + e(type)   -> compatibility score
```

Two consequences follow: overlapping and nested entities become representable (a single BIO sequence cannot carry two labels on one interval), and - in the third line above - the entity type can be supplied at inference time instead of being baked into the output layer.

## 2. Span Representations

### 2.1 Candidate Enumeration

A span is a pair (start, end) with end exclusive. Without limits, an $n$-token sentence yields $n(n+1)/2$ contiguous spans: the lecture's 6-token example gives 21 candidates, or 15 with `max_width = 3`. The `max_width` constraint is itself a structural modeling choice - a small value enumerates less computation but cannot detect long entity names.

### 2.2 The Span Vector

Each candidate interval is pooled into one fixed-size vector. The lecture and the companion walkthrough use a transparent concatenation:

$$g(i,j) = [\mathbf{h}_{start} \,;\, \mathbf{h}_{end-1} \,;\, \mathbf{h}_{mean} \,;\, w]$$

| Component | Meaning |
|---|---|
| $\mathbf{h}_{start}$ | contextual hidden vector of the first token in the span |
| $\mathbf{h}_{end-1}$ | contextual hidden vector of the last token in the span |
| $\mathbf{h}_{mean}$ | element-wise mean across all token representations in the span |
| $w$ | span width (token count), a scalar |

With hidden size $h = 4$ the span vector has $4 + 4 + 4 + 1 = 13$ dimensions. The exact formulation is a design choice: real systems may use boundary vectors only, attention pooling over the span, learned width embeddings, or additional syntactic and lexical features.

### 2.3 Structural Class Imbalance

Enumerating every interval up to `max_width` means most candidates are not entities: the majority of spans receive the negative label `NONE`. The imbalance is not an accident of the data but a consequence of the enumeration strategy - candidate pruning and negative-example sampling are therefore core components of any span system, alongside learned span proposals such as a separate SpanFinder.

## 3. Open-Domain, Zero-Shot, and Generalist Extraction

Open-domain NER removes the fixed label restriction: the target entity type becomes an input at inference time rather than a fixed output parameter.

### 3.1 GLiNER: Label-Conditioned Spans

GLiNER takes two inputs at inference time - the target text and a list of requested entity labels - and scores compatibility between each candidate span vector $g(span)$ and the text embedding of the requested type $e(type)$:

$$\text{Score}(span, type) = g(span) \cdot e(type)$$

Arbitrary entity types can be requested on demand while preserving fast bi-encoder inference efficiency - no generative LLM involved.

### 3.2 UniversalNER: Targeted Distillation

UniversalNER reaches the same flexibility from the opposite direction: it distills broad entity extraction capability from a large teacher LLM into a smaller, task-focused student model.

- **Supervision source**: open-NER instruction data generated by a large teacher LLM.
- **Evaluation scope**: 43 datasets across 9 distinct domains, including biomedicine, law, finance, programming, and social media.
- **Core question**: can a small student preserve LLM extraction flexibility without maintaining the full general-purpose LLM weight footprint?

### 3.3 ZeroNER: Hard Zero-Shot Evaluation

Zero-shot claims are easy to fake, so the protocol matters. Cocchieri et al. (2025) define:

- **Hard zero-shot** - testing on entity types where the model has received no labeled training examples at all.
- **Description-driven extraction** - unseen entity types need meaningful natural-language descriptions rather than isolated, potentially ambiguous label names.

Indirect supervision for the supposedly unseen type, semantically equivalent labels appearing in training data, and leakage through benchmark construction are the failure modes this stricter protocol guards against.

## 4. Threshold Tuning at Inference

Generalist extractors return a score for each candidate; a confidence threshold decides what counts as an entity:

```python
for threshold in thresholds:
    entities = gliner_model.predict_entities(text, labels, threshold=threshold)
```

| Threshold | Effect |
|---|---|
| higher | retains fewer spans - precision increases, recall may drop |
| lower | retains more spans - recall increases, precision may drop |

The threshold controls the precision-recall trade-off: it is an evaluation decision, not a cosmetic parameter, and must be tuned and reported like any other operating point.

## 5. Choosing a Paradigm

| Paradigm | Key structural property | Operational requirement | Ideal use case |
|---|---|---|---|
| Supervised sequence / span NER | fixed output label parameters | stable ontology and ample labeled data | high-throughput production with constant schema |
| Span-based NER | explicit interval enumeration | support for nested entities or joint NER and RE | complex domain texts with overlapping entities |
| Generalist / open NER (GLiNER) | label embeddings as inference inputs | dynamic label requirements without LLM latency | flexible extraction with low operational latency |
| Distilled student NER (UniversalNER) | LLM instruction distillation | multi-domain coverage (biomedicine, law, finance) | broad cross-domain extraction without large models |
| Description-driven zero-shot (ZeroNER) | natural-language entity descriptions | extraction of unseen types without training data | unseen entity types with detailed definitions |
| Generative LLM extraction | complex instruction and reasoning | structured-output controls and high compute budget | complex reasoning and joint extraction tasks |

The Transformer representation engine stays effective across all paradigms. The key architectural shift is changing the unit of prediction from tokens to spans, and moving from fixed output heads to inference-time label representations.

## 6. Take-Home Messages

1. Span NER replaces token BIO decoding with direct classification of candidate intervals - overlapping and nested entities become representable by construction.
2. Span vectors pool token hidden states (boundaries + mean + width in the toy; richer encoders in real systems); the exact form is a modeling choice.
3. Candidate enumeration grows quadratically, so `max_width`, pruning, negative sampling, and learned span proposals are structural, not optional.
4. The vast majority of candidates are `NONE`: class imbalance is built into the enumeration strategy.
5. GLiNER makes the entity type an input embedding compared against span embeddings; UniversalNER reaches open NER by distilling a teacher LLM instead.
6. Zero-shot evaluation needs a strict protocol (hard zero-shot, descriptions, no leakage), and thresholds encode the precision-recall trade-off.

## 7. Suggested Review Questions

1. Which two assumptions of token-level NER does span classification relax, and what does each relaxation enable?
2. How is $g(i,j)$ built in the lecture, and why is its 13-dimensional size not a fixed rule?
3. Why does candidate enumeration create class imbalance, and which engineering components counter it?
4. In GLiNER, what is compared with what - and why is that different from a fixed softmax head?
5. How does UniversalNER obtain open-NER ability, and what question did the distillation experiment test?
6. What does ZeroNER's hard zero-shot protocol rule out, and why do bare label names fail?

## Related

- [[Advanced Text Analytics]] - course hub
- [[Supervised Named Entity Recognition - Notebook]] - Lecture 11a: the token-level BIO formulation this lecture relaxes
- [[spaCy Toy NER - Notebook]] - Lecture 11b: the transition-based spaCy `ner` this lecture replaces with `SpanCategorizer`
- [[Advanced Named Entity Recognition - Notebook]] - companion walkthroughs for both L12 notebooks

## Sources

- [[ATA_L12]] (AY 2026/2027)
- Luan et al. (2019). *A General Framework for Information Extraction using Dynamic Span Graphs (DyGIE++)* (EMNLP-IJCNLP) - cited via the ATA_L12 source above
- Zaratiana et al. (2024). *GLiNER: Generalist Model for Named Entity Recognition using Bidirectional Transformer* (NAACL) - cited via the ATA_L12 source above
- Zhou et al. (2024). *UniversalNER: Targeted Distillation from Large Language Models for Open Named Entity Recognition* (ICLR) - cited via the ATA_L12 source above
- Cocchieri et al. (2025). *ZeroNER: Fueling Zero-Shot Named Entity Recognition via Entity Type Descriptions* (Findings of ACL) - cited via the ATA_L12 source above
