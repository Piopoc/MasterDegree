# Lecture 12 — Detailed Explanation of the Companion Notebooks

**Course:** Advanced Text Analytics (ATA)
**Topic:** Named Entity Recognition II — span-based NER, open / zero-shot NER, and generalist models

This document provides a detailed, cell-by-cell explanation of the two Python notebooks contained in the `L12` folder:

1. **`ATA_L12_spaCy_SpanCategorizer_Toy_NER.ipynb`** — a practical, end-to-end training example in spaCy that replaces the token-level `EntityRecognizer` of Lecture 11 with a `SpanCategorizer`.
2. **`ATA_L12_NER_II_Span_Open_ZeroShot_Generalist_companion.ipynb`** — a conceptually transparent, NumPy-only walkthrough of span NER mechanics, open-label matching, and a practical (optional) GLiNER demo.

Together the two notebooks illustrate the central shift of Lecture 12: **moving the unit of prediction from individual tokens to contiguous text spans, and moving from a fixed output label inventory to entity types supplied at inference time.**

---

## 1. Context: What Changes from Lecture 11

In Lecture 11, NER was formulated as token-level sequence labeling:

```text
text
 ↓
EntityRecognizer (Transformer encoder)
 ↓
BIO label per token
 ↓
doc.ents
```

This formulation rests on two strict assumptions:

1. **Token-level BIO decoding** — every token receives exactly one tag (B-*, I-*, O), and entity boundaries are inferred from tag transitions.
2. **Fixed output inventory** — the set of entity types is a fixed output layer decided at training time; adding a new type requires retraining.

Lecture 12 relaxes both assumptions in two steps:

```text
Step 1 (span NER):           candidate span → span representation → entity type / NONE

Step 2 (open NER):           candidate span + requested entity type → compatibility score
```

- **Step 1** changes the *prediction unit*: instead of tagging tokens, the model classifies whole candidate intervals. This makes overlapping and nested entities representable.
- **Step 2** changes the *representation of the label*: instead of a fixed output dimension, the entity type becomes an input (a text embedding) that is compared against span embeddings. This enables open / zero-shot / generalist extraction.

The two notebooks each cover one side of this story:

| Notebook | Focus |
|---|---|
| `ATA_L12_spaCy_SpanCategorizer_Toy_NER.ipynb` | Step 1 in practice: a real, trainable spaCy `SpanCategorizer` pipeline (still closed-set) |
| `ATA_L12_NER_II_Span_Open_ZeroShot_Generalist_companion.ipynb` | Steps 1 and 2 in theory: transparent NumPy toy implementations, plus optional GLiNER demo |

---

# 2. Notebook A — spaCy SpanCategorizer Toy NER

**File:** `ATA_L12_spaCy_SpanCategorizer_Toy_NER.ipynb`
**Requirements:** `spacy`, `pandas`, `matplotlib` (no pretrained Transformer; the model trains from scratch).

## 2.1 Title cell — the architectural contrast

The notebook opens by drawing the pipeline contrast explicitly:

**Lecture 11 pipeline:**

```text
text
 ↓
EntityRecognizer
 ↓
doc.ents
```

**Lecture 12 pipeline (this notebook):**

```text
text
 ↓
candidate spans
 ↓
SpanCategorizer
 ↓
doc.spans["sc"]
```

The key point stated in the notebook: **`doc.spans["sc"]` can represent overlapping and nested spans**, whereas `doc.ents` in spaCy requires non-overlapping, flat entities.

The toy task uses only two labels:

- `ORGANIZATION`
- `LOCATION`

## 2.2 Section 0 — Setup

```python
import random
import spacy
import pandas as pd
import matplotlib.pyplot as plt
from spacy.training import Example

SEED = 42
random.seed(SEED)
```

Sets the random seed for reproducibility and imports spaCy's `Example` class, the standard container that pairs a *predicted* `Doc` with a *reference* (gold) `Doc` for training.

## 2.3 Section 1 — Create a blank English pipeline

```python
nlp = spacy.blank("en")
```

A blank pipeline contains only **tokenization rules** — no statistical components (no tagger, no NER). Everything else will be added and trained from scratch on the tiny dataset below.

## 2.4 Section 2 — Add `spancat_singlelabel`

```python
spancat = nlp.add_pipe("spancat_singlelabel")
spancat.add_label("ORGANIZATION")
spancat.add_label("LOCATION")
```

**Why `spancat_singlelabel`?** As the notebook explains, this component is chosen because *each individual span receives at most one class* in this toy task. (spaCy also offers a multi-label `spancat` variant for cases where the same span can carry several labels simultaneously.)

### Why `doc.spans["sc"]` instead of `doc.ents`?

The notebook motivates this with the canonical nested example:

```text
University of Padua
              Padua

ORGANIZATION
              LOCATION
```

The span `University of Padua` (ORGANIZATION) and the span `Padua` (LOCATION) **overlap**. `doc.ents` cannot store both; `doc.spans["sc"]` (the SpanGroup produced by the spancat pipeline, keyed `"sc"` by default) can.

## 2.5 Section 3 — Define a tiny training set

Three examples, each with character-offset annotations:

| Text | Annotations (start, end, label) |
|---|---|
| "The University of Padua is located in Padua." | (4, 23, ORGANIZATION), (18, 23, LOCATION), (38, 43, LOCATION) |
| "Researchers at the University of Oxford work in Oxford." | (19, 39, ORGANIZATION), (33, 39, LOCATION), (48, 54, LOCATION) |
| "The University of Cambridge is based in Cambridge." | (4, 27, ORGANIZATION), (21, 30, LOCATION), (40, 49, LOCATION) |

Note that in each sentence the ORGANIZATION span *contains* a LOCATION span ("University of **Padua**"), so the gold annotations themselves exercise the overlapping-span capability.

## 2.6 Section 4 — Check character offsets

```python
for text, annotations in TRAIN_DATA:
    for start, end, label in annotations:
        print(f"{text[start:end]!r:25s} -> {label:15s} ({start}, {end})")
```

A diagnostic step: slice the original string with the stored offsets to visually confirm that the gold spans point at the intended substrings before any training begins.

## 2.7 Section 5 — Convert annotations into spaCy `Example` objects

This is the core data-preparation step:

```python
reference = nlp.make_doc(text)          # empty Doc, tokenized only
span = reference.char_span(start, end, label=label, alignment_mode="contract")
reference.spans["sc"] = gold_spans      # gold spans stored in the "sc" key
predicted = nlp.make_doc(text)
examples.append(Example(predicted, reference))
```

Key points:

- Gold spans are stored in **`reference.spans["sc"]`** — the same key the `spancat_singlelabel` component will read during training.
- `alignment_mode="contract"` shrinks the character range to fit token boundaries; if the offsets cannot align at all, `char_span` returns `None` and the notebook raises an explicit `ValueError`.
- An `Example` holds both a predicted and a reference doc; during `nlp.update` spaCy computes the loss by comparing the component's output against the reference.

## 2.8 Sections 6–7 — Inspect one gold example and token indices

These cells print, for the first example:

- each gold span as `span.text`, `span.label_`, and its **token** indices `(span.start, span.end)` — note these differ from the character offsets used in `TRAIN_DATA`;
- a full token table with `token.i`, `token.text`, and character ranges `(token.idx, token.idx + len(token.text))`.

This makes the char-offset ↔ token-index conversion explicit, which is a common source of bugs in NER annotation.

## 2.9 Section 8 — Initialize the model

```python
optimizer = nlp.initialize(get_examples=lambda: examples)
```

spaCy initializes the statistical weights using the provided examples (needed so the spancat component can infer the feature schema and number of labels). The notebook stresses: **this model is trained from scratch; no pretrained Transformer is involved.** The default `spancat_singlelabel` uses a tok2vec-based encoder, not BERT.

## 2.10 Section 9 — Train the SpanCategorizer

The training loop is standard spaCy supervised training:

```python
N_EPOCHS = 150
for epoch in range(N_EPOCHS):
    random.shuffle(examples)
    losses = {}
    nlp.update(examples, sgd=optimizer, drop=0.2, losses=losses)
    loss = losses.get("spancat_singlelabel", 0.0)
    loss_history.append(loss)
```

Conceptually (as drawn in the notebook):

```text
input text
   ↓
candidate spans
   ↓
span representations
   ↓
span classifier
   ↓
loss
   ↓
parameter updates
```

- `drop=0.2` applies dropout during training.
- The loss recorded is specifically the `spancat_singlelabel` loss component.
- 150 epochs over 3 examples is a deliberately over-parameterized toy run — it converges but does not generalize.

## 2.11 Section 10 — Plot training loss

A simple matplotlib plot of loss vs. epoch, with the important caveat stated in the notebook:

> *A decreasing training loss shows fitting to this tiny dataset, not robust generalization.*

## 2.12 Sections 11–12 — Inference and score inspection

```python
text = "The University of Padua collaborates with researchers in Padua."
doc = nlp(text)
for span in doc.spans["sc"]:
    print(repr(span.text), "->", span.label_)
```

Predictions come from **`doc.spans["sc"]`**, not `doc.ents`.

Section 12 additionally reads the per-span confidence scores stored in the SpanGroup attributes:

```python
spans = doc.spans["sc"]
scores = spans.attrs.get("scores")
```

and prints them alongside each predicted span — useful for thresholding decisions later.

## 2.13 Section 13 — Test unseen sentences

Four new sentences (different universities/cities than the three training ones) are run through the pipeline to qualitatively check generalization. Because the model has only 3 training examples, results are indicative rather than reliable.

## 2.14 Section 14 — Comparison table: Lecture 11 `ner` vs Lecture 12 `spancat`

| Lecture 11 | Lecture 12 |
|---|---|
| `ner` | `spancat_singlelabel` |
| transition-based entity recognizer | span categorizer |
| `doc.ents` | `doc.spans["sc"]` |
| flat named entities | overlapping spans allowed |
| entity recognition | more general span classification |

This table is the notebook's one-line summary of the architectural change.

## 2.15 Section 15 — Candidate-span combinatorics

```python
max_width = 4
for start in range(len(sample_doc)):
    for end in range(start + 1, min(len(sample_doc), start + max_width) + 1):
        candidates.append((start, end, sample_doc[start:end].text))
```

Enumerates every contiguous span up to width 4 for the 9-token sentence and displays them in a DataFrame. The notebook concludes that this combinatorial explosion is why span systems use:

- a **maximum span width**;
- **candidate pruning**;
- **negative-example sampling**;
- **learned span proposal** (e.g., a separate SpanFinder).

## 2.16 Section 16 — Optional extension: `SpanFinder` + `SpanCategorizer`

A two-stage pipeline is sketched (not implemented):

```text
text
 ↓
SpanFinder        → "where might an interesting span be?"
 ↓
candidate spans
 ↓
SpanCategorizer   → "what label should that span receive?"
 ↓
labeled spans
```

This decouples *span proposal* from *span classification*, addressing the combinatorics problem of Section 15.

## 2.17 Section 17 — Error-analysis exercise

Three sentences are classified, and the reader is asked to categorize each error as one of:

- **missed span** (false negative);
- **spurious span** (false positive);
- **boundary error** (right type, wrong extent);
- **type error** (right extent, wrong label);
- **nested-span error** (failure specific to overlapping ground truth).

This taxonomy is the standard diagnostic framework for span-based extraction.

## 2.18 Section 18 — Relation to GLiNER

The notebook closes by placing itself in the L12 arc:

- This spaCy example is still **closed-set supervised NER** — `ORGANIZATION` and `LOCATION` are defined *before* training.
- **GLiNER changes the task**: `text + requested labels → generalist model → span/label compatibility`. Labels become inputs at inference time.

```text
spaCy SpanCategorizer  →  span-based NER          (this notebook)
GLiNER                 →  open / label-conditioned generalist NER  (other notebook)
```

## 2.19 Take-home messages (Notebook A)

1. `SpanCategorizer` treats **spans as the prediction unit**.
2. `doc.spans[...]` can represent **overlapping** spans.
3. Nested entities are therefore natural in a span formulation.
4. **Candidate generation** is a central computational issue.
5. This spaCy example is still **supervised and closed-set**.
6. It bridges L11 (supervised token NER) and L12 (span NER).
7. GLiNER is the next step: entity labels supplied **at inference time**.

---

# 3. Notebook B — Span NER, Open / Zero-Shot NER, Generalist Models

**File:** `ATA_L12_NER_II_Span_Open_ZeroShot_Generalist_companion.ipynb`
**Requirements:** NumPy, pandas, matplotlib for the toy sections; optional `gliner` package + pretrained checkpoint for Section 10 onward.

The notebook has two parts:

1. **Transparent toy implementations** of span NER and open-label matching (fully runnable with NumPy only).
2. An **optional practical GLiNER example** (cells are written but left commented, so nothing auto-installs or auto-downloads).

## 3.1 Title cell — the formulation ladder

The notebook frames Lecture 12 as a progressive change of formulation:

```text
L11:   token contextual representation → fixed BIO classifier → one label per token

L12a:  candidate span → span representation → entity type / NONE

L12b:  candidate span + requested entity type → compatibility score
```

## 3.2 Section 1 — From BIO labels to candidate spans

Running example: **"BRCA1 mutations increase breast cancer risk"** (6 tokens).

- L11 asked: *What BIO label should each token receive?*
- Span NER asks: *Which contiguous token intervals are entities, and what type should each interval receive?*

### Section 1.1 — Generate contiguous candidate spans

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

A span is the pair `(start, end)` with **start inclusive, end exclusive** — e.g. `(3, 5)` = "breast cancer".

For `n` tokens, the total number of contiguous spans (before any width limit) is:

$$\frac{n(n+1)}{2}$$

which the notebook verifies: for 6 tokens, `6 * 7 // 2 = 21` spans.

### Exercise 1 — spans of maximum width 3

Calling `candidate_spans(tokens, max_width=3)` reduces the candidate set. With 6 tokens and max width 3 the count is `6 + 5 + 4 = 15` spans — the `max_width` constraint is the first and simplest mitigation of the quadratic growth.

## 3.3 Section 2 — Gold labels for spans

Gold entities for the example:

```text
BRCA1          → GENE
breast cancer  → DISEASE
```

```python
gold_entities = {(0, 1): "GENE", (3, 5): "DISEASE"}

def gold_span_label(span):
    return gold_entities.get(span, "NONE")
```

A DataFrame is built listing every candidate span with its gold label. The notebook draws the practical conclusion:

> *In span NER, the large majority of candidate spans are usually negative examples (`NONE`).*

Hence **candidate generation and negative-example balancing matter** — the class-imbalance problem is structural in span enumeration.

## 3.4 Section 3 — Contextual token representations

Assume (as in L09–L11) a Transformer has produced one hidden vector per token:

$$H = [h_1, h_2, \ldots, h_n]$$

For transparency the notebook uses a **synthetic 4-dimensional matrix** `H` (6 × 4), one hand-crafted row per token of the example sentence.

## 3.5 Section 4 — Build a representation for each candidate span

Since a span contains several token vectors, they must be pooled into one fixed-size vector. The toy choice:

$$g(i,j) = [h_i \,;\, h_{j-1} \,;\, \operatorname{mean}(h_i,\ldots,h_{j-1}) \,;\, \text{width}]$$

Components:

- **first-token vector** (start boundary) — 4 dims
- **last-token vector** (end boundary) — 4 dims
- **mean vector** (internal pooling) — 4 dims
- **width scalar** (number of tokens) — 1 dim

Total: $g(i,j) \in \mathbb{R}^{13}$ (4 + 4 + 4 + 1).

```python
def span_representation(H, start, end):
    first = H[start]
    last = H[end - 1]
    mean = H[start:end].mean(axis=0)
    width = np.array([end - start], dtype=float)
    return np.concatenate([first, last, mean, width])
```

The notebook stresses that **the exact representation is a modeling choice**; real systems may use start/end vectors only, attention pooling over the span, learned span-width embeddings, or additional syntactic/lexical features.

## 3.6 Section 5 — Train a toy supervised span classifier

The classifier receives one 13-dim vector per span and predicts one of three classes:

```text
NONE / GENE / DISEASE
```

Structure:

```python
label_list = ["NONE", "GENE", "DISEASE"]
X_span = np.stack([span_representation(H, s, e) for s, e in spans_max3])  # (15, 13)
y_span = np.array([label_to_id[gold_span_label(sp)] for sp in spans_max3])
```

The training loop is a hand-written multinomial logistic regression:

- `softmax` with max-shift for numerical stability;
- `cross_entropy` with a `1e-12` epsilon;
- gradients: `grad_logits = (probs - one_hot) / N`, then `grad_W = X.T @ grad_logits`, `grad_b = grad_logits.sum(axis=0)`;
- plain gradient descent, learning rate 0.2, 1000 epochs;
- loss history recorded and plotted.

The notebook then inspects predictions in a DataFrame showing, for each span: gold label, predicted label, and `P(NONE)`, `P(GENE)`, `P(DISEASE)`.

### Reflection (L11 vs L12 formulation)

$$\text{L11: } h_i \rightarrow \text{BIO label} \qquad\qquad \text{L12: } g(i,j) \rightarrow \text{entity type or NONE}$$

> *The supervised machinery is similar; the object being classified has changed.*

## 3.7 Section 6 — Why span NER can represent nested entities

A single flat BIO sequence cannot assign two labels to overlapping intervals. The example:

```text
University of Padua
    (0, 3) → ORGANIZATION
      (2, 3) → LOCATION
```

A span-based system simply scores both intervals independently:

```text
(0, 3) → ORGANIZATION
(2, 3) → LOCATION
```

Whether overlapping predictions are kept at decode time is then a **decoding/modeling choice**, not a representational impossibility.

## 3.8 Section 7 — From a fixed label inventory to open-label matching

The toy span classifier still has a fixed output matrix $W \in \mathbb{R}^{m \times K}$ with `K` known during training (`NONE`, `GENE`, `DISEASE`). Asking for a new type such as `DRUG` would normally require changing/retraining the classifier.

The open/generalist NER shift, stated as a principle:

> *Represent the entity type itself and compare it with candidate spans.*

## 3.9 Section 8 — Toy label-conditioned matching

Both candidate spans and entity-type descriptions are imagined as embedded in the **same semantic space**, and the decision becomes a compatibility score:

$$\operatorname{score}(span, type) = \operatorname{cosine}(g_{span}, e_{type})$$

Hand-designed vectors:

```python
span_vectors   = {"BRCA1": [0.95, 0.10, 0.05],
                  "breast cancer": [0.05, 0.95, 0.10],
                  "aspirin": [0.10, 0.10, 0.95]}

label_vectors  = {"gene": [1, 0, 0],
                  "disease": [0, 1, 0],
                  "drug": [0, 0, 1]}
```

A pivot table of cosine similarities shows each span matching its aligned label. The crucial observation:

> *The label `drug` did **not** have to be a fixed output dimension of a classifier. It is represented as an **input** vector.*

This is the central conceptual shift behind open / generalist NER.

### Section 8.1 — Description-driven labels

Short labels are ambiguous (`bank` = financial institution vs. river bank). Richer natural-language descriptions give a better semantic specification of the target concept — the motivation behind **description-driven zero-shot NER** (as in ZeroNER). The notebook prints two example descriptions for `financial_bank` and `river_bank`.

## 3.10 Section 9 — What does zero-shot mean here?

A strict (hard) zero-shot interpretation:

```text
target entity type
        ↓
no labeled training examples for that type
        ↓
model receives only the type name / description
        ↓
must recognize instances of the unseen type
```

Potential problems listed:

- **indirect supervision** for the supposedly unseen type;
- **semantically equivalent labels** appearing in training data;
- **leakage through benchmark construction**.

This is why recent work distinguishes stricter "hard zero-shot" settings (cf. ZeroNER, Cocchieri et al. 2025).

## 3.11 Section 10 — Generalist NER: practical example with GLiNER

Conceptual pipeline:

```text
text + requested labels
        ↓
bidirectional Transformer
        ↓
span / label compatibility
        ↓
entities
```

The section is **optional**: it requires installing `gliner` and downloading a pretrained checkpoint. All relevant cells are present but commented out.

### 10.1 Installation (commented)

```python
# %pip install --upgrade pip
# %pip install -U ipywidgets
# %pip install tiktoken
# %pip install -U sentencepiece protobuf
# %pip install gliner
```

Left commented so the notebook never installs packages automatically.

### 10.2 Load a pretrained model

```python
# from gliner import GLiNER
# gliner_model = GLiNER.from_pretrained("urchade/gliner_small-v2.1")
```

The small v2.1 checkpoint keeps the classroom example light; the first call downloads it.

### 10.3 Supply entity labels at inference time

```python
text = "BRCA1 mutations are associated with breast cancer and aspirin is sometimes studied in cancer prevention."
labels = ["gene", "disease", "drug"]

# entities = gliner_model.predict_entities(text, labels, threshold=0.5)
```

**Key difference from the L11 supervised classifier:** `labels` is provided *when we perform inference*, not baked into the output layer.

### 10.4 Change the requested ontology without retraining

```python
alternative_labels = ["gene", "medical condition", "chemical substance", "research activity"]
# entities = gliner_model.predict_entities(text, alternative_labels, threshold=0.5)
```

> *The model is not being retrained between these two calls.* — This is the point of the experiment.

### Exercise 2 — legal-domain labels

Applied to:

```text
The Supreme Court dismissed the appeal filed by Smith against the Ministry of Justice.
```

with labels `court`, `legal action`, `person`, `government organization`, `legal document`, then asks the reader to change the label descriptions and observe the effect — demonstrating both domain flexibility and description sensitivity.

## 3.12 Section 11 — Thresholds matter

Generalist span recognizers return a score; a threshold decides how much evidence is required:

```text
lower threshold → more entities → higher recall → more false positives
higher threshold → fewer entities → higher precision → more missed entities
```

A sweep over `thresholds = [0.30, 0.50, 0.70, 0.90]` prints, for each value, the entities retained with their scores, and a final DataFrame adds a boolean column `kept @ 0.xx` per threshold for every candidate.

> *This is an evaluation choice, not merely a cosmetic parameter.*

*(Note: the final DataFrame cell references a `scores` variable that is not defined in the notebook as written — it would need to be assembled from the `results` list collected in the threshold-sweep cell. Worth keeping in mind if you run the notebook.)*

## 3.13 Section 12 — Generalist NER versus a generative LLM

| | Generalist NER | Generative LLM |
|---|---|---|
| Input | text + labels | natural-language instruction + text |
| Mechanism | specialized encoder → span scores | autoregressive generation |
| Output | structured entities | generated structured answer |

Trade-off: a general LLM is more flexible, but a specialized NER model can be **smaller, faster, and easier to control** for extraction-only tasks.

## 3.14 Section 13 — Where UniversalNER fits

A different route to open NER via distillation:

```text
large teacher LLM
        ↓
generate broad NER supervision
        ↓
targeted instruction-tuning / distillation
        ↓
smaller model specialized for open NER
```

Conceptual question:

> *Can we retain broad entity-type generalization while distilling away capabilities that are irrelevant to NER?*

(UniversalNER: evaluated across 43 datasets / 9 domains — biomedicine, law, finance, programming, social media, etc.)

## 3.15 Section 14 — Summary: three NER formulations

```text
L11 token classification:      h_i                        → BIO label

L12 span classification:       g(i,j)                     → GENE / DISEASE / NONE

L12 open/generalist NER:       g(i,j) + e(requested type) → compatibility
```

The Transformer supports all three; what changes is:

- the **prediction unit** (token → span);
- the **representation of the output label** (fixed dimension → input embedding);
- the **supervision and generalization assumptions** (closed-set → open/zero-shot).

## 3.16 Section 15 — Take-home messages (Notebook B)

1. Span NER predicts entity types for candidate **token intervals**, not BIO labels per position.
2. Candidate generation matters: the number of spans grows **quadratically** with sequence length.
3. Span representations come from Transformer hidden states via boundaries, pooling, width, or learned span encoders.
4. A conventional span classifier is still **closed-set** if its output dimensions are a fixed label inventory.
5. Open/generalist NER makes the entity type **part of the input / matching process**.
6. Zero-shot claims require careful evaluation (no direct or indirect target-specific supervision).
7. **GLiNER** is a practical example of label-conditioned generalist NER.
8. Generalist NER and generative LLM prompting solve related problems with different cost/control/modeling trade-offs.

## 3.17 References (cited in the notebook)

- Luan et al. (2019). *A General Framework for Information Extraction using Dynamic Span Graphs (DyGIE++).* EMNLP-IJCNLP.
- Zaratiana et al. (2024). *GLiNER: Generalist Model for Named Entity Recognition using Bidirectional Transformer.* NAACL.
- Zhou et al. (2024). *UniversalNER: Targeted Distillation from Large Language Models for Open Named Entity Recognition.* ICLR.
- Cocchieri et al. (2025). *ZeroNER: Fueling Zero-Shot Named Entity Recognition via Entity Type Descriptions.* Findings of ACL.

---

# 4. How the Two Notebooks Fit Together

| Dimension | Notebook A (spaCy SpanCategorizer) | Notebook B (Span/Open/Zero-shot companion) |
|---|---|---|
| Nature | Runnable end-to-end training | Conceptual toy math + optional GLiNER demo |
| Dependencies | spaCy | NumPy/pandas/matplotlib (+ optional `gliner`) |
| Formulation covered | L12a: span classification (closed-set) | L12a (theory) **and** L12b (open-label matching) |
| Prediction unit | span (`doc.spans["sc"]`) | span (`g(i,j)`) |
| Labels | fixed before training (`ORGANIZATION`, `LOCATION`) | fixed in the toy part; supplied at inference in the GLiNER part |
| Nested/overlapping entities | trained and predicted directly | explained conceptually |
| Key mechanics shown | real pipeline, Example objects, training loop, scores | span enumeration math, span vector construction, hand-written logistic regression, cosine label matching, threshold sweep |

**Recommended reading order:** Notebook B first for the theory (sections 1–9), then Notebook A to see the same ideas realized in a production-grade framework, then Notebook B section 10 onward for the open/zero-shot generalist step.

---

# 5. Consolidated Take-Home Points for Lecture 12

1. **Span NER changes the prediction unit** from tokens to contiguous intervals — enabling overlapping and nested entities by construction.
2. **Span vectors** are pooled from token hidden states (boundaries + mean + width in the toy; richer encoders in real systems).
3. **Candidate enumeration is quadratic** (`n(n+1)/2`), so `max_width`, pruning, negative sampling, and learned span proposals are essential engineering components.
4. **Class imbalance is structural**: the vast majority of candidates are `NONE`.
5. **A span classifier alone is still closed-set** — its output layer fixes the ontology at training time.
6. **Open/generalist NER inverts the role of the label**: the entity type becomes an input embedding compared to span embeddings via a compatibility score (GLiNER's mechanism).
7. **Description-driven labels** resolve label ambiguity and underpin hard zero-shot evaluation protocols (ZeroNER).
8. **UniversalNER** reaches open NER by distilling teacher-LLM supervision into a smaller specialized student.
9. **Thresholds encode the precision/recall trade-off** and must be treated as an evaluation decision.
10. **Generalist NER ≠ generative LLM**: specialized encoders are smaller, faster, and more controllable for pure extraction; LLMs are more flexible but heavier.
