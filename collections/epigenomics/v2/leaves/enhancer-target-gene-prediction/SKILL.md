---
name: enhancer-target-gene-prediction
description: Use when benchmarking whether DNA foundation models capture enhancer-gene
  regulatory interactions over very long sequences, or when reproducing the DNALONGBENCH
  ETGP AUROC/AUPRC table.
when_to_use_negative:
- The task is a different DNALONGBENCH task (eQTL, 3D contact map prediction, regulatory
  sequence activity prediction, transcription initiation signals) — use that task's
  specific workflow and metrics instead.
- The question concerns short-range DNA tasks (< 450 kb context) or transformer-based
  DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer), which were
  excluded here because quadratic self-attention makes long-range tasks infeasible.
- Only a foundation model is needed without the expert/CNN baselines for comparison
  — the paper's key finding requires all five models to be reproduced on the same
  held-out test set.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DNALONGBENCH
  role: Benchmark suite providing the ETGP task, load_data utility, train/valid/test
    .tfr splits, and expert model; environments and experiment configs live in the
    repo
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Creates the dnalongbench environment (Python 3.9) and the Caduceus environment
    from caduceus_env.yml
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline evaluated on the ETGP task following
    experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline evaluated on ETGP following experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline evaluated on ETGP following experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: CNN baseline
  role: Lightweight convolutional baseline trained and evaluated per experiments/CNN/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:5223a1574e5cff4bfd3f2d714fe656c167f996d62dda5bf8e2e659916e13e488
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
  - CNN baseline
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# enhancer-target-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the enhancer-target gene prediction (ETGP) benchmark from DNALONGBENCH: a binary classification task over 450,000 bp input sequences (2,602 samples, AUROC metric) evaluated across a task-specific expert model, a CNN baseline, and three fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS).

## When to use

Use when benchmarking whether DNA foundation models capture enhancer-gene regulatory interactions over very long sequences, or when reproducing the DNALONGBENCH ETGP AUROC/AUPRC table.

## When NOT to use

- The task is a different DNALONGBENCH task (eQTL, 3D contact map prediction, regulatory sequence activity prediction, transcription initiation signals) — use that task's specific workflow and metrics instead.
- The question concerns short-range DNA tasks (< 450 kb context) or transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer), which were excluded here because quadratic self-attention makes long-range tasks infeasible.
- Only a foundation model is needed without the expert/CNN baselines for comparison — the paper's key finding requires all five models to be reproduced on the same held-out test set.

## Inputs

- Enhancer-Target Gene Prediction dataset from https://doi.org/10.7910/DVN/CTEQXX (450,000 bp input sequences, binary labels, 2,602 samples)
- train/valid/test .tfr (TensorFlow Record) split files
- ma-compbio/DNALONGBENCH repository with experiment configs for CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, and the expert model

## Outputs

- etgp_benchmark_scores.csv with AUROC and AUPRC per model (Expert 0.926/0.407, CNN 0.797/0.310, HyenaDNA 0.828/0.249, Caduceus-Ph 0.826/0.380, Caduceus-PS 0.821/0.376)
- Trained ETGP model checkpoints for each of the five evaluated models

## How to apply

Create the dnalongbench conda environment (Python 3.9) and clone/install ma-compbio/DNALONGBENCH with pip, then create the Caduceus environment from caduceus_env.yml. Download the Enhancer-Target Gene Prediction data (https://doi.org/10.7910/DVN/CTEQXX), reformulate it into train*/valid*/test*.tfr files, and load splits via dnalongbench.utils.load_data with the ETGP task name and batch_size=16. Train and evaluate five models on the held-out test set: the lightweight CNN baseline (experiments/CNN/README.md), HyenaDNA (experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md), Caduceus-Ph and Caduceus-PS (experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md), and the ETGP expert model. Record AUROC and AUPRC for each and compile into etgp_benchmark_scores.csv; the expert model should rank highest (AUROC 0.926, AUPRC 0.407), consistent with the paper's finding that foundation models lag behind expert models on long-range genomic dependencies.

## Related tools

- **DNALONGBENCH** (Benchmark suite providing the ETGP task, load_data utility, train/valid/test .tfr splits, and expert model; environments and experiment configs live in the repo) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Creates the dnalongbench environment (Python 3.9) and the Caduceus environment from caduceus_env.yml)
- **HyenaDNA** (Fine-tuned DNA foundation model baseline evaluated on the ETGP task following experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline evaluated on ETGP following experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md) — https://github.com/ma-compbio/DNALONGBENCH.git
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline evaluated on ETGP following experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md) — https://github.com/ma-compbio/DNALONGBENCH.git
- **CNN baseline** (Lightweight convolutional baseline trained and evaluated per experiments/CNN/README.md) — https://github.com/ma-compbio/DNALONGBENCH.git

## Examples

```
from dnalongbench.utils import load_data; train_loader, valid_loader, test_loader = load_data(root='data', task_name='enhancer_target_gene_prediction', batch_size=16)
```

## Evaluation signals

- Reproduced AUROC on the held-out test set matches the reported ETGP row: Expert Model 0.926, CNN 0.797, HyenaDNA 0.828, Caduceus-Ph 0.826, Caduceus-PS 0.821.
- Reproduced AUPRC matches: Expert 0.407, CNN 0.310, HyenaDNA 0.249, Caduceus-Ph 0.380, Caduceus-PS 0.376 (AUPRC well below AUROC, indicating class imbalance).
- The expert model ranks highest on both AUROC and AUPRC, consistent with the paper's claim that expert models consistently outperform foundation models.
- Splits are loaded via dnalongbench.utils.load_data with the ETGP task name and batch_size=16, using train/valid/test .tfr files rather than ad-hoc splits.
- etgp_benchmark_scores.csv contains exactly five rows (one per model) with both AUROC and AUPRC columns populated.

## Limitations

- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because the quadratic cost of self-attention renders 450 kb tasks infeasible.
- Simple fine-tuning may be insufficient for foundation models to outperform highly specialized expert architectures, so benchmark scores should not be read as an upper bound on foundation model capability.
- The ETGP task has only 2,602 samples, the smallest DNALONGBENCH task, and model performance varies substantially across tasks, so ETGP results may not generalize to the other four tasks.

## Evidence
