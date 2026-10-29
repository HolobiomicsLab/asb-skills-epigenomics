---
name: long-range-genomic-dependency-benchmarking
description: Use when comparing fine-tuned DNA foundation models against task-specific
  expert architectures on genomic prediction tasks whose input sequences span long
  ranges (up to 1,000,000 bp), e.g. eQTL binary classification over 450,000-bp inputs.
when_to_use_negative:
- The task involves only short-range sequences — existing benchmarks (Genomic Benchmarks,
  NT Benchmark, GUE) already cover short-range tasks, and DNALONGBENCH targets long-range
  dependencies up to 1 million bp
- You need to evaluate transformer-based DNA foundation models (DNABERT-1, DNABERT-2,
  Nucleotide Transformer) — the quadratic self-attention cost renders these long-range
  tasks infeasible for such models
- You only need pretrained-model inference or embeddings without fine-tuning and test-split
  metric evaluation against expert-model baselines
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline evaluated on the eQTL long-range
    task
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline (Mamba-based) evaluated on DNALONGBENCH
    tasks
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline (Mamba-based) evaluated on DNALONGBENCH
    tasks
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: CNN
  role: Lightweight convolutional baseline fine-tuned via experiments/CNN/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Environment management for the dnalongbench (Python 3.9) and caduceus_env
    environments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:144fc9e9c49ed5aea0bfb2e9fcc4261cf5925ca7d734c06c6b1441c6a209bbdb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/HazyResearch/hyena-dna
  tools:
  - CNN
  - Caduceus-PS
  - Caduceus-Ph
  - HyenaDNA
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# long-range-genomic-dependency-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) against a task-specific expert model (Enformer) and a CNN baseline on DNALONGBENCH, a suite of five genomics tasks with long-range dependencies up to 1 million bp, using AUROC (eQTL, enhancer-target) and task-appropriate metrics.

## When to use

Use when comparing fine-tuned DNA foundation models against task-specific expert architectures on genomic prediction tasks whose input sequences span long ranges (up to 1,000,000 bp), e.g. eQTL binary classification over 450,000-bp inputs.

## When NOT to use

- The task involves only short-range sequences — existing benchmarks (Genomic Benchmarks, NT Benchmark, GUE) already cover short-range tasks, and DNALONGBENCH targets long-range dependencies up to 1 million bp
- You need to evaluate transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) — the quadratic self-attention cost renders these long-range tasks infeasible for such models
- You only need pretrained-model inference or embeddings without fine-tuning and test-split metric evaluation against expert-model baselines

## Inputs

- DNALONGBENCH task data (bed files, e.g. eQTL from https://doi.org/10.7910/DVN/YUP2G5) reformulated as train*/valid*/test*.tfr files
- Pretrained HyenaDNA checkpoints from HuggingFace (LongSafari, e.g. medium-450k for 450k-bp inputs)
- Caduceus-Ph and Caduceus-PS model configurations via caduceus_env.yml
- DNALONGBENCH repository with experiments/ READMEs (CNN, HyenaDNA, Caduceus)

## Outputs

- Per-cell-type AUROC scores on the held-out test split for each model (e.g. Expert Model: 0.639, 0.689, 0.612, 0.710, 0.700, 0.621, 0.683, 0.741, 0.736 across nine cell types)
- Average AUROC per model across cell types (eqtl_auroc_results.csv), reproducing Table 7

## How to apply

Create the dnalongbench conda environment (Python 3.9), clone https://github.com/ma-compbio/DNALONGBENCH.git and install with 'pip install .'. Download task data (e.g. eQTL bed files from https://doi.org/10.7910/DVN/YUP2G5), reformulate into train*/valid*/test*.tfr files, and load splits via dnalongbench.utils.load_data (e.g. task_name='eqtl', batch_size=16). Fine-tune each candidate model — the CNN baseline per experiments/CNN/README.md, HyenaDNA per the official HazyResearch/hyena-dna environment, and Caduceus-Ph/Caduceus-PS inside the caduceus_env created from caduceus_env.yml. Evaluate every model on the held-out test split computing the task metric (AUROC for eQTL, which spans 31,282 samples across nine cell types), then tabulate per-cell-type scores plus the average (eqtl_auroc_results.csv). The rationale: DNALONGBENCH was built because existing benchmarks lack ultra-long-range tasks, and expert models like Enformer are expected to outperform foundation models on these dependencies.

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model baseline evaluated on the eQTL long-range task) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline (Mamba-based) evaluated on DNALONGBENCH tasks) — https://github.com/ma-compbio/DNALONGBENCH.git
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline (Mamba-based) evaluated on DNALONGBENCH tasks) — https://github.com/ma-compbio/DNALONGBENCH.git
- **CNN** (Lightweight convolutional baseline fine-tuned via experiments/CNN/README.md) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Environment management for the dnalongbench (Python 3.9) and caduceus_env environments)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='eqtl', batch_size=16)
```

## Evaluation signals

- Reproduced AUROC values should match Table 7: Expert Model averages 0.681 across nine eQTL cell types and attains the highest AUROC on all of them
- The test split is loaded from test*.tfr files produced from the original bed data (31,282 eQTL samples, 450,000-bp input length, binary labels)
- eqtl_auroc_results.csv contains one AUROC per cell type plus the cross-cell-type average for each of the five evaluated methods
- Expert Model (Enformer) must outperform the CNN and all three fine-tuned foundation models, consistent with the paper's claim that expert models consistently outperform them

## Limitations

- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because quadratic self-attention cost makes ultra-long-range tasks infeasible
- Simple fine-tuning may be insufficient for foundation models to outperform highly specialized expert architectures on long-range tasks
- Model performance varies substantially across tasks, so results on one DNALONGBENCH task (e.g. eQTL) do not transfer to the other four tasks

## Evidence
