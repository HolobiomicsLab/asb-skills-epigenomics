---
name: training-log-and-metrics-recording
description: Use when training a DeepSE model (specific or integrative) on dna2vec-encoded
  SE/TE data and you need reproducible training logs plus saved evaluation metrics
  tied to the trained Keras model.
when_to_use_negative:
- You only need one-off results and no reproducibility record — the log files are
  supplementary, not required for a model to train
- You are only running the data-encoding step (get_datavec.py) and no model training
  occurs
- Input is already a trained model checkpoint with an existing metrics record
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: DeepSE
  role: Two-layer CNN training pipeline whose training process output is captured
    and logged
  repo: https://github.com/QiaoyingJi/DeepSE
- name: LogUtils.py
  role: Creates a log file during training so console training information is recorded
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Keras
  role: Implements the two-layer CNN model whose training progress and evaluation
    metrics are recorded
- name: Python
  role: Runtime environment (>=3.6) for the logging utilities and training scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:1f13bd50ca2393910351d659603a587c8c8a00be3c407ae0f2519f32070d95e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/QiaoyingJi/DeepSE
  repo_url: https://github.com/QiaoyingJi/DeepSE
  tools:
  - DeepSE
  - Keras
  - LogUtils.py
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Training log and metrics recording

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Records console training output (loss/progress per epoch) into log files using DeepSE's LogUtils.py while a two-layer Keras CNN (DeepSE) is trained on dna2vec-encoded SE/TE matrices, and saves held-out evaluation metrics (e.g. accuracy/AUC) alongside the trained model.

## When to use

Use when training a DeepSE model (specific or integrative) on dna2vec-encoded SE/TE data and you need reproducible training logs plus saved evaluation metrics tied to the trained Keras model.

## When NOT to use

- You only need one-off results and no reproducibility record — the log files are supplementary, not required for a model to train
- You are only running the data-encoding step (get_datavec.py) and no model training occurs
- Input is already a trained model checkpoint with an existing metrics record

## Inputs

- Encoded mESC (or other cell-line) SE/TE dna2vec feature matrices from './data/<cell_name>'
- Training run configuration: cell_name, learning_rate (e.g. 0.001), epochs (e.g. 80)
- Held-out SE/TE test data for evaluation

## Outputs

- Log file(s) capturing console training output (per-epoch loss/progress)
- Trained Keras two-layer CNN model file
- Evaluation metrics record (e.g. accuracy/AUC) on held-out SE/TE data

## How to apply

After encoding the SE/TE FASTA sequences into k-mer dna2vec feature matrices (e.g. './data/mESC'), instantiate the two-layer CNN in model.py and launch training with train_specific.py (e.g. cell_name=mESC, learning_rate=0.001, epochs=80 as in the paper's demo). LogUtils.py hooks into the training run so that information printed during the training process is simultaneously written to log files, giving a persistent epoch-level record of parameters and progress. Once training completes, evaluate the model on the held-out SE/TE data for that cell line and save a metrics record (accuracy/AUC) next to the serialized Keras model so each trained checkpoint is paired with its performance. The saved log and metrics files are what make reruns of the demo command comparable and auditable.

## Related tools

- **DeepSE** (Two-layer CNN training pipeline whose training process output is captured and logged) — https://github.com/QiaoyingJi/DeepSE
- **LogUtils.py** (Creates a log file during training so console training information is recorded) — https://github.com/QiaoyingJi/DeepSE
- **Keras** (Implements the two-layer CNN model whose training progress and evaluation metrics are recorded)
- **Python** (Runtime environment (>=3.6) for the logging utilities and training scripts)

## Examples

```
python train_specific.py mESC 0.001 80
```

## Evaluation signals

- Log file exists and contains the training information that was printed on the console (epochs, learning_rate, per-epoch progress)
- Log parameters match the invocation (e.g. epochs=80, learning_rate=0.001 for the mESC demo)
- A metrics record (accuracy/AUC) is saved alongside the trained Keras model after held-out evaluation
- Encoded input matrices are present under './data/mESC' before training starts
- Rerunning the same demo command produces a comparable log and metrics record

## Limitations

- Third-party packages must be installed from requirements.txt in a Python >=3.6 environment or logging/training scripts will fail
- The README's install line ('pip install requirements.txt') is not the correct pip syntax; correct usage is 'pip install -r requirements.txt'
- Logging records training-process information only; evaluation metrics must be saved separately by the practitioner

## Evidence
