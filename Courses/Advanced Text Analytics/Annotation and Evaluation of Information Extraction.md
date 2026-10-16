---
type: course-note
date: 2026-10-16
tags:
  - course
  - advanced-text-analytics
  - evaluation
  - information-extraction
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L06]]"
ai-first: true
---

## For Future Agent

Study note for the [[Advanced Text Analytics]] course (Lecture 6) on **annotation and evaluation of Information Extraction (IE)** - how humans mark spans and how machine output is scored. Created 2026-10-16 from [[ATA_L06]] (English, worked NotebookLM export). Use it when a question involves span annotation, BIO/BILOU, inter-annotator agreement, precision/recall/F1, exact vs relaxed matching, micro vs macro averaging, relation-extraction scoring, or pipeline evaluation. The recurring thesis: **a performance number means nothing on its own** - it only has meaning relative to the annotation scheme, the guidelines, and the matching rules that produced it.

## 1. Why Annotation and Evaluation Belong Together

These two topics are usually taught separately, but the lecture connects them because they are two ends of one chain: the way humans annotate text determines the way a machine's output can be scored. Annotation defines the target; evaluation measures the distance from it. Without the scheme, no score is interpretable.

## 2. Span Annotation

The atomic unit of an IE annotation is a **character span**, represented as a triple:

$$\text{annotation} = (\text{start},\ \text{end},\ \text{label})$$

In `"The patient has breast cancer."` the annotation `(16, 29, 'DISEASE')` points at `breast cancer`.

Two properties worth internalizing:

- **Offsets are half-open**: `end` is the position one character *past* the last character, so `end - start` equals the span length.
- **Offsets are the ground representation**: token-level labels, JSON exports, and conll files are all *projections* of those two integers plus a label.

Worked exercise:

| Sentence fragment | Span |
|---|---|
| `"BRCA1 mutations increase breast cancer risk."` | `(0, 5, 'GENE')` for `BRCA1` |
| same | `(25, 38, 'DISEASE')` for `breast cancer` |
| `"The Court applied Article 6 of Regulation (EU) 2016/679."` | `(18, 27, 'LEGAL_PROVISION')` for `Article 6` |
| same | `(31, 55, 'LEGISLATION')` for `Regulation (EU) 2016/679` |

## 3. BIO Encoding

BIO attaches a tag to **each token**:

- `B` - first token of an entity (*beginning*);
- `I` - a following token *inside* the entity;
- `O` - everything else (*outside*).

It is a per-token view of spans that already exist. The legal sentence encodes as:

```text
The             O
Court           O
applied         O
Article         B-LEGAL_PROVISION
6               I-LEGAL_PROVISION
of              O
Regulation      B-LEGISLATION
(EU)            I-LEGISLATION
2016/679        I-LEGISLATION
.               O
```

The `B-` tag is **not decoration**: without it, two adjacent entities of the same type would be indistinguishable from one long entity.

## 4. BILOU Encoding

BILOU uses five tags instead of three:

| Tag | Meaning |
|---|---|
| `B` | beginning |
| `I` | inside |
| `L` | last |
| `O` | outside |
| `U` | unit - entity that fits in a single token |

In the running example, `BRCA1` becomes `U-GENE` and `breast cancer` becomes `B-DISEASE L-DISEASE`.

**BIO and BILOU carry the same information.** The choice between them is a *modelling* decision, not an *annotation* one.

## 5. Annotation Ambiguity

For a phrase like *Article 6 of the GDPR*, should the span be `[Article 6]` or `[Article 6 of the GDPR]`? The lecture gives the only honest answer: **the guidelines decide.** The right unit depends on:

- the task definition;
- the desired granularity;
- whether legislation is annotated as a separate entity;
- what the downstream consumer of the annotation needs.

**Annotator consistency with the written guidelines matters more than any claim that one option is objectively correct.**

## 6. Inter-Annotator Agreement

Two annotators label six items and agree on five: **raw agreement = 0.833**. But raw agreement *overstates* reliability whenever one class dominates, because annotators agree on those items by chance. **Cohen's kappa** corrects for chance:

$$\kappa = \frac{P_o - P_e}{1 - P_e}$$

where $P_o$ is observed agreement and $P_e$ is the agreement expected by chance. For the running example, $\kappa = 0.75$.

Interpretation cuts both ways:

- **High agreement** means the two annotators applied the scheme **consistently**; it does *not* mean the scheme was applied **correctly**.
- **Low agreement is diagnostic**: unclear guidelines, genuinely hard cases, or annotators who need more training.

Fixing disagreement *without* looking at its cause produces an annotation set that is **consistent and uniformly wrong**.

## 7. Precision, Recall, and F1

- **Precision** - of everything the system extracted, how much was correct?
- **Recall** - of everything that should have been extracted, how much was found?

$$P = \frac{TP}{TP + FP} \qquad R = \frac{TP}{TP + FN} \qquad F_1 = 2\cdot\frac{P\cdot R}{P + R}$$

Worked exercise: with gold `{BRCA1, TP53, breast cancer, lung cancer}` and prediction `{BRCA1, EGFR, breast cancer}`: $TP=2$, $FP=1$, $FN=2$, so $P = 0.667$, $R = 0.5$, $F_1 = 0.571$.

**F1 is only one summary.** With $P = 1.0$ and $R = 0.2$, the arithmetic mean is $0.6$ while $F_1 = 0.333$: because F1 is a **harmonic mean**, it is pulled toward whichever component is smaller. That is usually the desired behaviour, but it is a *choice*:

| Summary | Leans toward |
|---|---|
| $F_2$ | recall |
| $F_1$ | balanced |
| $F_{0.5}$ | precision |

Whatever the summary, **report the two components next to it.**

## 8. Exact versus Relaxed Matching

- **Exact matching** - predicted and gold spans must share offsets *and* label. Gold `[breast cancer]` against prediction `[cancer]` is a *miss*, even though the spans overlap. A boundary slip therefore costs twice: one false positive and one false negative.
- **Relaxed matching** - a prediction gets credit when the *label matches* and the spans *overlap*, with each gold span consumed at most once.

The relaxed-matching conditions: the gold annotation is not already matched, the labels are identical, and the two spans share some text.

In the worked example, exact matching gives $F_1 = 0.5$ and relaxed matching gives $F_1 = 1.0$. **The gap between those two numbers is protocol, not model quality**, so any reported score must state which protocol produced it.

Related distinction to keep during error analysis:

| Error type | Example | Fix |
|---|---|---|
| Boundary error | `[cancer]` instead of `[breast cancer]` | adjust span detection |
| Type error | `[breast cancer]` labelled `DRUG` instead of `DISEASE` | adjust classifier |

## 9. Micro versus Macro Averaging

On the lecture's per-class counts (GENE, DISEASE, DRUG):

```text
Per-class F1:  GENE 0.90, DISEASE 0.73, DRUG 0.27
Micro:         P 0.80, R 0.825, F1 0.812
Macro F1:      0.631
```

- **Micro** pools all entities together -> frequent classes dominate. It answers *"how good is the system on the average entity?"*
- **Macro** scores each class and averages the scores -> the rare DRUG class weighs as much as GENE. It answers *"how good is the system on the average class?"*

For imbalanced IE tasks the two diverge sharply, which is exactly why **both should be reported.**

## 10. Relation Extraction Evaluation

The scored unit becomes a tuple:

$$\text{prediction} = (\text{arg}_1,\ \text{relation\_type},\ \text{arg}_2)$$

A prediction counts as correct when the **relation type and both arguments match**. In the worked example the system gets one of two relations right: $TP=1$, $FP=1$, $FN=1$, $F_1 = 0.5$.

Relations also have **direction** in most schemes, so `(A, PART_OF, B)` and `(B, PART_OF, A)` are different predictions.

## 11. Pipeline Evaluation

An entity-recognition error **propagates downstream**: the relation extractor works on whatever spans it receives. The lecture separates two protocols:

| Protocol | Input | Meaning |
|---|---|---|
| **Component evaluation** | gold entities fed to the relation extractor | measures one component in isolation - an **upper bound** |
| **End-to-end evaluation** | whole pipeline on raw text | what users actually experience |

Publishing only one of them **hides half the story.**

## 12. The Two Applied Exercises

### 12.1 Biomedical

Gold: `BRCA1` GENE, `breast cancer` DISEASE, and `ASSOCIATED_WITH(BRCA1, breast cancer)`.
System: `BRCA1` GENE, `cancer` DISEASE, `ASSOCIATED_WITH(BRCA1, cancer)`.

- Exact NER: `cancer` has neither the boundaries nor the full span of `breast cancer` -> $TP=1$, $FP=1$, $FN=1$ ($F_1 = 0.5$).
- Relaxed matching: the overlap gives $TP=2$, $FP=0$, $FN=0$ ($F_1 = 1.0$).
- Relation: wrong under exact argument matching (arguments differ) -> $F_1 = 0$.

### 12.2 Legal

Gold: `[Article 6]LEGAL_PROVISION`, `[Regulation (EU) 2016/679]LEGISLATION`, plus a `PART_OF` relation.
Prediction: one merged span `[Article 6 of Regulation (EU) 2016/679]LEGAL_PROVISION`.

- Exact matching: $TP=0$, $FP=1$, $FN=2$.
- Relaxed matching: $TP=1$, $FN=1$ ($F_1 = 0.667$) - the merged span overlaps the provision with the right label but never matches the legislation label.
- The `PART_OF` relation is never predicted.

**Whether the prediction is acceptable depends on the annotation scheme**: if the task expects two separate spans it is wrong; if a citation counts as a single legal-reference entity it may be right. *Without the scheme, the question has no answer.*

## 13. The Evaluation Pipeline

```text
TASK DEFINITION -> ANNOTATION SCHEME -> GUIDELINES -> GOLD STANDARD
                -> EVALUATION METRIC -> REPORTED PERFORMANCE
```

Each stage **constrains the next**. Two systems score comparably only when the *whole chain* matches - which is why numbers from different papers often disagree for reasons that have nothing to do with the technology.

## 14. Common Evaluation Mistakes

- evaluating on training data;
- tuning repeatedly against the test set;
- reporting accuracy on imbalanced IE tasks;
- leaving the span-matching criterion unspecified;
- ignoring class imbalance;
- comparing scores computed under different annotation schemes;
- reporting one average without saying whether it is micro or macro;
- treating the gold standard as error-free.

## 15. Final Exercise and Take-Home

System A: $P = 0.95$, $R = 0.55$. System B: $P = 0.78$, $R = 0.82$.

1. $F_1(A) = 2\cdot0.95\cdot0.55 / 1.5 = 0.697$. $F_1(B) = 2\cdot0.78\cdot0.82 / 1.6 = 0.7995$, printed as **0.799**.
2. If **missing entities is costly**, take **B**: its recall is far higher, so it leaves fewer gold entities behind.
3. If **false positives are costly**, take **A**: its precision is higher, so its predictions are more often right.

The lesson: **the choice between two systems is a cost decision.** F1 ranks them the same way here, but the reason to prefer one comes from the application, not from the metric.

Take-home messages:

1. Annotation defines what the system is expected to extract.
2. A gold standard is the output of an explicit annotation procedure, **not unquestionable truth**.
3. Precision, recall, and F1 capture different aspects of performance; report them together.
4. Exact vs relaxed matching can move a score substantially (0.5 -> 1.0 in the examples above).
5. Micro and macro averaging answer different questions.
6. **Evaluation choices are part of the task definition** - state them with every number you publish.

## Related

- [[NLP Linguistic Structures and Annotation]] - the annotation layers (POS, morphology, dependencies) that IE labels are built on
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L06]] (AY 2026/2027)
