---
name: cross-cell-line-performance-comparison
description: Use when a DNA-sequence-only super-enhancer vs typical-enhancer classifier
  must be evaluated per cell line and compared against reported baseline methods on
  shared test benchmarks.
when_to_use_negative:
- The task involves predicting super-enhancers with ChIP-seq-derived features rather
  than DNA sequence only — DeepSE is explicitly designed to avoid ChIP-seq data
- The input is a single pooled dataset with no cell-line stratification, since this
  skill depends on per-cell-line shared test benchmarks
- Only one method or one cell line is in scope, so no cross-method/cross-cell-line
  comparison table can be constructed
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepSE
  role: Two-layer CNN model (DeepSE-specific and DeepSE-integrative variants) trained
    and evaluated per cell line for SE vs TE discrimination
  repo: https://github.com/QiaoyingJi/DeepSE
- name: Keras
  role: Implements the two-layer CNN model (model.py) whose per-cell-line metrics
    are collected
- name: dna2vec
  role: Provides the pre-trained k-mer feature embeddings used to encode SE/TE DNA
    sequences before benchmarking
  repo: https://ngpatrick.com/dna2vec/
- name: Python
  role: Runtime environment (>=3.6) for running get_datavec.py, train_specific.py,
    train_integrative1/2/3.py and LogUtils.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ygeno.2021.10.007
    title: 'DeepSE: Detecting super-enhancers among typical enhancers using only sequence
      feature embeddings'
  source_content_hash: sha256:2f015fbd9312dbaef3db453cebbd1a299e6d45ab880f3bc4bfcf77e705548a64
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-cell-line-performance-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the DeepSE two-layer CNN (trained on dna2vec sequence embeddings) against state-of-the-art SE/TE discrimination methods across the paper's nine encoded cell-line datasets, merging reproduced and reported metrics into one side-by-side comparison table.

## When to use

Use when a DNA-sequence-only super-enhancer vs typical-enhancer classifier must be evaluated per cell line and compared against reported baseline methods on shared test benchmarks.

## When NOT to use

- The task involves predicting super-enhancers with ChIP-seq-derived features rather than DNA sequence only — DeepSE is explicitly designed to avoid ChIP-seq data
- The input is a single pooled dataset with no cell-line stratification, since this skill depends on per-cell-line shared test benchmarks
- Only one method or one cell line is in scope, so no cross-method/cross-cell-line comparison table can be constructed

## Inputs

- Encoded cell-line matrices in './data/<cell_name>' (mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87, mESC_constituent) produced by get_datavec.py
- Pre-trained dna2vec k-mer embedding matrices for human (hg19) and mouse (mm9)
- Reported state-of-the-art baseline performance values for the same cell-line benchmarks (from the paper's reference results)

## Outputs

- Per-cell-line classification performance metrics for DeepSE-specific and DeepSE-integrative models
- benchmark_comparison.csv: merged side-by-side table of reproduced DeepSE metrics and reported baseline metrics

## How to apply

Load the encoded cell-line datasets (mESC, myotube, macrophage, pro-B cell, Th-cell, H2171, MM1.S, U87, and optionally mESC_constituent) from './data/<cell_name>' as produced by get_datavec.py. Train DeepSE-specific models per cell line with train_specific.py at learning_rate = 0.001 and epochs = 80, capturing training logs via LogUtils.py; then train DeepSE-integrative variants with train_integrative1.py, train_integrative2.py, and train_integrative3.py per species setting (hg or mm) using the same 0.001/80 settings. Evaluate each trained Keras two-layer CNN on the shared cell-line test sets and compile classification metrics. Merge reproduced DeepSE metrics with the reported state-of-the-art baseline values into a single side-by-side table (benchmark_comparison.csv) and verify DeepSE outperforms all baselines on every shared cell-line benchmark.

## Related tools

- **DeepSE** (Two-layer CNN model (DeepSE-specific and DeepSE-integrative variants) trained and evaluated per cell line for SE vs TE discrimination) — https://github.com/QiaoyingJi/DeepSE
- **Keras** (Implements the two-layer CNN model (model.py) whose per-cell-line metrics are collected)
- **dna2vec** (Provides the pre-trained k-mer feature embeddings used to encode SE/TE DNA sequences before benchmarking) — https://ngpatrick.com/dna2vec/
- **Python** (Runtime environment (>=3.6) for running get_datavec.py, train_specific.py, train_integrative1/2/3.py and LogUtils.py)

## Examples

```
python train_specific.py mESC 0.001 80
```

## Evaluation signals

- The compiled benchmark_comparison.csv shows DeepSE outperforming all state-of-the-art methods on each shared cell-line benchmark
- Training commands used learning_rate = 0.001 and epochs = 80, matching the paper's settings (e.g. 'python train_specific.py mESC 0.001 80')
- Training logs written via LogUtils.py show convergence for all cell-line and species (hg/mm) settings
- DeepSE-integrative models generalize to held-out cell lines and across species, consistent with the finding that DeepSE 'can be generalized well across different cell lines'
- Encoded matrices for all datasets exist under './data/<cell_name>' before training begins

## Limitations

- Only two pre-trained embedding matrices are provided (human hg19 and mouse mm9); other genomes require training a custom dna2vec model
- Baseline metrics are taken from the paper's reported reference results rather than re-run, so the comparison mixes reproduced and reported values
- Cross-species integrative models (train_integrative3.py) depend on the hg/mm embedding and dataset availability; performance on a cell line not in the nine encoded datasets is untested here

## Evidence

- [methods] Cell-line specific training command: [evidence span withheld — anchor status: not_found]
