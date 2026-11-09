---
name: deep-learning-training-log-management
description: Use when training a Keras/TensorFlow model (e.g., a two-layer CNN on
  dna2vec-encoded data) whose console output must be captured into a persistent log
  file for later review or provenance.
when_to_use_negative:
- You only need final performance metrics and the training script already writes them
  to a structured output file (e.g., a CSV of per-cell-line results) — a log adds
  provenance but is not the metric source of record
- You are running dna2vec encoding (get_datavec.py), which is a preprocessing step
  before training and does not produce the training-progress console output this skill
  captures
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3071
tools:
- name: LogUtils.py
  role: Dedicated logging utility that creates the log file capturing training output
    from the DeepSE CNN training scripts
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Keras
  role: Framework whose model fitting (two-layer CNN) generates the training console
    output that LogUtils.py records
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Python
  role: Runtime environment (>=3.6) in which the logging utility and training scripts
    execute
  repo: https://github.com/QiaoyingJi/DeepSE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:d4addd54fa693788d143d82d09f145893d1ddb766ed0f55073d2162610e14ace
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
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# deep-learning-training-log-management

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A logging practice for deep learning model training that records console output (loss, metrics, progress) from Keras-based training runs to persistent log files via a dedicated logger utility, as used in DeepSE's CNN training scripts.

## When to use

Use when training a Keras/TensorFlow model (e.g., a two-layer CNN on dna2vec-encoded data) whose console output must be captured into a persistent log file for later review or provenance.

## When NOT to use

- You only need final performance metrics and the training script already writes them to a structured output file (e.g., a CSV of per-cell-line results) — a log adds provenance but is not the metric source of record
- You are running dna2vec encoding (get_datavec.py), which is a preprocessing step before training and does not produce the training-progress console output this skill captures

## Inputs

- Training script console output from a Keras model fit (e.g., train_integrative1.py with learning_rate and epochs arguments)
- LogUtils.py logger module from the DeepSE repository

## Outputs

- Log file recording the full training process (per-epoch progress and metrics) for a given DeepSE training run

## How to apply

Import the LogUtils.py logger into the training script (train_specific.py, train_integrative1.py, train_integrative2.py, or train_integrative3.py) before model fitting begins. Configure the logger to write everything printed to the console — epoch-level training progress and metrics — to a log file alongside the run, so that long trainings (e.g., 80 epochs at learning rate 0.001) can be audited afterward. The logging runs in parallel with console output; it does not alter model configuration or the training data pipeline (dna2vec-encoded k-mer feature matrices loaded from './data/<cell_name>'). After training, use the log file to confirm the run completed all epochs and to retrieve the recorded performance metrics for compiling result tables.

## Related tools

- **LogUtils.py** (Dedicated logging utility that creates the log file capturing training output from the DeepSE CNN training scripts) — https://github.com/QiaoyingJi/DeepSE
- **Keras** (Framework whose model fitting (two-layer CNN) generates the training console output that LogUtils.py records) — https://github.com/QiaoyingJi/DeepSE
- **Python** (Runtime environment (>=3.6) in which the logging utility and training scripts execute) — https://github.com/QiaoyingJi/DeepSE

## Examples

```
python train_integrative1.py mm 0.001 80  # LogUtils.py records the console training output to a log file
```

## Evaluation signals

- A log file exists in the working directory after the training run completes and is non-empty
- The log file content matches the console output, including entries for all requested epochs (e.g., 80) and the specified learning rate setting (e.g., 0.001)
- The log contains the final-epoch performance metrics that can be cross-checked against metrics compiled downstream (e.g., per-cell-line performance tables)
- Enabling logging did not change model behavior — identical training command without the logger yields the same model configuration and data pipeline

## Limitations

- Logging captures training output but does not itself save model weights or encoded data; reproducibility of results still depends on the DNA2vec-encoded matrices in './data/<cell_name>'
- Requires Python >= 3.6 and the DeepSE requirements.txt environment to be installed first

## Evidence

- [readme] The modules requried by DeepSE is provied in the file requirements.txt, and you can install them by pip install requirements.txt: [evidence span withheld — anchor status: not_found]
- [other] Record training progress with the LogUtils.py logger, writing console output to a log file.: [evidence span withheld — anchor status: not_found]
