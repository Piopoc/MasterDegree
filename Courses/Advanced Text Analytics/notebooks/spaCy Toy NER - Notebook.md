---
type: course-note
date: 2026-11-10
tags:
  - course
  - advanced-text-analytics
  - notebook
  - ner
  - spacy
course: "[[Advanced Text Analytics]]"
sources:
  - "[[ATA_L11_spaCy_Toy_NER]]"
ai-first: true
---

## For Future Agent

Notebook companion for Lecture 11, part II: theory of the spaCy toy NER fine-tuning notebook (`ATA_L11_spaCy_Toy_NER_Finetuning.ipynb`), extracted 2026-11-10 from the ASD-STE100 theory notes in raw. It covers the blank-pipeline training workflow (`Example` objects, `nlp.initialize`, `nlp.update`), character-offset annotations, the transition-based BILUO `EntityRecognizer` vs a BIO token classifier, and fitting-vs-generalization with the four error types. Central caveats: the model trains **from scratch** (not from pretrained weights) on five sentences, and the loss curve shows fitting, never generalization.

## 1. Position in the Course

The notebook sits at the middle level of a three-level progression:

```text
EntityRuler                      -> rule-based NER
blank spaCy + trainable ner     -> supervised NER from scratch   <- this notebook
pretrained spaCy / Transformer  -> fine-tuned NER
```

Between the deterministic `EntityRuler` and the pretrained pipelines: it shows learning mechanics without the complexity of a pretrained Transformer. The workflow it exercises:

```text
annotated text -> spaCy Example objects -> trainable NER component
   -> predicted entity spans -> training loss -> parameter updates
```

**On the word "fine-tuning":** the title says fine-tuning, but the model starts from a **blank** pipeline - it trains from *scratch* while reusing the fine-tuning *workflow* (same loop, different starting point). A real fine-tuning run starts from pretrained weights; the loop is the same in principle, the result is not.

## 2. The spaCy Pipeline Model

The `nlp` object holds the vocabulary, the tokenizer and the pipeline components; `nlp(text)` tokenizes, then each component annotates the shared `Doc`. `spacy.blank("en")` builds an English pipeline with **no trained components** - tokenizer only, `nlp.pipe_names` empty. Useful for teaching: add one component, see its effect, no hidden pretrained model.

`EntityRuler` vs `ner` (`EntityRecognizer`) - both find entities, different way:

| Item | `EntityRuler` | `ner` |
|---|---|---|
| Method | rules and patterns | statistical model |
| Training | none | annotated examples |
| Output | deterministic spans | predicted spans |
| Adding names | add a pattern | add examples / retrain |
| Errors | only wrong/missing rules | can generalize, makes statistical errors |

They are complementary (a common pipeline keeps both); this notebook trains only `ner` to show the statistical workflow.

## 3. Annotation with Character Offsets

Gold entities arrive as `(start_char, end_char, label)`:

```text
"BRCA1 mutations increase breast cancer risk"
BRCA1         (0, 5)    GENE
breast cancer (25, 38)  DISEASE
```

- **End is exclusive** (Python slice convention): `(25, 38)` covers characters 25..37, length `end - start` = 13 = len(`breast cancer`).
- Offsets **must align with token boundaries** - a span starting mid-token cannot be assigned to a whole token and the annotation is invalid for the model. The notebook checks offsets before training: a wrong offset is a wrong gold label, and the model learns the wrong pattern.
- Why offsets: tokenizer-independent and they preserve the original text (tokenizers can change, positions stay); the cost is that they must be correct - a verification tool that prints text + span is worth it.

## 4. `Example` Objects

An `Example` pairs two `Doc`s:

1. the **predicted** document - what the model sees, built with `nlp.make_doc(text)` (tokenized, pipeline not run);
2. the **reference** document - the gold annotation.

`Example.from_dict(doc, {"entities": [(start, end, label), ...]})` builds it: spaCy creates the reference and aligns entities to tokens. The `Example` is the bridge between annotation format and training loop; it keeps input and gold separate so spaCy can compare them - and **invalid annotations fail here, before training**, instead of silently producing wrong labels.

## 5. Initialization

`optimizer = nlp.initialize(lambda: examples)` prepares the pipeline:

1. reads the labels from the components;
2. builds the model parameters;
3. infers input/output shapes from the examples (the `lambda` hands them over);
4. creates the optimizer.

Because the pipeline is blank, parameters start from random values - `tok2vec` is a small network trained from scratch, unlike a real pretrained spaCy pipeline. The returned **optimizer is not the model**: the model lives in the pipeline; the optimizer holds the update rule and optimization state, and `nlp.update` uses it to change parameters.

## 6. The Training Loop

- **Epochs**: one full pass over the data; 50 epochs on five examples - necessary only to see a clear loss decrease.
- **Shuffle**: `random.shuffle(examples)` at each epoch start, so the model does not learn a fixed order and batches differ across epochs.
- **Minibatches**: `minibatch(examples, size=2)` - with five examples, three batches (2 + 2 + 1); the small last batch is normal.
- **`nlp.update(batch, sgd=optimizer, drop=0.2, losses=losses)`** does the full step: run pipeline -> compare with gold -> NER loss -> gradients -> optimizer update. `drop` is dropout (20% of values zeroed in the forward pass, off at inference); with five examples it can slow fitting, but it stays to show the standard workflow.
- **Losses**: `losses.get("ner", 0.0)` reads the NER key (0.0 as safe default). The value is a **sum over the epoch's batches**, not normalized - fine for a rough curve on a fixed dataset.
- **The curve**: decreasing loss = the model fits the training examples. It does **not** show generalization; with five sentences the model can memorize. The curve is a diagnostic, not a performance metric - performance needs a test set and entity-level F1.

## 7. Transition-Based NER (BILUO)

The PyTorch companion notebook classifies each token independently (`vector -> linear classifier -> BIO label`) and needs a decoder to repair invalid sequences. spaCy's standard `EntityRecognizer` instead is a **transition-based parser**: it predicts a sequence of actions, each building part of an entity, and the sequence is **valid by construction**.

BILUO actions:

| Symbol | Meaning |
|---|---|
| `U` | unit - single-token entity |
| `B` | begin - first token of a multi-token entity |
| `I` | inside - middle token |
| `L` | last - final token |
| `O` | outside any entity |

```text
BRCA1      U-GENE
mutations  O
increase   O
breast     B-DISEASE
cancer     L-DISEASE
risk       O
```

The parser reads tokens left to right, keeps a state (current position + entities built so far), scores the **valid** actions and greedily takes the highest. Validity constraints (`I-TYPE`/`L-TYPE` need an open entity of type TYPE) make `I-GENE` without an open GENE entity unrepresentable. The `tok2vec` layer provides token vectors; in this notebook it is small and trained from scratch.

**Training signal**: gold spans derive a gold action sequence; the parser's scores are compared against it -> loss -> weight update. The exact loss is an internal spaCy detail; the principle is identical to the PyTorch notebook: prediction -> compare with gold -> loss -> gradient update.

**Greedy decoding and error propagation**: no backtracking - a wrong action (e.g. starting an entity at the wrong token) can make later actions wrong too. A known limit of greedy transition-based systems; beam search reduces it at a cost. The notebook keeps the default greedy behavior.

BIO token classifier vs transition-based NER:

| Item | BIO token classifier | Transition-based NER |
|---|---|---|
| Output | one label per token | one action per step |
| Validity | can emit invalid sequences | always valid by construction |
| Decision | independent per token | depends on state and history |
| Decoding | external decoder needed | the action sequence *is* the decode |
| Errors | local label errors | local action errors can propagate |
| Example | PyTorch companion | spaCy `EntityRecognizer` |

Both learn from gold annotations with a loss and gradients; the difference is the output structure.

## 8. Fitting vs Generalization

50 epochs on five sentences: the NER loss decreases (fitting), but the notebook then tests unseen sentences:

```text
TP53 is associated with breast cancer
EGFR mutations occur in lung cancer
BRCA2 mutations increase cancer risk
Alzheimer disease affects the brain
```

Results can be inconsistent - some entities found, others missed. Expected: five examples cannot teach a general rule; the model can memorize the exact sentences and still fail on new words in new orders. **A low training loss is not a quality score.**

A real evaluation needs: train set, dev set for tuning, test set for the final score, entity-level P/R/F1, one fixed evaluation rule, and an error analysis. The notebook has none of these - teaching example, not a benchmark.

## 9. Error Taxonomy

| Error type | Definition | Example |
|---|---|---|
| Missed entity | gold entity not predicted | gold `BRCA1` GENE; model finds nothing |
| Extra entity | predicted entity not in gold | model tags `many` as DISEASE |
| Boundary error | right type, wrong start/end | gold `breast cancer`; prediction `cancer` |
| Type error | right span, wrong type | gold `BRCA1` GENE; prediction `BRCA1` DISEASE |

Under strict entity-level evaluation: missed -> one FN; extra -> one FP; boundary and type errors -> one FP **and** one FN each - strict matching gives every error type a clear penalty.

Inspection recipe: (1) diff gold vs predicted spans, (2) assign each difference one of the four types, (3) count and look for patterns - does the model miss long entities? stop at the first token? confuse GENE and DISEASE? The answers pick the next step: more data, better annotation, another model, or a rule.

## 10. Comparison with the PyTorch Companion

| PyTorch toy | spaCy toy |
|---|---|
| explicit tensors | framework abstractions |
| explicit hidden states | representation model internal |
| explicit BIO classifier | transition-based NER component |
| explicit loss computation | spaCy computes the NER loss |
| `loss.backward()` | inside `nlp.update()` |
| `optimizer.step()` | via `sgd=optimizer` |

Same learning principle (prediction -> gold -> loss -> gradient update); the difference is **level of control** - the PyTorch notebook shows every step, spaCy hides the low-level steps. Architecturally: token classifier vs transition-based parser, with three effects - validity (spaCy cannot emit invalid BILUO), history (spaCy conditions on state; the PyTorch classifier does not, unless encoder/CRF adds it), and error propagation (spaCy's wrong action cascades; a wrong token label stays local, though the decoder can still produce a wrong span). Both designs are valid; the choice depends on task, data and need for control.

## 11. What the Notebook Does Not Do

Deliberately omitted: train/dev/test split (five sentences would leave too few per part), pretrained spaCy model or Transformer (would hide training mechanics), subword tokenization (would add a second alignment problem), systematic evaluation (needs more data), hyperparameter tuning. These omissions are a **teaching choice, not a recommendation** - the purpose is architectural and procedural transparency: show the workflow, not the best result.

## 12. Natural Next Steps

1. **`EntityRuler`**: add gene/disease name lists and patterns; measure P/R on a small test set.
2. **Blank spaCy + `ner`**: more annotated examples, a dev set, entity-level F1, the four error types analyzed.
3. **Pretrained pipeline**: fine-tune a pretrained spaCy model or Transformer, compare with the from-scratch run, check cost and speed - this step reuses the Lecture 10 ideas (pretrained encoder, task head, supervised loss, full vs head-only fine-tuning): same workflow, different model.

## Key Points

1. spaCy trains a statistical NER model from annotated entity spans (`Example` = input Doc + gold reference).
2. `nlp.initialize` prepares the pipeline and returns the optimizer; `nlp.update` does forward, loss, backward and update in one call.
3. The standard `EntityRecognizer` is **transition-based with BILUO actions**, not a simple BIO classifier - valid sequences by construction, greedy decisions, hence error propagation.
4. Decreasing loss on five examples shows fitting, not generalization.
5. Error analysis uses four types: missed, extra, boundary, type (strict match: boundary/type = FP + FN).
6. This notebook bridges rule-based `EntityRuler` NER and pretrained fine-tuning.

## Related

- [[Supervised Named Entity Recognition - Notebook]] - Lecture 11 part I: BIO token classification, manual gradients, the toy Transformer
- [[Annotation and Evaluation of Information Extraction]] - parent theory note: BIO/BILOU schemes, P/R/F1, exact vs relaxed matching
- [[Knowledge Resources for Text Analytics]] - Lecture 5: the rule-based `EntityRuler`/`PhraseMatcher` level of the same progression
- [[Fine-Tuning and Domain Adaptation]] - Lecture 10: what real fine-tuning from pretrained weights adds
- [[Advanced Text Analytics]] - course hub

## Sources

- [[ATA_L11_spaCy_Toy_NER]] (AY 2026/2027)