# Summary: Annotation and Evaluation of Information Extraction

*Lecture 6 of Advanced Text Analytics (ATA), worked version: `ATA_L06_Annotation_and_Evaluation_IE_svolto.ipynb`.*

This lesson connects two topics that are often taught separately: the way humans annotate text, and the way a machine's output is scored. The recurring message is that a performance number means nothing on its own. It only has a meaning relative to the annotation scheme, the guidelines, and the matching rules that produced it.

## 1. Span annotation

The atomic unit of an information extraction annotation is a character span, represented as a triple `(start, end, label)`. In the sentence `"The patient has breast cancer."` the annotation `(16, 29, 'DISEASE')` points at `breast cancer`.

Two points worth internalizing. Offsets are half-open: `end` is the position one character past the last character, so `end - start` equals the length of the span. And offsets are the ground representation. Token-level labels, JSON exports, conll files: all of them are projections of those two integers plus a label.

**Exercises solved.** In `"BRCA1 mutations increase breast cancer risk."` the spans are `(0, 5, 'GENE')` for `BRCA1` and `(25, 38, 'DISEASE')` for `breast cancer`. In the legal sentence `"The Court applied Article 6 of Regulation (EU) 2016/679."` they are `(18, 27, 'LEGAL_PROVISION')` for `Article 6` and `(31, 55, 'LEGISLATION')` for `Regulation (EU) 2016/679`.

## 2. BIO encoding

BIO attaches a tag to each token: `B` for the first token of an entity, `I` for the following tokens inside it, `O` for everything else. It is a per-token view of spans that already exist.

**Exercise solved.** The legal sentence encodes as:

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

The `B-` tag is not decoration. Without it, two adjacent entities of the same type would be indistinguishable from one long entity.

## 3. BILOU encoding

BILOU uses five tags instead of three: `B` beginning, `I` inside, `L` last, `O` outside, `U` for an entity that fits in a single token. In the running example, `BRCA1` becomes `U-GENE` and `breast cancer` becomes `B-DISEASE L-DISEASE`. BIO and BILOU carry the same information; the choice between them is a modelling decision, not an annotation one.

## 4. Annotation ambiguity

For a phrase like *Article 6 of the GDPR*, should the span be `[Article 6]` or `[Article 6 of the GDPR]`? The lesson gives the only honest answer: the guidelines decide. The right unit depends on the task definition, on the desired granularity, on whether legislation is annotated as a separate entity, and on what the downstream consumer of the annotation needs. Annotator consistency with the written guidelines matters more than any claim that one option is objectively correct.

## 5. Inter-annotator agreement

Two annotators label six items and agree on five: raw agreement 0.833. Raw agreement overstates things whenever a class dominates, because annotators agree on those items by chance. Cohen's kappa corrects for that:

```text
kappa = (Po - Pe) / (1 - Pe)
```

For the running example `kappa = 0.75`.

The interpretation cuts both ways. High agreement means the two annotators applied the scheme consistently; it does not mean the scheme was applied correctly. Low agreement is diagnostic: unclear guidelines, genuinely hard cases, or annotators who need more training. Fixing disagreement without looking at its cause produces an annotation set that is consistent and uniformly wrong.

## 6. Precision, recall, and F1

Precision asks: of everything the system extracted, how much was correct? Recall asks: of everything that should have been extracted, how much was found? F1 combines them into one number:

```text
P = TP / (TP + FP)      R = TP / (TP + FN)      F1 = 2 * P * R / (P + R)
```

**Exercises solved.** With gold `{BRCA1, TP53, breast cancer, lung cancer}` and prediction `{BRCA1, EGFR, breast cancer}`: TP=2, FP=1, FN=2, so P = 0.667, R = 0.5, F1 = 0.571.

F1 is only one way to summarize. With P = 1.0 and R = 0.2, the arithmetic mean is 0.6 while F1 is 0.333. Because it is a harmonic mean, F1 is pulled toward whichever of the two is smaller. That is usually the behaviour we want, but it is a choice: F2 leans toward recall, F0.5 toward precision. Whatever the summary, report the two components next to it.

## 7. Exact versus relaxed matching

Under exact matching, predicted and gold spans must share offsets and label. Gold `[breast cancer]` against prediction `[cancer]` is a miss, even though the spans overlap. A boundary slip therefore costs twice: one false positive and one false negative.

Under relaxed matching, a prediction gets credit when the label matches and the spans overlap, with each gold span consumed at most once. The conditions are: the gold annotation has not already been matched, the labels are identical, and the two spans share some text. In the worked example, exact matching gives F1 = 0.5 and relaxed matching gives F1 = 1.0. The gap between those two numbers is protocol, not model quality, so any reported score must state which protocol produced it.

Related idea: a **boundary error** (`[cancer]` instead of `[breast cancer]`) and a **type error** (`[breast cancer]DISEASE` labelled `DRUG`) are different failures. They call for different fixes, so it pays to keep them apart during error analysis.

## 8. Micro versus macro averaging

On the per-class counts of the lesson (GENE, DISEASE, DRUG):

```text
Per class F1:  GENE 0.90, DISEASE 0.73, DRUG 0.27
Micro:         P 0.80, R 0.825, F1 0.812
Macro F1:      0.631
```

Micro pooling counts all entities together, so frequent classes dominate the result. Macro averaging scores each class and averages those scores, so the rare DRUG class weighs as much as GENE. Micro answers "how good is the system on the average entity"; macro answers "how good on the average class". For imbalanced IE tasks they diverge sharply, which is exactly why both should be reported.

## 9. Relation extraction evaluation

The scored unit becomes a tuple `(arg1, relation_type, arg2)`. A prediction counts as correct when the relation type and both arguments match. In the worked example the system gets one of two relations right: TP=1, FP=1, FN=1, F1 = 0.5. Relations also have direction in most schemes, so `(A, PART_OF, B)` and `(B, PART_OF, A)` are different predictions.

## 10. Pipeline evaluation

An entity recognition error propagates downstream: the relation extractor works on whatever spans it receives. The lesson separates two protocols. **Component evaluation** feeds gold entities to the relation extractor and measures that component in isolation. **End-to-end evaluation** runs the whole pipeline on raw text and scores the final output. The first is an upper bound, the second is what users experience. Publishing only one of them hides half the story.

## 11. The two applied exercises

**Biomedical.** Gold: `BRCA1` GENE, `breast cancer` DISEASE, and `ASSOCIATED_WITH(BRCA1, breast cancer)`. System: `BRCA1` GENE, `cancer` DISEASE, `ASSOCIATED_WITH(BRCA1, cancer)`. Exact NER gives TP=1, FP=1, FN=1 (F1 = 0.5), because `cancer` has neither the boundaries nor the full span of `breast cancer`. Under relaxed matching the overlap gives TP=2, FP=0, FN=0 (F1 = 1.0). The relation is wrong under exact argument matching: the arguments differ, so F1 = 0.

**Legal.** Gold: `[Article 6]LEGAL_PROVISION`, `[Regulation (EU) 2016/679]LEGISLATION`, plus a `PART_OF` relation. Prediction: one merged span `[Article 6 of Regulation (EU) 2016/679]LEGAL_PROVISION`. Exact matching gives TP=0, FP=1, FN=2. Relaxed matching gives TP=1, FN=1 (F1 = 0.667), since the merged span overlaps the provision with the right label but never matches the legislation label. The `PART_OF` relation is never predicted. Whether the prediction is acceptable depends on the annotation scheme: if the task expects two separate spans it is wrong; if a citation counts as a single legal reference entity it may be right. Without the scheme, the question has no answer.

## 12. The evaluation pipeline

```text
TASK DEFINITION -> ANNOTATION SCHEME -> GUIDELINES -> GOLD STANDARD
                -> EVALUATION METRIC -> REPORTED PERFORMANCE
```

Each stage constrains the next. Two systems score comparably only when the whole chain matches, which is why numbers taken from different papers often disagree for reasons that have nothing to do with the technology.

## 13. Common evaluation mistakes

- evaluating on training data;
- tuning repeatedly against the test set;
- reporting accuracy on imbalanced IE tasks;
- leaving the span-matching criterion unspecified;
- ignoring class imbalance;
- comparing scores computed under different annotation schemes;
- reporting one average without saying whether it is micro or macro;
- treating the gold standard as error-free.

## 14. Final exercise

System A: P = 0.95, R = 0.55. System B: P = 0.78, R = 0.82.

1. F1(A) = 2 * 0.95 * 0.55 / 1.5 = 0.697. F1(B) = 2 * 0.78 * 0.82 / 1.6 = 0.7995, printed as 0.799 by the notebook.
2. If missing entities is costly, take B: its recall is far higher, so it leaves fewer gold entities behind.
3. If false positives are costly, take A: its precision is higher, so its predictions are more often right.

The lesson of this exercise is that the choice between two systems is a cost decision. F1 ranks them both the same way here, but the reason to prefer one over the other comes from the application, not from the metric.

## Take-home messages

1. Annotation defines what the system is expected to extract.
2. A gold standard is the output of an explicit annotation procedure, not unquestionable truth.
3. Precision, recall, and F1 capture different aspects of performance; report them together.
4. Exact versus relaxed matching can move a score substantially, from 0.5 to 1.0 in the examples above.
5. Micro and macro averaging answer different questions.
6. Evaluation choices are part of the task definition. State them with every number you publish.
