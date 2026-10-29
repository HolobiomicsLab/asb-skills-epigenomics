---
name: cross-cell-line-model-generalization
description: Use when you have encoded SE/TE sequence datasets for multiple cell lines
  and need to evaluate whether a sequence-only classifier trained on some cell lines
  predicts super-enhancers in a novel, held-out cell line.
when_to_use_negative:
- Input already consists of numeric feature tables (e.g. pre-encoded matrices) — re-running
  get_datavec.py is unnecessary; proceed directly to training
- The goal is ChIP-seq-based or epigenomic-feature-based SE/TE classification — DeepSE
  is designed to identify super-enhancers without ChIP-seq data using DNA sequence
  only
- You need a cell-line specific predictor rather than a generalization test — use
  train_specific.py instead of the integrative scripts
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: DeepSE
  role: Two-layer CNN model that distinguishes super-enhancers from typical enhancers
    using only sequence feature embeddings; train_integrative1.py and train_integrative2.py
    implement the pooled and held-out cross-cell-line evaluations
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Provides the pre-trained k-mer DNA feature embeddings (hg19 and mm9 matrices)
    used to encode SE/TE sequences into feature vectors before CNN training
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Framework in which the two-layer CNN model (model.py) is implemented
- name: Python
  role: Runtime environment (Python >=3.6) in which encoding, training, and logging
    scripts are executed
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:e3112bac5436ca0c17b30563682ddfb2c01312e6ce20da8fd1c5c6d0e712e347
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
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-cell-line-model-generalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains DeepSE, a two-layer CNN on dna2vec k-mer feature embeddings, on pooled cell line datasets (e.g. mESC, myotube, H2171) and tests whether the pooled or integrative model generalizes to held-out cell lines, testing whether cell-type specific super-enhancers share hidden sequence patterns.

## When to use

Use when you have encoded SE/TE sequence datasets for multiple cell lines and need to evaluate whether a sequence-only classifier trained on some cell lines predicts super-enhancers in a novel, held-out cell line.

## When NOT to use

- Input already consists of numeric feature tables (e.g. pre-encoded matrices) — re-running get_datavec.py is unnecessary; proceed directly to training
- The goal is ChIP-seq-based or epigenomic-feature-based SE/TE classification — DeepSE is designed to identify super-enhancers without ChIP-seq data using DNA sequence only
- You need a cell-line specific predictor rather than a generalization test — use train_specific.py instead of the integrative scripts

## Inputs

- DNA sequence files in 'fasta' format for SEs and TEs of each cell line (e.g. mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)
- Pre-trained dna2vec k-mer embedding matrices for hg19 (human) or mm9 (mouse)
- Pre-encoded cell line feature matrices under './data/<cell_name>' (uploaded for all nine cell lines)
- Training hyperparameters: species setting (hg or mm), learning_rate (e.g. 0.001), epochs (e.g. 80)

## Outputs

- Trained DeepSE-integrative two-layer CNN model
- Per-cell-line classification performance metrics for the pooled universal model (deepse_integrative_per_cell_line_performance.csv)
- Discrimination metrics on the held-out cell line (integrative2_heldout_metrics.csv)
- Training log file created by LogUtils.py recording console output

## How to apply

Encode the DNA sequences of TEs and SEs for each cell line (FASTA input) into k-mer feature vectors using pre-trained dna2vec embeddings via get_datavec.py with the cell name and genome version (hg19 for human, mm9 for mouse), producing encoded matrices under './data/<cell_name>'. Pool the encoded datasets and train the two-layer Keras CNN via train_integrative1.py (universal model tested per cell line) or train_integrative2.py (pooled training on several cell lines, tested in a new cell line), using learning_rate 0.001 and epochs 80, with the species parameter set to hg or mm. Log training progress with LogUtils.py so epochs completed and errors are recorded. Collect per-cell-line classification metrics on each tested cell line into a table (e.g. deepse_integrative_per_cell_line_performance.csv or integrative2_heldout_metrics.csv) and check whether performance on unseen cell lines supports cross-cell-line generalization.

## Related tools

- **DeepSE** (Two-layer CNN model that distinguishes super-enhancers from typical enhancers using only sequence feature embeddings; train_integrative1.py and train_integrative2.py implement the pooled and held-out cross-cell-line evaluations) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Provides the pre-trained k-mer DNA feature embeddings (hg19 and mm9 matrices) used to encode SE/TE sequences into feature vectors before CNN training) — https://ngpatrick.com/dna2vec/
- **Keras** (Framework in which the two-layer CNN model (model.py) is implemented)
- **Python** (Runtime environment (Python >=3.6) in which encoding, training, and logging scripts are executed)

## Examples

```
python train_integrative2.py hg 0.001 80
```

## Evaluation signals

- Verify the output metrics table (deepse_integrative_per_cell_line_performance.csv or integrative2_heldout_metrics.csv) contains a performance record for every tested cell line, including the held-out one
- Confirm the training log file created by LogUtils.py records the epochs completed (e.g. 80) without errors
- Confirm encoded matrices exist under './data/<cell_name>' for each cell line after running get_datavec.py
- Check that per-cell-line metrics on the pooled model and held-out cell line metrics support the reported finding that DeepSE generalizes well across different cell lines
- Verify the species parameter (hg or mm) matches the genome version of the embedding matrix and cell line datasets used

## Limitations

- Cross-species generalization requires a different script (train_integrative3.py) training on one species and testing on another; hg and mm datasets are handled separately
- Only two pre-trained embedding matrices are provided (hg19 and mm9); other genomes require training a custom dna2vec model
- The article's README notes no discussion-section changelog was extracted, so nuance about why generalization works beyond the stated 'hidden sequence patterns' claim is not documented
- The 'cell_name' for 'pro-B cell' contains a space, which may complicate directory naming under './data/<cell_name>'

## Evidence

- [abstract] DeepSE can be generalized well across different cell lines, which implied that cell-type specific SEs may share hidden sequence patterns across different cell lines: "DeepSE can be generalized well across different cell lines, which implied that cell-type specific SEs may share hidden sequence patterns across different cell lines"
- [methods] Train an integrative model by pooling several cell line datasets, and testing in a new cell line.: "Train an integrative model by pooling several cell line datasets, and testing in a new cell line."
