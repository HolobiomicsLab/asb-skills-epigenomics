---
name: multi-cell-line-dataset-pooling
description: 'Use when the goal is cross-cell-line generalization of sequence-based
  SE/TE classification: you have per-cell-line encoded dna2vec feature matrices and
  want one universal pooled model evaluated on each cell line.'
when_to_use_negative:
- You want a cell-line specific model evaluated only within one cell line — use train_specific.py
  instead of pooling
- You want to test generalization to a completely held-out new cell line — use train_integrative2.py
  (pool several lines, test on a new one), not pooled training over all lines
- You want cross-species transfer (train on human, test on mouse) — use train_integrative3.py
  instead
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DeepSE
  role: Two-layer CNN framework distinguishing super-enhancers from typical enhancers;
    the integrative variant pools all cell line datasets for a universal model
  repo: https://github.com/QiaoyingJi/DeepSE
- name: train_integrative1.py
  role: Training script that pools all cell line datasets together, trains the universal
    model, and tests it on every cell line respectively
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Provides the k-mer feature embedding matrices used to encode SE/TE DNA sequences
    before pooling
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Implements the two-layer CNN model (model.py) that is trained on the pooled
    datasets
- name: LogUtils.py
  role: Creates a log file recording the training process
  repo: https://github.com/QiaoyingJi/DeepSE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:2c52419260d5b08a42c384d7de581651e8c214a39524f462c61f7103b180acf4
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
  - dna2vec
  - train_integrative1.py
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# multi-cell-line-dataset-pooling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a universal DeepSE-integrative model by pooling pre-encoded dna2vec k-mer feature matrices from all nine cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87) with a two-layer Keras CNN, then tests the pooled model per cell line to assess cross-cell-line generalization of super-enhancer vs typical-enhancer prediction.

## When to use

Use when the goal is cross-cell-line generalization of sequence-based SE/TE classification: you have per-cell-line encoded dna2vec feature matrices and want one universal pooled model evaluated on each cell line.

## When NOT to use

- You want a cell-line specific model evaluated only within one cell line — use train_specific.py instead of pooling
- You want to test generalization to a completely held-out new cell line — use train_integrative2.py (pool several lines, test on a new one), not pooled training over all lines
- You want cross-species transfer (train on human, test on mouse) — use train_integrative3.py instead

## Inputs

- Pre-encoded k-mer dna2vec feature matrices for nine cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87) located under './data/<cell_name>'
- Pre-trained dna2vec embedding matrices for human (hg19) and mouse (mm9), or a custom embedding trained via dna2vec
- SE/TE DNA sequence files in FASTA format per cell line, if encoding from scratch

## Outputs

- A universal DeepSE-integrative two-layer CNN model trained on pooled data
- Per-cell-line classification performance metrics for all nine tested cell lines, compiled into deepse_integrative_per_cell_line_performance.csv
- Training log file produced by LogUtils.py

## How to apply

Load the pre-encoded k-mer dna2vec feature matrices for all nine cell lines from their './data/<cell_name>' directories (each produced by get_datavec.py with genome hg19 or mm9, or use the pre-uploaded encoded datasets). Pool all nine datasets together and train the two-layer CNN (model.py in Keras) via train_integrative1.py, setting the species parameter to the pooled datasets, learning_rate = 0.001, and epochs = 80, mirroring the demo settings used for the specific model. Record training progress with the LogUtils.py logger so console output is captured in a log file. Then test the single universal model on each of the nine cell lines separately and collect per-cell-line classification metrics, compiling them into a performance table (e.g. deepse_integrative_per_cell_line_performance.csv). The rationale is that if the pooled model performs well per cell line, cell-type specific super-enhancers share hidden sequence patterns across cell lines.

## Related tools

- **DeepSE** (Two-layer CNN framework distinguishing super-enhancers from typical enhancers; the integrative variant pools all cell line datasets for a universal model) — https://github.com/QiaoyingJi/DeepSE
- **train_integrative1.py** (Training script that pools all cell line datasets together, trains the universal model, and tests it on every cell line respectively) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Provides the k-mer feature embedding matrices used to encode SE/TE DNA sequences before pooling) — https://ngpatrick.com/dna2vec/
- **Keras** (Implements the two-layer CNN model (model.py) that is trained on the pooled datasets)
- **LogUtils.py** (Creates a log file recording the training process) — https://github.com/QiaoyingJi/DeepSE

## Examples

```
python train_integrative1.py mm 0.001 80
```

## Evaluation signals

- Output performance table contains a record for every one of the nine tested cell lines (mESC_constituent, mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)
- Per-cell-line test metrics support the paper's finding that DeepSE generalizes well across different cell lines (competitive SE/TE classification performance from a single pooled model)
- Encoded data directory './data/<cell_name>' exists and is non-empty for each of the nine cell lines before training
- Training log file from LogUtils.py records the full 80-epoch run at learning_rate = 0.001

## Limitations

- Requires Python >= 3.6 runtime environment with third-party packages installed from requirements.txt
- Encoding depends on genome-specific pre-trained embeddings (human hg19 or mouse mm9); custom embeddings require training a dna2vec model separately
- Pooling everything means the universal model never tests on a fully unseen cell line; for that, train_integrative2.py or train_integrative3.py are the appropriate designs

## Evidence

- [readme] Create a universal prediction model by pooling all kinds of cell line data together.: [evidence span withheld — anchor status: not_found]
