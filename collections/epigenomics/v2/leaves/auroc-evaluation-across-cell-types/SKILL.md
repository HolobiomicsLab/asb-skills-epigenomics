---
name: auroc-evaluation-across-cell-types
description: Use when benchmarking a sequence model on the DNALONGBENCH eQTL task
  and you need per-cell-type AUROC values plus an average to compare against the paper's
  reported scores (Expert Model average AUROC 0.681).
when_to_use_negative:
- The task is a regression or per-base-pair-resolution task rather than binary classification
  — DNALONGBENCH specifies AUROC only for its binary classification tasks (Enhancer-target
  Gene, eQTL)
- You need to evaluate on short-range benchmarks (Genomic Benchmarks, NT Benchmark,
  GUE) where long-range 450,000 bp input handling is unnecessary
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline evaluated per cell type on the eQTL
    test split
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline evaluated per cell type on the eQTL
    test split
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline evaluated per cell type on the eQTL
    test split
- name: CNN
  role: Lightweight convolutional baseline fine-tuned following experiments/CNN/README.md
    and evaluated on the eQTL test split
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Creates the dnalongbench (Python 3.9) and caduceus environments needed to
    run fine-tuning and evaluation
- name: dnalongbench.utils.load_data
  role: Loads the eQTL train/valid/test splits (task_name='eqtl', batch_size=16) from
    the .tfr files
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:9b238726c537f37ff3ea013129ff02e90d348bc7e4bb2ba7d371027b3edcb62f
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
  - dnalongbench.utils.load_data
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# auroc-evaluation-across-cell-types

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the eQTL binary-classification AUROC benchmark table (Table 7) from DNALONGBENCH by evaluating fine-tuned models (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) and the Expert Model (Enformer) per cell type and averaging across the nine cell types.

## When to use

Use when benchmarking a sequence model on the DNALONGBENCH eQTL task and you need per-cell-type AUROC values plus an average to compare against the paper's reported scores (Expert Model average AUROC 0.681).

## When NOT to use

- The task is a regression or per-base-pair-resolution task rather than binary classification — DNALONGBENCH specifies AUROC only for its binary classification tasks (Enhancer-target Gene, eQTL)
- You need to evaluate on short-range benchmarks (Genomic Benchmarks, NT Benchmark, GUE) where long-range 450,000 bp input handling is unnecessary

## Inputs

- eQTL task bed files (train/valid/test splits) from https://doi.org/10.7910/DVN/YUP2G5
- Fine-tuned model checkpoints (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) and the Expert Model (Enformer) predictions on the held-out test split
- Binary eQTL labels for the 31,282 samples with 450,000 bp input sequences

## Outputs

- eqtl_auroc_results.csv with per-cell-type AUROC values and the average AUROC across the nine cell types for each model
- Reproduced Table 7 AUROC benchmark table

## How to apply

Set up the dnalongbench conda environment (Python 3.9), clone https://github.com/ma-compbio/DNALONGBENCH.git and install with 'pip install .'. Download the eQTL bed data from https://doi.org/10.7910/DVN/YUP2G5 and load the train/valid/test splits with dnalongbench.utils.load_data(task_name='eqtl', batch_size=16); the eQTL task is a binary classification problem with 450,000 bp input length and 31,282 samples. Fine-tune each baseline model (CNN, HyenaDNA via HazyResearch/hyena-dna, Caduceus-Ph and Caduceus-PS via the caduceus_env.yml environment), then evaluate each fine-tuned model on the held-out test split. Compute AUROC per cell type across the nine cell types and average them, writing the results to eqtl_auroc_results.csv. Validation: the reproduced values should match the Expert Model scores 0.639, 0.689, 0.612, 0.710, 0.700, 0.621, 0.683, 0.741, 0.736 (average 0.681) and confirm the expert model attains the highest AUROC, consistent with the paper's finding that expert models consistently outperform fine-tuned DNA foundation models.

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model baseline evaluated per cell type on the eQTL test split) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline evaluated per cell type on the eQTL test split)
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline evaluated per cell type on the eQTL test split)
- **CNN** (Lightweight convolutional baseline fine-tuned following experiments/CNN/README.md and evaluated on the eQTL test split) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Creates the dnalongbench (Python 3.9) and caduceus environments needed to run fine-tuning and evaluation)
- **dnalongbench.utils.load_data** (Loads the eQTL train/valid/test splits (task_name='eqtl', batch_size=16) from the .tfr files) — https://github.com/ma-compbio/DNALONGBENCH.git

## Examples

```
train_loader, valid_loader, test_loader = dnalongbench.utils.load_data(root='data', task_name='eqtl', batch_size=16)
```

## Evaluation signals

- Per-cell-type AUROC values fall in the 0.6–0.75 range and the Expert Model average equals 0.681 across the nine cell types
- The Expert Model (Enformer) attains the highest AUROC on every one of the nine cell types, matching the paper's claim
- The test split is the held-out test_loader from dnalongbench.utils.load_data, not the train or valid loaders
- Results file eqtl_auroc_results.csv contains nine per-cell-type rows plus an average per model, matching the Table 7 layout

## Limitations

- Simple fine-tuning may not be sufficient for DNA foundation models to outperform the highly specialized expert architectures on this task
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded from the benchmark because quadratic self-attention cost renders 450,000 bp long-range tasks infeasible
- AUROC on the eQTL task is modest (average 0.681 for the best model), reflecting the difficulty of long-range genomic dependency prediction

## Evidence

- [methods] Expert Model per-cell-type AUROC scores: [evidence span withheld — anchor status: not_found]
- [methods] data loading API: "train_loader, valid_loader, test_loader = load_data(root=root, task_name = 'contact_map_prediction', subset='HFF', batch_size=16)"
- [readme] task specification from README: "eQTL                           | Binary Classification            | 450,000      | 1                                               | 31,282                  | AUROC"
- [abstract] headline finding: "showing that current foundation models still lag behind expert models in capturing long-range genomic dependencies"
- [discussion] discussion caveat: "Consequently, simple fine-tuning may not be sufficient to outperform these highly specialized expert architectures."
