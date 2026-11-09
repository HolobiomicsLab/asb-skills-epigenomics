---
name: binary-classification-auroc-auprc-evaluation
description: Use when comparing binary classification models on DNALONGBENCH tasks
  such as enhancer-target gene prediction, where AUROC and AUPRC on a held-out test
  split are the benchmark metrics.
when_to_use_negative:
- The task is not binary classification — contact map prediction, regulatory sequence
  activity, and transcription initiation signals use different output shapes and metrics
  than AUROC/AUPRC
- Evaluation on short-range DNA tasks (e.g., GenomicBenchmarks, NT Benchmark, GUE)
  that the paper distinguishes from its long-range benchmark
- Comparing against transformer-based DNA foundation models (DNABERT-1, DNABERT-2,
  Nucleotide Transformer) — the paper explicitly excluded these since quadratic self-attention
  is infeasible for 450,000 bp inputs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DNALONGBENCH
  role: Provides the load_data utility to build train/valid/test loaders with the
    ETGP task name and batch_size=16, and defines AUROC as the task metric
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: HyenaDNA
  role: Fine-tuned DNA foundation model evaluated on the ETGP task, yielding AUROC/AUPRC
    for comparison
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model evaluated on the ETGP task, yielding AUROC/AUPRC
    for comparison
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model evaluated on the ETGP task, yielding AUROC/AUPRC
    for comparison
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Creates the dnalongbench (Python 3.9) and caduceus environments needed to
    run training and evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:b0e4f96a3aa42d4b94b8e3cfdc70b3d3a80606c45d7c21fcee1d79ab3d01208f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/ma-compbio/DNALONGBENCH.git
  tools:
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# binary-classification-auroc-auprc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate binary classification models on long-range DNA prediction tasks (e.g., enhancer-target gene prediction in DNALONGBENCH) by reporting held-out test AUROC and AUPRC, and compare them against the paper's reported reference scores for the expert model, CNN, and fine-tuned DNA foundation models.

## When to use

Use when comparing binary classification models on DNALONGBENCH tasks such as enhancer-target gene prediction, where AUROC and AUPRC on a held-out test split are the benchmark metrics.

## When NOT to use

- The task is not binary classification — contact map prediction, regulatory sequence activity, and transcription initiation signals use different output shapes and metrics than AUROC/AUPRC
- Evaluation on short-range DNA tasks (e.g., GenomicBenchmarks, NT Benchmark, GUE) that the paper distinguishes from its long-range benchmark
- Comparing against transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) — the paper explicitly excluded these since quadratic self-attention is infeasible for 450,000 bp inputs

## Inputs

- DNALONGBENCH enhancer-target gene prediction train/valid/test .tfr split files (download from https://doi.org/10.7910/DVN/CTEQXX)
- Trained model checkpoints or predictions on the held-out test set (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, or expert model)
- Reference reported AUROC/AUPRC values from the paper's benchmark table

## Outputs

- Per-model AUROC and AUPRC values on the held-out test split
- Compiled benchmark table (e.g., etgp_benchmark_scores.csv) comparing all five models

## How to apply

Load the task's train/valid/test splits via dnalongbench.utils.load_data with the task name and batch_size=16 (e.g., ETGP data from https://doi.org/10.7910/DVN/CTEQXX). Train the candidate model — CNN baseline, fine-tuned HyenaDNA, Caduceus-Ph, or Caduceus-PS — following the README for its experiments directory, then score all models on the same held-out test set. Compute AUROC and AUPRC as the benchmark reports them (e.g., AUROC is the metric column for the enhancer-target gene task with 450,000 bp input, 2,602 samples). Compile scores into a CSV and compare against the paper's reference row: Expert Model 0.926/0.407, CNN 0.797/0.310, HyenaDNA 0.828/0.249, Caduceus-Ph 0.826/0.380, Caduceus-PS 0.821/0.376. Correct application is judged by reproduced AUROC values matching the reported row (Expert 0.926, CNN 0.797, HyenaDNA 0.828, Caduceus-Ph 0.826, Caduceus-PS 0.821) with the expert model ranking highest.

## Related tools

- **DNALONGBENCH** (Provides the load_data utility to build train/valid/test loaders with the ETGP task name and batch_size=16, and defines AUROC as the task metric) — https://github.com/ma-compbio/DNALONGBENCH.git
- **HyenaDNA** (Fine-tuned DNA foundation model evaluated on the ETGP task, yielding AUROC/AUPRC for comparison) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model evaluated on the ETGP task, yielding AUROC/AUPRC for comparison) — https://github.com/ma-compbio/DNALONGBENCH.git
- **Caduceus-PS** (Fine-tuned DNA foundation model evaluated on the ETGP task, yielding AUROC/AUPRC for comparison) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Creates the dnalongbench (Python 3.9) and caduceus environments needed to run training and evaluation)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='enhancer_target_gene_prediction', batch_size=16)
```

## Evaluation signals

- Reproduced AUROC values match the reported ETGP row: Expert Model 0.926, CNN 0.797, HyenaDNA 0.828, Caduceus-Ph 0.826, Caduceus-PS 0.821
- Reproduced AUPRC values match: Expert Model 0.407, CNN 0.310, HyenaDNA 0.249, Caduceus-Ph 0.380, Caduceus-PS 0.376
- The expert model ranks highest on both AUROC and AUPRC among all five models evaluated
- All models are scored on the same held-out test split loaded with batch_size=16, not on train or validation data
- Scores are compiled into the expected output artifact (etgp_benchmark_scores.csv) with one row per model

## Limitations

- Current DNA foundation models still lag behind expert models in capturing long-range genomic dependencies, so foundation-model scores (e.g., HyenaDNA AUPRC 0.249) are not expected to match the expert model
- Simple fine-tuning may be insufficient for foundation models to outperform highly specialized expert architectures
- Transformer-based DNA foundation models could not be evaluated on these tasks because the quadratic cost of self-attention makes 450,000 bp inputs infeasible

## Evidence

- [readme] Show that the expert model achieves the highest AUROC: "Enhancer-target Gene                           | Binary Classification            | 450,000      | 1                                               | 2,602                   | AUROC"
