---
name: long-range-genomic-sequence-modeling
description: Use when evaluating or comparing sequence models on genomic prediction
  tasks with long-range dependencies (e.g., enhancer-target gene prediction, eQTL,
  contact map), or when deciding whether a DNA foundation model can replace a task-specific
  expert model.
when_to_use_negative:
- Your task involves only short-range sequences or base-pair-resolution without long-range
  dependencies — other benchmarks (Genomic Benchmarks, NT Benchmark, GUE) cover short-range
  tasks
- You need transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide
  Transformer) — these were excluded from DNALONGBENCH evaluation because quadratic
  self-attention cost renders 450k–1M bp tasks infeasible
- You require structure-related tasks needing ultra-long sequences beyond the five
  DNALONGBENCH tasks — the suite covers only enhancer-target gene, eQTL, 3D genome
  organization, regulatory sequence activity, and transcription initiation signals
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: DNALONGBENCH
  role: Benchmark suite providing the five long-range tasks, .tfr data loaders, and
    per-experiment training READMEs
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: HyenaDNA
  role: Fine-tuned long-range DNA foundation model baseline (up to 1M tokens at single
    nucleotide resolution)
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline evaluated on the ETGP and eQTL tasks
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline evaluated on the ETGP and eQTL tasks
- name: conda
  role: 'Environment management: dnalongbench env (Python 3.9) and caduceus env from
    caduceus_env.yml'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:ca7de6b59532082b9a1425b69c11fc7f51e7dcb4da2e63c0c0ebada15dd6a6d9
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# long-range-genomic-sequence-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) against a CNN baseline and a task-specific expert model on DNALONGBENCH long-range DNA prediction tasks (e.g., enhancer-target gene prediction on 450,000 bp inputs) using AUROC and AUPRC. Rationale: quantify whether foundation models can capture long-range genomic dependencies up to 1 million base pairs.

## When to use

Use when evaluating or comparing sequence models on genomic prediction tasks with long-range dependencies (e.g., enhancer-target gene prediction, eQTL, contact map), or when deciding whether a DNA foundation model can replace a task-specific expert model.

## When NOT to use

- Your task involves only short-range sequences or base-pair-resolution without long-range dependencies — other benchmarks (Genomic Benchmarks, NT Benchmark, GUE) cover short-range tasks
- You need transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) — these were excluded from DNALONGBENCH evaluation because quadratic self-attention cost renders 450k–1M bp tasks infeasible
- You require structure-related tasks needing ultra-long sequences beyond the five DNALONGBENCH tasks — the suite covers only enhancer-target gene, eQTL, 3D genome organization, regulatory sequence activity, and transcription initiation signals

## Inputs

- DNALONGBENCH task data split into train*/valid*/test*.tfr files (e.g., Enhancer-Target Gene Prediction from https://doi.org/10.7910/DVN/CTEQXX)
- Pretrained DNA foundation model checkpoints (HyenaDNA e.g. medium-450k from HuggingFace; Caduceus-Ph; Caduceus-PS)
- DNALONGBENCH repository (ma-compbio/DNALONGBENCH) with per-task experiment READMEs

## Outputs

- Test-set AUROC and AUPRC per model (expert, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS), e.g. compiled into etgp_benchmark_scores.csv
- Fine-tuned foundation model checkpoints per task

## How to apply

Create the dnalongbench conda environment (Python 3.9), clone ma-compbio/DNALONGBENCH and install with pip, then create the Caduceus environment from caduceus_env.yml. Download the task data (e.g., Enhancer-Target Gene Prediction from https://doi.org/10.7910/DVN/CTEQXX), reformulate it into train*/valid*/test*.tfr files, and load splits via dnalongbench.utils.load_data with the task name and batch_size=16. Train the lightweight CNN baseline and the task-specific expert model, then fine-tune HyenaDNA, Caduceus-Ph, and Caduceus-PS following the per-experiment READMEs. Evaluate all five models on the held-out test set, recording AUROC (for binary classification tasks like enhancer-target gene prediction) and AUPRC. Decision point: expect expert models to rank highest; treat any foundation model surpassing the expert as an anomaly requiring verification, since the benchmark shows foundation models still lag behind expert models on long-range dependencies.

## Related tools

- **DNALONGBENCH** (Benchmark suite providing the five long-range tasks, .tfr data loaders, and per-experiment training READMEs) — https://github.com/ma-compbio/DNALONGBENCH.git
- **HyenaDNA** (Fine-tuned long-range DNA foundation model baseline (up to 1M tokens at single nucleotide resolution)) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline evaluated on the ETGP and eQTL tasks)
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline evaluated on the ETGP and eQTL tasks)
- **conda** (Environment management: dnalongbench env (Python 3.9) and caduceus env from caduceus_env.yml)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='enhancer_target_gene_prediction', batch_size=16)
```

## Evaluation signals

- Reproduced ETGP test scores match the reported row: Expert Model AUROC 0.926 / AUPRC 0.407, CNN 0.797/0.310, HyenaDNA 0.828/0.249, Caduceus-Ph 0.826/0.380, Caduceus-PS 0.821/0.376
- The expert model ranks highest on AUROC and AUPRC for the task; any foundation model outranking it signals a pipeline error
- Data loads correctly as train/valid/test .tfr splits via dnalongbench.utils.load_data with the task name and batch_size=16 (2,602 samples for enhancer-target gene)
- Metrics are computed on the held-out test set, not validation data
- Caduceus-Ph/PS environments built from caduceus_env.yml run to completion with flash-attention-compatible dependencies

## Limitations

- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because the quadratic cost of self-attention renders such tasks infeasible
- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures
- Substantial room remains to improve foundation models through novel architectural designs, advanced fine-tuning strategies, and task-specific training objectives

## Evidence
