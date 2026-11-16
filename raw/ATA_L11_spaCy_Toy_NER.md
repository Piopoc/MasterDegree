# spaCy Toy NER Fine-Tuning — Theory Companion

**Source notebook:** `ATA_L11_spaCy_Toy_NER_Finetuning.ipynb`  
**Course:** Advanced Text Analytics — Lecture 11  
**Writing guide:** ASD-STE100 Simplified Technical English (target: about 80%)  
**Document type:** Theory notes. This document does not replace the notebook.

---

## Summary

The notebook shows the spaCy training workflow. A blank pipeline gets a statistical NER component. Annotated spans become `Example` objects. `nlp.initialize` builds the model. `nlp.update` trains it. The spaCy NER component is transition-based and uses BILUO actions. The notebook uses five sentences to show the workflow, not to get a high score.

---

## 0. About this document

### 0.1 Purpose

This document explains the theory in the source notebook. It answers three questions for each part:

1. What is the problem?
2. Why does the notebook use this method?
3. What does the method do?

The notebook shows a small spaCy training workflow. This document explains the ideas behind the workflow. It does not copy the code.

### 0.2 Language note

ASD-STE100 is an English standard for technical text. This document uses the standard as a guide. The target is about 80% compliance. We keep these items unchanged:

- code names, for example `nlp.update`;
- standard NLP terms, for example "tokenizer";
- spaCy component names, for example `EntityRecognizer`.

We define these terms in Section 0.3. We use short sentences, active voice, and one meaning for each word. We avoid idioms and metaphors. We use "must" for a requirement and "can" for a possibility.

### 0.3 Terms

#### Text and annotation

| Term | Meaning in this document |
|---|---|
| Pipeline | A sequence of components that process a text. |
| Component | One step in a pipeline, for example a tokenizer or an NER model. |
| Tokenizer | A component that splits text into tokens. |
| Token | A small unit of text. A token can be a word or a symbol. |
| Entity | A text span with a type, for example a gene name. |
| Span | A sequence of tokens. We identify a span by a start position and an end position. |
| Boundary | The start and the end of a span. |
| Annotation | A label that a human gives to a text. |
| Gold annotation | The correct annotation. Also "reference annotation". |
| Character offset | A pair of numbers that gives the start and the end of a text span. |
| Label | A class name, for example `GENE` or `DISEASE`. |
| Prediction | The entity span that the model gives. |

#### spaCy objects

| Term | Meaning in this document |
|---|---|
| `nlp` | The spaCy language object. It holds the pipeline and the vocabulary. |
| `Doc` | A spaCy object that holds a tokenized text and its annotations. |
| `Example` | A training object. It pairs a model input with a gold reference. |
| `reference` | The gold `Doc` inside an `Example`. |
| `EntityRuler` | A rule-based component. It uses patterns and lists of names. |
| `EntityRecognizer` | The statistical NER component. Its pipeline name is `ner`. |
| `tok2vec` | The part of the model that turns tokens into vectors. |
| Transition | One action in a transition-based parser. |
| BILUO | A label scheme with `B`, `I`, `L`, `U`, and `O`. |

#### Training

| Term | Meaning in this document |
|---|---|
| Parameter | A value that the model learns. |
| Weight | A parameter in a matrix. |
| Loss | A number that measures the error of the model. |
| Gradient | A vector that shows the direction to change a parameter. |
| Backpropagation | The method that computes gradients through a network. |
| Transformer | A neural network architecture that uses self-attention. |
| CRF | Conditional random field. A layer that models dependencies between labels. |
| Optimizer | The object that applies the parameter updates. |
| Learning rate | A small number that controls the size of a parameter update. |
| Epoch | One full pass over the training data. |
| Batch | A small group of examples. |
| Minibatch | A small batch used for one update. |
| Dropout | A training method that randomly sets some values to zero. |
| Inference | The use of a trained model to make predictions. |
| Generalization | The ability to work on new data. |
| Overfitting | A model fits the training data but does not generalize. |
| Memorization | The model stores the training examples but does not learn a general rule. |

#### Evaluation

| Term | Meaning in this document |
|---|---|
| True positive (TP) | A correct predicted entity. |
| False positive (FP) | An incorrect extra predicted entity. |
| False negative (FN) | A gold entity that the model does not find. |
| Precision | The fraction of predicted entities that are correct. |
| Recall | The fraction of gold entities that the model finds. |
| F1 | The harmonic mean of precision and recall. |
| Missed entity | A gold entity that the model does not predict. |
| Extra entity | A predicted entity that is not in the gold annotation. |
| Boundary error | The type is correct, but the start or the end is wrong. |
| Type error | The span is correct, but the type is wrong. |

---

## 1. Purpose and position in the course

### 1.1 What the notebook does

The notebook trains a small statistical NER model with spaCy. The workflow is:

```text
annotated text
   ↓
spaCy Example objects
   ↓
trainable NER component
   ↓
predicted entity spans
   ↓
training loss
   ↓
parameter updates
```

The dataset is intentionally small. The goal is to understand the spaCy workflow, not to build a good biomedical NER system.

### 1.2 Position in the course

The course moves through three levels:

```text
EntityRuler
   ↓
rule-based NER

blank spaCy + trainable ner
   ↓
supervised NER from scratch

pretrained spaCy / Transformer pipeline
   ↓
fine-tuned NER
```

This notebook is the second level. It sits between the deterministic rules of `EntityRuler` and the pretrained models of a real pipeline. This position is important. The notebook shows the learning mechanics without the complexity of a pretrained Transformer.

### 1.3 What "fine-tuning" means here

The notebook title says "fine-tuning". But the model starts from a blank pipeline, not from a pretrained model. Therefore, the notebook trains from scratch. It uses the same *workflow* as fine-tuning, but not the same *starting point*.

This distinction is important. In a real fine-tuning setup, we start from pretrained weights. In this notebook, we start from random weights. The training loop is the same in principle. The result is different.

---

## 2. The spaCy pipeline model

### 2.1 The `nlp` object

The `nlp` object is the main spaCy object. It has three important parts:

1. the vocabulary;
2. the tokenizer;
3. the pipeline components.

When we call `nlp(text)`, spaCy does these steps:

1. The tokenizer splits the text into tokens.
2. Each pipeline component reads the `Doc` and adds information.
3. The final `Doc` has tokens, entities, and other annotations.

### 2.2 The blank pipeline

`spacy.blank("en")` makes an English pipeline with no trained components. The pipeline has a tokenizer, but no statistical model. `nlp.pipe_names` is empty.

This is useful for teaching. We can add one component and see its effect. There is no hidden pretrained model.

### 2.3 Pipeline components

A component is one step in the pipeline. Examples:

- `tok2vec`: turns tokens into vectors;
- `ner`: finds entity spans;
- `entity_ruler`: finds entities from patterns;
- `tagger`: gives part-of-speech labels;
- `parser`: gives syntactic dependencies.

The order of the components matters. A component can use the output of an earlier component.

### 2.4 `EntityRuler` vs `ner`

These two components both find entities. But they work in different ways.

| Item | `EntityRuler` | `ner` (`EntityRecognizer`) |
|---|---|---|
| Method | Rules and patterns. | Statistical model. |
| Training | No training. | Training on annotated examples. |
| Input | Lists of names, patterns. | Gold entity spans. |
| Output | Deterministic spans. | Predicted spans. |
| New names | Add a pattern. | Add training examples or retrain. |
| Errors | The ruler can make errors only if a rule is wrong or missing. | Can generalize, but can make statistical errors. |

The two components are complementary. A common pipeline uses `EntityRuler` for known names and `ner` for learned patterns. The notebook uses only `ner`, because the goal is to show the statistical workflow.

---

## 3. Annotation with character offsets

### 3.1 The format

spaCy receives gold entities as character offsets:

```text
(start_char, end_char, label)
```

Example:

```text
"BRCA1 mutations increase breast cancer risk"
```

| Entity | Start | End | Label |
|---|---|---|---|
| BRCA1 | 0 | 5 | GENE |
| breast cancer | 25 | 38 | DISEASE |

### 3.2 End is exclusive

The end position is exclusive. The span `(25, 38)` includes characters 25 to 37. It does not include character 38. This is the same convention as Python slices.

The length of the span is `end - start`. For `(25, 38)`, the length is 13. The text `breast cancer` has 13 characters.

### 3.3 Alignment with token boundaries

The offsets must align with token boundaries. If a span starts or ends inside a token, spaCy cannot give the label to a whole token. The annotation is then invalid for the model.

Example: if the tokenizer makes `breast` one token and `cancer` one token, the span `(25, 38)` aligns with the two tokens. If the tokenizer makes `breastcancer` one token, the span does not align.

The notebook checks the offsets before training. This is a good habit. A wrong offset gives a wrong gold label, and the model learns the wrong pattern.

### 3.4 Why character offsets

Character offsets have two advantages:

1. They do not depend on the tokenizer.
2. They keep the original text.

The tokenizer can change. The character positions stay the same. This makes the annotation reusable.

The disadvantage is that the offsets must be correct. A small error can change the span. A tool that shows the text and the span is useful.

---

## 4. `Example` objects

### 4.1 Two documents in one object

An `Example` contains two documents:

1. the **predicted** document: the input that the model sees;
2. the **reference** document: the gold annotation.

The predicted document comes from `nlp.make_doc(text)`. This step tokenizes the text but does not run the pipeline. The reference document comes from the annotations.

Why two documents? Because the model can make different predictions. The `Example` object keeps the input and the gold annotation separate. During training, spaCy compares the prediction with the reference. In this notebook, both documents use the same tokenizer.

### 4.2 `Example.from_dict`

`Example.from_dict(doc, annotations)` builds an `Example` from:

- a `Doc` (the input);
- a dictionary with the gold annotations.

The dictionary has this form:

```python
{"entities": [(start, end, label), ...]}
```

The function builds the reference `Doc` and aligns the entities to the tokens.

### 4.3 Why this matters

The `Example` object is the bridge between the annotation format and the training loop. It gives the model a clean input and a clean gold reference. It also lets spaCy check the alignment.

If the annotation is invalid, the error appears here, before training. This is better than a silent wrong label.

---

## 5. Initialization

### 5.1 `nlp.initialize`

`nlp.initialize(...)` prepares the pipeline for training. It does these steps:

1. It reads the labels from the components.
2. It builds the model parameters.
3. It infers the input and output shapes from the examples.
4. It creates the optimizer.

The notebook calls:

```python
optimizer = nlp.initialize(lambda: examples)
```

The `lambda` gives the examples to the initialization step. spaCy uses them to find the number of labels and the shape of the input.

### 5.2 From scratch

The notebook uses a blank pipeline. Therefore, the parameters start from random values. The model is not pretrained. The `tok2vec` layer is a small neural network trained from scratch.

This is different from a real spaCy pipeline, which starts from pretrained weights. The notebook says this explicitly.

### 5.3 The optimizer

`nlp.initialize` returns an optimizer. The optimizer holds the update rule and the optimization state. The model holds the parameter values. `nlp.update` uses the optimizer to change the parameters.

The optimizer is not the model. The model is inside the pipeline. The optimizer is the object that applies the updates.

---

## 6. The training loop

### 6.1 Epochs

An epoch is one full pass over the training data. The notebook uses 50 epochs. With five examples, one epoch is very short. 50 epochs are necessary to see a clear loss decrease.

### 6.2 Shuffle

At the start of each epoch, the notebook shuffles the examples:

```python
random.shuffle(examples)
```

Shuffling changes the order of the examples. This helps the model avoid a fixed order pattern. It also makes the batches different in each epoch.

### 6.3 Minibatches

The notebook uses `minibatch(examples, size=2)`. A minibatch is a small group of examples. The model computes the loss for the group and updates the parameters once.

With five examples and a batch size of 2, one epoch has three batches:

- batch 1: examples 1 and 2;
- batch 2: examples 3 and 4;
- batch 3: example 5.

The last batch is smaller. This is normal.

### 6.4 `nlp.update`

`nlp.update` does the full training step for one batch:

1. It runs the pipeline on the batch.
2. It compares the prediction with the gold reference.
3. It computes the NER loss.
4. It computes the gradients.
5. It updates the parameters with the optimizer.

The notebook calls:

```python
nlp.update(batch, sgd=optimizer, drop=0.2, losses=losses)
```

- `batch` is the list of `Example` objects.
- `sgd` is the optimizer.
- `drop` is the dropout rate.
- `losses` is a dictionary that receives the loss values.

### 6.5 Dropout

Dropout randomly sets some values to zero during training. It is a regularization method. It makes the model less dependent on one specific value.

The notebook uses `drop=0.2`. This means that 20% of the values are dropped during the forward pass. At inference, dropout is off.

With five examples, dropout can slow the fitting. The notebook still uses it, because the goal is to show the standard workflow.

### 6.6 Losses

`nlp.update` writes the loss into the `losses` dictionary. The notebook reads:

```python
ner_loss = losses.get("ner", 0.0)
```

The key `"ner"` gives the loss of the NER component. If the key is missing, the code uses 0.0. This is a safe default.

The loss is a sum over the batches in the epoch. It is not normalized by the number of examples. Because the dataset is fixed, the curve is still useful for a rough check.

### 6.7 The loss curve

A decreasing loss shows that the model fits the training examples. It does not show that the model generalizes. With five sentences, the model can memorize the data. The notebook says this.

The curve is a diagnostic tool, not a performance metric. For performance, we need a test set and entity-level F1.

---

## 7. Transition-based NER

### 7.1 Why spaCy does not use a simple token classifier

The PyTorch notebook uses a token classifier:

```text
contextual token vector
        ↓
linear classifier
        ↓
BIO label
```

Each token gets a label independently. The model can produce invalid BIO sequences. A decoder must repair them.

spaCy's standard `EntityRecognizer` uses a different architecture: a **transition-based parser**. It predicts a sequence of actions. Each action builds a part of an entity. The action sequence is always valid by construction.

### 7.2 The BILUO actions

spaCy uses the BILUO scheme:

| Symbol | Meaning |
|---|---|
| `U` | Unit: a single-token entity. |
| `B` | Begin: the first token of a multi-token entity. |
| `I` | Inside: a middle token of a multi-token entity. |
| `L` | Last: the last token of a multi-token entity. |
| `O` | Outside: the token is not in an entity. |

Example:

```text
BRCA1      U-GENE
mutations  O
increase   O
breast     B-DISEASE
cancer     L-DISEASE
risk       O
```

A two-token entity uses `B` then `L`. A three-token entity uses `B`, `I`, then `L`. A single-token entity uses `U`.

### 7.3 The parser state

The parser reads the tokens from left to right. It keeps a state with:

- the current token position;
- the entities that it has built.

At each step, the model scores the valid actions. The parser selects the action with the highest score. This is a **greedy** decision. The parser does not search all possible action sequences.

### 7.4 Valid actions

The transition system allows only actions that keep the sequence valid:

- `O` marks the current token as outside an entity.
- `U-TYPE` starts a single-token entity.
- `B-TYPE` starts a multi-token entity.
- `I-TYPE` needs an open entity of type TYPE.
- `L-TYPE` needs an open entity of type TYPE.

This constraint prevents invalid sequences. For example, the parser cannot produce `I-GENE` without an open GENE entity. An open entity starts with `B-GENE`.

### 7.5 The model

The `tok2vec` layer turns each token into a vector. The parser uses these vectors and features from the current token and the last entity to score the actions.

The model is small in this notebook. It is trained from scratch. It is not a pretrained Transformer.

### 7.6 The training signal

During training, we know the gold entity spans. We can derive the gold action sequence. The parser compares its scores with the gold actions. It computes a loss. Then it updates the weights.

The exact loss function is an internal spaCy detail. The notebook does not show it. The important point is the same as in the PyTorch notebook:

```text
prediction
   ↓
compare with gold annotation
   ↓
loss
   ↓
gradient update
```

### 7.7 Greedy decoding and error propagation

The parser makes one decision at each step. It does not go back. If it makes a wrong decision, the next decisions can be wrong too. This is **error propagation**.

Example: if the parser starts an entity at the wrong token, the entity boundary is wrong. The next action can continue the wrong entity.

This is a known limit of greedy transition-based systems. A beam search can reduce the problem, but it costs more time. The notebook uses the default greedy behavior.

### 7.8 Comparison with BIO token classification

| Item | BIO token classifier | Transition-based NER |
|---|---|---|
| Output | One label for each token. | One action for each step. |
| Validity | Can produce invalid sequences. | Always produces valid sequences. |
| Decision | Independent for each token. | Depends on the state and the history. |
| Decoding | Needs a decoder. | The action sequence is the decode. |
| Error type | Local label errors. | Local action errors can propagate. |
| Example | PyTorch notebook. | spaCy `EntityRecognizer`. |

Both methods learn from gold annotations. Both use a loss and gradients. The difference is the output structure.

---

## 8. Fitting vs generalization

### 8.1 Training loss

The notebook trains for 50 epochs on five sentences. The NER loss decreases. This shows that the model fits the training data.

Fitting is necessary, but it is not sufficient. A model can fit the training data and still fail on new data. This is overfitting.

### 8.2 Unseen sentences

The notebook tests the model on new sentences:

```text
TP53 is associated with breast cancer
EGFR mutations occur in lung cancer
BRCA2 mutations increase cancer risk
Alzheimer disease affects the brain
```

The results can be inconsistent. The model can find some entities and miss others. This is expected. The training set is too small to learn a general rule.

### 8.3 Memorization

With five examples, the model can memorize the exact sentences. Memorization is not generalization. The model can get a low training loss and still make errors on a new sentence with the same words in a different order.

The notebook says this explicitly. Do not read the loss curve as a quality score.

### 8.4 What a real evaluation needs

A real evaluation needs:

1. a train set;
2. a dev set for tuning;
3. a test set for the final score;
4. entity-level precision, recall, and F1;
5. a fixed evaluation rule;
6. an error analysis.

The notebook has none of these. It is a teaching example, not a benchmark.

---

## 9. Error taxonomy

### 9.1 Four error types

The notebook asks you to classify prediction errors:

| Error type | Definition | Example |
|---|---|---|
| Missed entity | A gold entity is not predicted. | Gold has `BRCA1` as GENE; the model finds nothing. |
| Extra entity | A predicted entity is not in gold. | The model finds `many` as DISEASE. |
| Boundary error | The type is correct, but the start or the end is wrong. | Gold is `breast cancer`; prediction is `cancer`. |
| Type error | The span is correct, but the type is wrong. | Gold is `BRCA1` as GENE; prediction is `BRCA1` as DISEASE. |

### 9.2 Relation to TP, FP, FN

Under strict entity-level evaluation:

- a missed entity gives one FN;
- an extra entity gives one FP;
- a boundary error gives one FP and one FN;
- a type error gives one FP and one FN.

This is why strict evaluation is useful. It gives a clear penalty for each error type.

### 9.3 How to inspect errors

A good error analysis has three steps:

1. Compare the gold spans and the predicted spans.
2. Put each difference into one of the four types.
3. Count the types and look for a pattern.

Example questions:

- Does the model miss long entities?
- Does it stop at the first token?
- Does it confuse GENE and DISEASE?
- Does it find entities in new contexts?

The answers guide the next step: more data, better annotation, a different model, or a rule.

---

## 10. Comparison with the PyTorch notebook

### 10.1 The comparison table

| PyTorch toy | spaCy toy |
|---|---|
| Explicit tensors. | Framework abstractions. |
| Explicit hidden states. | The representation model is internal. |
| Explicit BIO classifier. | Transition-based NER component. |
| Explicit loss computation. | spaCy computes the NER loss. |
| `loss.backward()`. | Handled inside `nlp.update()`. |
| `optimizer.step()`. | Handled through `sgd=optimizer`. |

### 10.2 The common principle

Both notebooks use the same learning principle:

```text
prediction
   ↓
compare with gold annotation
   ↓
loss
   ↓
gradient update
```

The difference is the level of control. The PyTorch notebook shows every step. The spaCy notebook hides the low-level steps. This is the main trade-off.

### 10.3 The architectural difference

The PyTorch model is a token classifier. It gives one label to each token. The spaCy model is a transition-based parser. It gives one action at each step and builds the entities.

This difference has three effects:

1. **Validity**: the spaCy parser cannot produce an invalid BILUO sequence. The PyTorch model can produce an invalid BIO sequence.
2. **History**: the spaCy parser uses the state, which includes the entities built so far. The PyTorch classifier does not use the previous labels, unless the encoder or a CRF layer adds this information.
3. **Error propagation**: a wrong action in spaCy can cause more errors. A wrong token label in PyTorch stays local, but the decoder can still create a wrong span.

Both designs are valid. The choice depends on the task, the data, and the need for control.

---

## 11. What the notebook does not do

The notebook intentionally omits:

- a train/dev/test split;
- a pretrained spaCy model;
- a pretrained Transformer;
- subword tokenization;
- systematic evaluation;
- hyperparameter tuning.

The purpose is architectural and procedural transparency. The notebook shows the workflow, not the best result.

### 11.1 Why these omissions are acceptable here

The notebook has five sentences. A train/dev/test split would leave too few examples for each part. A pretrained model would hide the training mechanics. Subword tokenization would add a second alignment problem. Systematic evaluation would need more data to be meaningful.

The omissions are a teaching choice. They are not a recommendation for a real project.

---

## 12. Natural next step

### 12.1 The progression

```text
EntityRuler
    ↓
rule-based NER

blank spaCy + trainable ner
    ↓
supervised NER from scratch

pretrained spaCy / Transformer pipeline
    ↓
fine-tuned NER
```

### 12.2 What to add at each step

**Step 1: `EntityRuler`**

- Add a list of gene names and disease names.
- Use patterns for common forms.
- Measure precision and recall on a small test set.

**Step 2: blank spaCy + `ner`**

- Add more annotated examples.
- Add a dev set.
- Measure entity-level F1.
- Analyze the four error types.

**Step 3: pretrained pipeline**

- Start from a pretrained spaCy model or a Transformer.
- Fine-tune the NER component.
- Compare the result with the from-scratch model.
- Check the cost and the speed.

### 12.3 The link to the previous lecture

The third step uses the fine-tuning ideas from the previous lecture:

- a pretrained encoder;
- a task-specific head;
- a supervised loss;
- gradient updates;
- a choice between full fine-tuning and head-only training.

The spaCy workflow is the same. The model is different.

---

## 13. Key points

1. **spaCy can train a statistical NER model from annotated entity spans.**
2. **`Example` objects pair the model input with the gold annotation.**
3. **`nlp.initialize` prepares the pipeline and returns an optimizer.**
4. **`nlp.update` does the forward pass, the loss, the backward pass, and the update.**
5. **The standard spaCy `EntityRecognizer` is transition-based, not a simple BIO classifier.**
6. **The transition system uses BILUO actions and always produces valid entity sequences.**
7. **The parser is greedy. A wrong action can propagate to later actions.**
8. **A decreasing loss on five examples shows fitting, not generalization.**
9. **Error analysis uses four types: missed, extra, boundary, and type.**
10. **The notebook connects rule-based NER and pretrained fine-tuning.**

---

## Appendix A: BILUO example

Text: `BRCA1 mutations increase breast cancer risk`

| Token | Action | Result |
|---|---|---|
| BRCA1 | `U-GENE` | A single-token GENE entity. |
| mutations | `O` | Outside an entity. |
| increase | `O` | Outside an entity. |
| breast | `B-DISEASE` | Start a multi-token DISEASE entity. |
| cancer | `L-DISEASE` | End the DISEASE entity. |
| risk | `O` | Outside an entity. |

The final entities are:

```text
BRCA1          → GENE
breast cancer  → DISEASE
```

---

## Appendix B: Offset example

Text: `BRCA1 mutations increase breast cancer risk`

| Span | Start | End | Text | Label |
|---|---|---|---|---|
| 1 | 0 | 5 | BRCA1 | GENE |
| 2 | 25 | 38 | breast cancer | DISEASE |

The end is exclusive. The span `(25, 38)` includes characters 25 to 37. The length is 13 characters.

A simple check is:

```python
text = "BRCA1 mutations increase breast cancer risk"
assert text[0:5] == "BRCA1"
assert text[25:38] == "breast cancer"
```

If a check fails, the offset is wrong. Correct the offset before training.

---

## Appendix C: Writing rules used

This document uses these ASD-STE100 rules as a guide:

- Use short sentences. Most sentences have fewer than 25 words.
- Use active voice. Name the actor.
- Use one term for one concept. Do not use "tag" for "label".
- Define technical terms before use.
- Use "must" for a requirement and "can" for a possibility.
- Avoid idioms, metaphors, and phrasal verbs.
- Use a list for a sequence of steps.
- Use a table for a set of parallel items.
- Use "for example" instead of "e.g.".
- Use "that is" instead of "i.e.".

The document keeps code names and spaCy component names unchanged. A full dictionary check is not part of this document.
