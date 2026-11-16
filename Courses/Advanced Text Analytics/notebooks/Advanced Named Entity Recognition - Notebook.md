---
type: course-note
date: 2026-11-10
tags:
  - course
  - advanced-text-analytics
  - notebook
  - ner
  - spacy
  - gliner
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L12_notebook_explained]]"
ai-first: true
---

## For Future Agent

Notebook companion for Lecture 12: theory of both L12 notebooks - `ATA_L12_spaCy_SpanCategorizer_Toy_NER.ipynb` (part A: a trainable spaCy `SpanCategorizer` that replaces Lecture 11's `EntityRecognizer`) and `ATA_L12_NER_II_Span_Open_ZeroShot_Generalist_companion.ipynb` (part B: NumPy-only span NER mechanics, open-label matching, optional GLiNER demo), extracted 2026-11-10 from the cell-by-cell walkthrough in raw. It covers the two relaxed assumptions of the lecture (span as prediction unit, label as input), `doc.spans["sc"]` and nested entities, candidate enumeration and the 13-dim span vector, the hand-written logistic-regression span classifier, cosine label matching and description-driven labels, hard zero-shot caveats, the threshold sweep, and generalist NER vs a generative LLM. Central caveats: both toys demonstrate **mechanics, never model quality** (3 and 6-token toys), the GLiNER section is optional with every cell commented out (nothing auto-installs or auto-downloads), and the final threshold DataFrame cell uses an undefined `scores` variable that must be assembled from the `results` list. Theory note: [[Advanced Named Entity Recognition]].

## 1. Position in the Course

Lecture 12 changes two things at once:

```text
L11:  token representation            -> fixed BIO classifier    -> one label per token
L12a: candidate span (i, j)           -> span representation     -> entity type / NONE
L12b: candidate span + requested type -> compatibility score
```

- **Step 1** changes the *prediction unit*: whole candidate intervals instead of per-token tags, which makes overlapping and nested entities representable.
- **Step 2** changes the *representation of the label*: the entity type becomes an input embedding compared against span embeddings, which enables open / zero-shot / generalist extraction.

The two notebooks cover one side each:

| Notebook | Focus |
|---|---|
| `ATA_L12_spaCy_SpanCategorizer_Toy_NER.ipynb` | Step 1 in practice: a real, trainable spaCy pipeline (still closed-set) |
| `ATA_L12_NER_II_Span_Open_ZeroShot_Generalist_companion.ipynb` | Steps 1-2 in theory: transparent NumPy toy implementations, plus optional GLiNER demo |

Requirements: A needs `spacy`, `pandas`, `matplotlib` and trains **from scratch** (no pretrained Transformer); B needs NumPy, pandas, matplotlib, plus the optional `gliner` package and a pretrained checkpoint for section 10 onward.

## 2. Notebook A - spaCy SpanCategorizer Toy NER

### 2.1 The Pipeline Contrast

```text
L11: text -> EntityRecognizer                  -> doc.ents
L12: text -> candidate spans -> SpanCategorizer -> doc.spans["sc"]
```

`doc.ents` requires flat, non-overlapping entities; the SpanGroup `doc.spans["sc"]` (the key the spancat pipeline uses by default) stores both spans of the canonical nested example:

```text
University of Padua
              Padua
ORGANIZATION          LOCATION
```

The component is `spancat_singlelabel` because each individual span receives at most one class in this toy (spaCy's multi-label `spancat` variant handles spans that carry several labels simultaneously). Two labels are defined: `ORGANIZATION`, `LOCATION`.

### 2.2 Data: Character Offsets to `Example` Objects

Three sentences whose gold annotations nest a LOCATION inside an ORGANIZATION span ("University of **Padua**"), so the gold data itself exercises the overlapping-span capability.

- A diagnostic cell slices the original string with the stored offsets before training - catches off-by-one annotation bugs early.
- Gold spans go into `reference.spans["sc"]` - the exact key the component reads during training.
- `char_span(..., alignment_mode="contract")` shrinks the character range to token boundaries; no alignment at all returns `None` and the notebook raises an explicit `ValueError`.
- An `Example` pairs a tokenized `predicted` doc with the gold `reference` doc; `nlp.update` computes the loss between them.
- Printing token indices `(span.start, span.end)` next to character offsets makes the conversion explicit - a classic source of NER bugs.

### 2.3 Training From Scratch

`nlp.initialize(get_examples=...)` builds the statistical weights (the component infers its feature schema and label set). The default encoder is **tok2vec, not BERT** - no pretrained model anywhere. The loop is standard spaCy supervised training:

```python
N_EPOCHS = 150
for epoch in range(N_EPOCHS):
    random.shuffle(examples)
    losses = {}
    nlp.update(examples, sgd=optimizer, drop=0.2, losses=losses)
    loss_history.append(losses.get("spancat_singlelabel", 0.0))
```

`drop=0.2` applies dropout; the recorded loss is specifically the `spancat_singlelabel` component. 150 epochs over 3 examples is a deliberately over-parameterized toy run (`SEED = 42`): the decreasing loss shows fitting to this tiny dataset, **never robust generalization**.

### 2.4 Inference and Inspection

Predictions are read from `doc.spans["sc"]`, not `doc.ents`:

```python
doc = nlp("The University of Padua collaborates with researchers in Padua.")
for span in doc.spans["sc"]:
    print(repr(span.text), "->", span.label_)
```

Per-span confidences live in `spans.attrs.get("scores")` - useful for thresholding decisions later. Four unseen sentences (different universities and cities than the three training ones) give a qualitative check only; with 3 training examples the results are indicative rather than reliable.

### 2.5 Candidate Combinatorics and SpanFinder

Enumerating every contiguous span up to width 4 over the 9-token sample sentence produces the full candidate table in a DataFrame. The combinatorial explosion is why span systems use a **maximum span width**, **candidate pruning**, **negative-example sampling**, and **learned span proposal**. The optional extension sketches a two-stage pipeline (not implemented):

```text
text -> SpanFinder        -> "where might an interesting span be?"
     -> candidates
     -> SpanCategorizer   -> "what label should that span receive?"
     -> labeled spans
```

decoupling span proposal from span classification.

### 2.6 Error Analysis and the Bridge to GLiNER

The exercise asks to categorize each error as one of: **missed span** (false negative), **spurious span** (false positive), **boundary error** (right type, wrong extent), **type error** (right extent, wrong label), or **nested-span error** (failure specific to overlapping ground truth).

Closing comparison - the notebook's one-line summary of the architectural change:

| Lecture 11 `ner` | Lecture 12 `spancat_singlelabel` |
|---|---|
| transition-based entity recognizer | span categorizer |
| `doc.ents` | `doc.spans["sc"]` |
| flat named entities | overlapping spans allowed |
| entity recognition | more general span classification |

This notebook stays **supervised and closed-set**: `ORGANIZATION` and `LOCATION` were defined *before* training. GLiNER changes the task - `text + requested labels` are supplied at inference time.

## 3. Notebook B - Span, Open, and Zero-Shot NER

Running example: **"BRCA1 mutations increase breast cancer risk"** (6 tokens), with gold entities `BRCA1 -> GENE` and `breast cancer -> DISEASE`.

### 3.1 Candidate Spans and Gold Labels

```python
def candidate_spans(tokens, max_width=None):
    n = len(tokens)
    if max_width is None:
        max_width = n
    spans = []
    for start in range(n):
        for end in range(start + 1, min(n, start + max_width) + 1):
            spans.append((start, end))
    return spans
```

A span is (start, end) with end exclusive - `(3, 5)` is "breast cancer". For 6 tokens the total is $6 \cdot 7 / 2 = 21$ spans; with `max_width=3` the count drops to `6 + 5 + 4 = 15`, the first and simplest mitigation of quadratic growth (Exercise 1). Mapping every candidate through `gold_span_label` (gold: `{(0, 1): "GENE", (3, 5): "DISEASE"}`, everything else `NONE`) shows the structural imbalance: the large majority of candidate spans are negative examples, so candidate generation and negative-example balancing are first-class design problems.

### 3.2 The Span Vector

Assume - as in L09-L11 - a Transformer produced one hidden vector per token; for transparency the notebook uses a synthetic hand-crafted $H \in \mathbb{R}^{6 \times 4}$, one row per token. Each candidate is pooled into one vector:

$$g(i,j) = [h_i \,;\, h_{j-1} \,;\, \operatorname{mean}(h_i,\ldots,h_{j-1}) \,;\, \text{width}] \in \mathbb{R}^{13}$$

First-token vector (4 dims) + last-token vector (4) + mean vector (4) + width scalar (1) = 13. `span_representation(H, start, end)` concatenates the four parts. The notebook stresses that **the exact representation is a modeling choice**: real systems may use boundary vectors only, attention pooling over the span, learned span-width embeddings, or syntactic and lexical features.

### 3.3 A Toy Supervised Span Classifier

The classifier receives one 13-dim vector per span and predicts one of `NONE / GENE / DISEASE`. Features are stacked into a `(15, 13)` matrix; the training loop is a hand-written multinomial logistic regression:

- `softmax` with max-shift for numerical stability; cross-entropy with a `1e-12` epsilon;
- gradients `grad_logits = (probs - one_hot) / N`, then `grad_W = X.T @ grad_logits`, `grad_b = grad_logits.sum(axis=0)`;
- plain gradient descent, learning rate 0.2, 1000 epochs, loss history recorded and plotted.

A predictions DataFrame then shows, per span: gold label, predicted label, and $P(\text{NONE})$, $P(\text{GENE})$, $P(\text{DISEASE})$. Reflection: $\text{L11: } h_i \to \text{BIO label}$ versus $\text{L12: } g(i,j) \to \text{entity type or NONE}$ - the supervised machinery is similar; the object being classified has changed.

### 3.4 Why Nested Entities Are Representable

"University of Padua" can be scored as `(0, 3) -> ORGANIZATION` and `(2, 3) -> LOCATION` independently - a flat BIO sequence cannot assign two labels to overlapping intervals, while a span system simply scores both candidate intervals. Whether overlapping predictions are kept at decode time is a **decoding/modeling choice**, not a representational impossibility.

### 3.5 Open-Label Matching

The toy span classifier still has a fixed head $W \in \mathbb{R}^{m \times K}$ with $K$ known at training; asking for a new type such as `DRUG` would normally require retraining. The open/generalist shift, stated as a principle: **represent the entity type itself and compare it with candidate spans** - both imagined in the same semantic space:

$$\operatorname{score}(span, type) = \cos(g_{span}, e_{type})$$

Hand-designed span vectors (`BRCA1`, `breast cancer`, `aspirin`) and label vectors (`gene`, `disease`, `drug`) produce a cosine pivot table where each span matches its aligned label. The crucial observation: label `drug` did **not** have to be a fixed output dimension - it is an **input** vector. Section 8.1 adds **description-driven labels**: short labels are ambiguous (`bank` = financial institution vs. river bank), so richer natural-language descriptions specify the target concept better - the motivation behind description-driven zero-shot NER (ZeroNER).

### 3.6 What Zero-Shot Means Here

A strict (hard) zero-shot reading:

```text
target entity type -> no labeled training examples for that type
    -> model receives only the type name / description
    -> must recognize instances of the unseen type
```

Potential problems listed by the notebook: **indirect supervision** for the supposedly unseen type, semantically equivalent labels appearing in training data, and **leakage through benchmark construction**. This is why stricter "hard zero-shot" settings exist (cf. ZeroNER, Cocchieri et al. 2025).

### 3.7 Optional GLiNER Demo

```text
text + requested labels -> bidirectional Transformer
    -> span / label compatibility -> entities
```

The section is **optional**: it requires installing `gliner` and downloading a checkpoint, so every cell is present but **commented out** - nothing auto-installs or auto-downloads:

```python
# %pip install gliner
# from gliner import GLiNER
# gliner_model = GLiNER.from_pretrained("urchade/gliner_small-v2.1")
# entities = gliner_model.predict_entities(text, labels, threshold=0.5)
```

Labels (`["gene", "disease", "drug"]`) are provided *when performing inference*, not baked into the output layer. The point of the experiment: calling again with `alternative_labels = ["gene", "medical condition", "chemical substance", "research activity"]` changes the ontology **without retraining the model**. Exercise 2 applies the same setup to a legal sentence ("The Supreme Court dismissed the appeal filed by Smith against the Ministry of Justice.") with labels `court, legal action, person, government organization, legal document`, asking the reader to rewrite the label descriptions and observe the effect - domain flexibility and description sensitivity in one go.

### 3.8 Thresholds Matter

```text
lower threshold -> more entities -> higher recall  -> more false positives
higher threshold -> fewer entities -> higher precision -> more missed entities
```

A sweep over `thresholds = [0.30, 0.50, 0.70, 0.90]` prints the entities retained with their scores per value, and a final DataFrame adds a boolean `kept @ 0.xx` column per threshold for every candidate. This is an **evaluation choice, not merely a cosmetic parameter**. Caveat if you run the notebook: the final DataFrame cell references a `scores` variable that is not defined as written - it must be assembled from the `results` list collected in the sweep cell.

### 3.9 Generalist NER vs a Generative LLM, and UniversalNER

| | Generalist NER | Generative LLM |
|---|---|---|
| Input | text + labels | natural-language instruction + text |
| Mechanism | specialized encoder -> span scores | autoregressive generation |
| Output | structured entities | generated structured answer |

Trade-off: a general LLM is more flexible, but a specialized NER model is **smaller, faster, and easier to control** for extraction-only tasks. UniversalNER takes a different route to open NER via distillation: large teacher LLM -> broad NER supervision -> targeted instruction-tuning -> smaller open-NER student, evaluated across 43 datasets and 9 domains. Conceptual question: *can broad entity-type generalization be retained while distilling away capabilities irrelevant to NER?*

## 4. How the Two Notebooks Fit Together

| Dimension | Notebook A (SpanCategorizer) | Notebook B (span/open companion) |
|---|---|---|
| Nature | runnable end-to-end training | conceptual toy math + optional GLiNER demo |
| Dependencies | spaCy | NumPy/pandas/matplotlib (+ optional `gliner`) |
| Formulation covered | L12a: span classification (closed-set) | L12a theory **and** L12b open-label matching |
| Prediction unit | span (`doc.spans["sc"]`) | span (`g(i,j)`) |
| Labels | fixed before training | fixed in the toy part; supplied at inference in the GLiNER part |
| Nested/overlapping entities | trained and predicted directly | explained conceptually |
| Key mechanics | Example objects, training loop, scores | enumeration math, span vectors, hand-written logistic regression, cosine label matching, threshold sweep |

**Recommended reading order:** Notebook B first for the theory (sections 1-9), then Notebook A to see the same ideas realized in a production-grade framework, then Notebook B section 10 onward for the open / zero-shot generalist step.

## 5. Key Points

1. Span NER predicts entity types for candidate **token intervals**, not BIO labels per position - overlap and nesting become representable.
2. Candidate growth is quadratic ($n(n+1)/2$); `max_width`, pruning, negative sampling, and learned span proposals are structural responses.
3. Span vectors pool token hidden states (boundaries + mean + width in the toy); the exact form is a modeling choice.
4. A span classifier with a fixed output layer is still **closed-set** - its ontology is fixed at training time.
5. Open/generalist NER moves the label into the input: type embeddings are compared with span embeddings (GLiNER's mechanism).
6. Zero-shot claims need strict protocols - no direct or indirect target-specific supervision, and descriptions beat bare labels (ZeroNER).
7. Generalist NER and generative LLM prompting solve related problems with different flexibility, cost, and control trade-offs; UniversalNER distills the teacher's flexibility into a small student.
8. Thresholds encode precision/recall and must be treated as evaluation decisions.
9. Both toys show mechanics, never model quality - and the GLiNER section stays commented until you choose to run it.

## Related

- [[Advanced Named Entity Recognition]] - Lecture 12 theory note: paradigm survey and selection table
- [[Supervised Named Entity Recognition - Notebook]] - Lecture 11a: the token-level BIO formulation both notebooks relax
- [[spaCy Toy NER - Notebook]] - Lecture 11b: transition-based `ner` training vs Notebook A's `SpanCategorizer`
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L12_notebook_explained]] (AY 2026/2027)
- Luan et al. (2019). *A General Framework for Information Extraction using Dynamic Span Graphs (DyGIE++)* (EMNLP-IJCNLP) - cited via the ATA_L12_notebook_explained source above
- Zaratiana et al. (2024). *GLiNER: Generalist Model for Named Entity Recognition using Bidirectional Transformer* (NAACL) - cited via the ATA_L12_notebook_explained source above
- Zhou et al. (2024). *UniversalNER: Targeted Distillation from Large Language Models for Open Named Entity Recognition* (ICLR) - cited via the ATA_L12_notebook_explained source above
- Cocchieri et al. (2025). *ZeroNER: Fueling Zero-Shot Named Entity Recognition via Entity Type Descriptions* (Findings of ACL) - cited via the ATA_L12_notebook_explained source above
