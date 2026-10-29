---
name: cnn-super-enhancer-classification
description: Use when the task is to predict SEs vs TEs from raw DNA sequences (FASTA)
  alone, without ChIP-seq data, for one cell line, pooled cell lines, a held-out cell
  line, or across species (hg19 vs mm9).
when_to_use_negative:
- Input is already a feature table or includes ChIP-seq/experimental annotations —
  DeepSE is designed to operate on DNA sequence only and assumes raw FASTA sequences
  as its starting point
- Sequences derive from a genome whose k-mer embedding matrix is unavailable (only
  hg19 and mm9 pre-trained matrices are supplied); you would need to train a custom
  dna2vec matrix first
- The goal is to identify enhancer activity or constituents without any labeled SE/TE
  training data — the CNN requires labeled SE and TE training examples per cell line
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepSE
  role: Two-layer CNN model (Keras) that classifies SEs vs TEs from dna2vec-encoded
    sequences; provides train_specific.py, train_integrative1/2/3.py, get_datavec.py,
    model.py, LogUtils.py
  repo: https://github.com/QiaoyingJi/DeepSE
- name: dna2vec
  role: Pre-trained k-mer feature embeddings (hg19, mm9 matrices) used to convert
    FASTA DNA sequences of TEs and SEs into feature vectors
  repo: https://ngpatrick.com/dna2vec/
- name: Keras
  role: Framework implementing the two-layer CNN (model.py) trained with learning_rate=0.001
    for 80 epochs
- name: Python
  role: Runtime environment required by DeepSE (Python >=3.6); scripts invoked from
    the command line
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:f57d8f139c20096825e362dc4c03f5a1c94be5d188dd244e03b4f384ffcdcca6
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
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cnn-super-enhancer-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

DeepSE-style two-layer CNN classification that distinguishes super-enhancers (SEs) from typical enhancers (TEs) using only DNA sequence information, represented as dna2vec k-mer feature embeddings — the first SE/TE discriminator that requires no ChIP-seq data.

## When to use

Use when the task is to predict SEs vs TEs from raw DNA sequences (FASTA) alone, without ChIP-seq data, for one cell line, pooled cell lines, a held-out cell line, or across species (hg19 vs mm9).

## When NOT to use

- Input is already a feature table or includes ChIP-seq/experimental annotations — DeepSE is designed to operate on DNA sequence only and assumes raw FASTA sequences as its starting point
- Sequences derive from a genome whose k-mer embedding matrix is unavailable (only hg19 and mm9 pre-trained matrices are supplied); you would need to train a custom dna2vec matrix first
- The goal is to identify enhancer activity or constituents without any labeled SE/TE training data — the CNN requires labeled SE and TE training examples per cell line

## Inputs

- DNA sequences of TEs and SEs in 'fasta' format, organized per cell name (e.g. mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87)
- Pre-trained dna2vec k-mer embedding matrix for human (hg19) or mouse (mm9), or a user-trained dna2vec embedding matrix
- Training parameters: species setting (hg or mm), learning_rate (0.001), epochs (80)

## Outputs

- Encoded k-mer feature matrices per cell line under './data/<cell_name>'
- Trained Keras two-layer CNN classifier distinguishing SEs from TEs
- Prediction/classification performance metrics per test cell line (compiled into e.g. integrative2_heldout_metrics.csv, cross_species_performance.csv, benchmark_comparison.csv)
- Training log files created by LogUtils.py

## How to apply

Convert the DNA sequences of TEs and SEs into feature vectors with pre-trained k-mer dna2vec embeddings (human hg19 and mouse mm9 matrices are supplied; custom embeddings can be trained via dna2vec) by running get_datavec.py with the cell name and genome, writing encoded matrices to './data/<cell_name>'. Then train the two-layer CNN implemented in Keras (model.py) with learning_rate=0.001 and epochs=80: train_specific.py for a cell-line specific model, train_integrative1.py to pool all cell line datasets into a universal model, train_integrative2.py to train on several cell lines and test on a new one, and train_integrative3.py to train on one species and test on another (species set to 'hg' or 'mm'). Log training via LogUtils.py, which records the process in log files. Classification performance on the test cell lines is the primary output; the paper reports that with only DNA sequence information DeepSE outperformed all state-of-the-art methods, and that it generalizes well across cell lines.

## Related tools

- **DeepSE** (Two-layer CNN model (Keras) that classifies SEs vs TEs from dna2vec-encoded sequences; provides train_specific.py, train_integrative1/2/3.py, get_datavec.py, model.py, LogUtils.py) — https://github.com/QiaoyingJi/DeepSE
- **dna2vec** (Pre-trained k-mer feature embeddings (hg19, mm9 matrices) used to convert FASTA DNA sequences of TEs and SEs into feature vectors) — https://ngpatrick.com/dna2vec/
- **Keras** (Framework implementing the two-layer CNN (model.py) trained with learning_rate=0.001 for 80 epochs)
- **Python** (Runtime environment required by DeepSE (Python >=3.6); scripts invoked from the command line)

## Examples

```
python get_datavec.py mESC mm9; python train_specific.py mESC 0.001 80
```

## Evaluation signals

- Encoded matrices for each cell line exist under './data/<cell_name>' after running get_datavec.py (e.g. './data/mESC' for the mESC dataset)
- The training log file created by LogUtils.py records the epochs completed without errors
- Classification performance metrics for the tested cell lines are in the [0, 1] range and present for every tested cell line (e.g. all nine cell lines in the integrative evaluation)
- Held-out and cross-species metrics (integrative2_heldout_metrics.csv, cross_species_performance.csv) support the reported finding that DeepSE generalizes well across cell lines and datasets
- Benchmark comparison (benchmark_comparison.csv) shows DeepSE outperforming state-of-the-art methods on the shared cell-line benchmarks using DNA sequence only

## Limitations

- Only two pre-trained embedding matrices are provided (human hg19 and mouse mm9); other genomes require training your own dna2vec model and embedding matrices
- The generalization finding is interpreted as evidence that cell-type specific SEs may share hidden sequence patterns across cell lines — the CNN learns sequence-level patterns, not regulatory mechanisms
- Performance depends on the quality of the pre-trained k-mer embeddings; encoded data must be regenerated with get_datavec.py for any new cell line or genome before training

## Evidence

- [readme] DeepSE is a model based on a two-layer CNN model using computational methods to predict the SEs from TEs.: "DeepSE is a model based on a two-layer CNN model using computational methods to predict the SEs from TEs."
- [abstract] dna2vec feature embeddings: "DeepSE represent the DNA sequences using the dna2vec feature embeddings"
- [readme] Convert DNA sequences to feature vectors: "Convert the DNA sequence of TEs and SEs into feature vectors with the pre-trained k-mer feature vectors."
- [methods] Keras implementation: "A two-layer CNN model was implemented by Keras."
- [abstract] Generalization across cell lines: "DeepSE can be generalized well across different cell lines, which implied that cell-type specific SEs may share hidden sequence patterns across different cell lines"
