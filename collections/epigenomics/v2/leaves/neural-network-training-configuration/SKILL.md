---
name: neural-network-training-configuration
description: Use when training the DeepSE two-layer CNN (Keras) on encoded SE/TE feature
  vectors for a cell line or species, e.g. via train_specific.py or train_integrative*.py
  with species hg/mm, learning_rate, and epochs arguments.
when_to_use_negative:
- Input is raw FASTA sequences not yet encoded — run get_datavec.py to produce dna2vec
  feature-vector matrices before any training script
- The task is cell-line specific prediction with no held-out or cross-cell-line generalization
  requirement — use train_specific.py rather than the integrative modes
- The goal is evaluating cross-species transfer (train on one species, test on another,
  e.g. hg to mm) via train_integrative3.py rather than within-species held-out cell
  line testing (train_integrative2.py)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepSE
  role: Two-layer CNN framework whose training scripts (train_specific.py, train_integrative1/2/3.py)
    implement this skill; distinguishes SEs from TEs
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Keras
  role: Implements the two-layer CNN model (model.py) that the training scripts configure
    and fit
- name: dna2vec
  role: Provides the pre-trained k-mer feature embeddings used to encode SE/TE sequences
    into training matrices
- name: Python
  role: Runtime environment (>=3.6) in which the training scripts and their parameters
    (learning_rate, epochs) are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:676914685f6e8fca8964f9ace52dfa379ab4fdcc0484a341245db4e0a134263e
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
  - Python
  - dna2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# neural-network-training-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run two-layer CNN training for DeepSE, distinguishing super-enhancers (SEs) from typical enhancers (TEs) using dna2vec sequence embeddings, with command-line control of learning rate and epoch count for cell-line specific, integrative, and cross-species training modes.

## When to use

Use when training the DeepSE two-layer CNN (Keras) on encoded SE/TE feature vectors for a cell line or species, e.g. via train_specific.py or train_integrative*.py with species hg/mm, learning_rate, and epochs arguments.

## When NOT to use

- Input is raw FASTA sequences not yet encoded — run get_datavec.py to produce dna2vec feature-vector matrices before any training script
- The task is cell-line specific prediction with no held-out or cross-cell-line generalization requirement — use train_specific.py rather than the integrative modes
- The goal is evaluating cross-species transfer (train on one species, test on another, e.g. hg to mm) via train_integrative3.py rather than within-species held-out cell line testing (train_integrative2.py)

## Inputs

- Encoded SE/TE feature-vector matrices per cell line under ./data/<cell_name> (dna2vec k-mer embeddings, produced by get_datavec.py)
- DNA sequence files in FASTA format for TEs and SEs, organized by cell name
- Pre-trained dna2vec embedding matrices for human (hg19) or mouse (mm9)
- CLI parameters: cell_name or species (hg/mm), learning_rate, epochs

## Outputs

- Trained DeepSE-specific or DeepSE-integrative two-layer CNN model (Keras)
- Log file recording the training process (via LogUtils.py)
- Prediction results / classification metrics on the test cell line (e.g. integrative2_heldout_metrics.csv)

## How to apply

First prepare the input by running get_datavec.py (e.g. 'python get_datavec.py mESC mm9') so SE/TE FASTA sequences are converted into dna2vec k-mer embedding matrices under ./data/<cell_name>; pre-trained embeddings exist for human hg19 and mouse mm9. Then select a training script: train_specific.py for a single cell line, train_integrative1.py to pool all cell line datasets, train_integrative2.py to pool several cell lines and test on a new cell line, or train_integrative3.py to train on one species and test on another, using the syntax 'python <script> species/cell_name learning_rate epochs'. The paper's runs used learning_rate 0.001 and epochs 80 (e.g. 'python train_integrative2.py hg 0.001 80'), with the 'species' parameter set to hg (human) or mm (mouse). The two-layer CNN is implemented by Keras in model.py, and LogUtils.py creates a log file during training so progress is recorded on console and in log files. Validation consists of confirming the training log records completed epochs without errors and collecting prediction metrics on the held-out or test cell line (e.g. into integrative2_heldout_metrics.csv).

## Related tools

- **DeepSE** (Two-layer CNN framework whose training scripts (train_specific.py, train_integrative1/2/3.py) implement this skill; distinguishes SEs from TEs) — https://github.com/QiaoyingJi/DeepSE
- **Keras** (Implements the two-layer CNN model (model.py) that the training scripts configure and fit)
- **dna2vec** (Provides the pre-trained k-mer feature embeddings used to encode SE/TE sequences into training matrices)
- **Python** (Runtime environment (>=3.6) in which the training scripts and their parameters (learning_rate, epochs) are executed)

## Examples

```
python train_integrative2.py hg 0.001 80
```

## Evaluation signals

- The command runs with the documented parameter pattern: python train_integrative2.py species learning_rate epochs, with species limited to hg or mm
- LogUtils.py creates a log file and the training log records the configured epochs (e.g. 80) completed without errors
- Encoded data exist at ./data/<cell_name> for every cell line used in training before invoking a training script
- Prediction results on the held-out cell line are produced and compiled into a metrics table (e.g. integrative2_heldout_metrics.csv) containing classification metrics for the unseen cell line
- The model is a two-layer CNN implemented in Keras (model.py), matching the paper's architecture

## Limitations

- The reported hyperparameter setting (learning_rate 0.001, epochs 80) is specific to the paper's experiments; other tasks may require re-tuning
- Only two pre-trained dna2vec embedding matrices are provided (human hg19 and mouse mm9); other genomes require training a custom dna2vec model
- Cross-species and held-out-cell-line modes (train_integrative3.py, train_integrative2.py) make generalization assumptions — DeepSE's generalization across cell lines is an empirical finding, not guaranteed for any new species or cell type
- Generalization is claimed only across cell lines; the article notes no discussion-section caveats are available for extraction

## Evidence

- [methods] To predict SEs in cell-line specific way, you can type the command line like follows: python train_specific.py cell_name learning_rate epochs: [evidence span withheld — anchor status: not_found]
