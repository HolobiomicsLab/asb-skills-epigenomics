---
name: spearman-correlation-computation-2d-genomic-predictions
description: Use when evaluating 2D contact map predictions (e.g., Hi-C-derived maps
  from the DNALONGBENCH contact_map_prediction task) for models such as Akita, CNN,
  HyenaDNA, or Caduceus, where rank-based correlation is required.
when_to_use_negative:
- Evaluating 1D sequence-level regression tasks (e.g., regulatory sequence activity
  prediction) that use Pearson correlation (PCC) rather than 2D contact-map SCC
- Binary classification tasks in DNALONGBENCH (enhancer-target gene, eQTL) that are
  scored with AUROC instead of SCC
- Contact maps that have not been generated on the same well-split test partitions
  — comparing models on different splits invalidates the cross-model SCC table
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DNALONGBENCH
  role: Provides the contact_map_prediction dataset, load_data utility, and evaluation
    harness for computing SCC across cell lines
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: HyenaDNA
  role: Fine-tuned DNA foundation model whose contact map predictions are scored with
    SCC
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model whose contact map predictions are scored with
    SCC
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model whose contact map predictions are scored with
    SCC
- name: conda
  role: Environment management for the dnalongbench Python 3.9 environment used for
    evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:ed7701222a0b441543241388a346b37b5dbd7d8da83bb9df4372e3f069a5b82c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/ma-compbio/DNALONGBENCH
  tools:
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# spearman-correlation-computation-2d-genomic-predictions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the stratum-adjusted correlation coefficient (SCC) between predicted and true 2D genomic contact maps, the metric used in DNALONGBENCH to score the contact map prediction task across nine human cell lines.

## When to use

Use when evaluating 2D contact map predictions (e.g., Hi-C-derived maps from the DNALONGBENCH contact_map_prediction task) for models such as Akita, CNN, HyenaDNA, or Caduceus, where rank-based correlation is required.

## When NOT to use

- Evaluating 1D sequence-level regression tasks (e.g., regulatory sequence activity prediction) that use Pearson correlation (PCC) rather than 2D contact-map SCC
- Binary classification tasks in DNALONGBENCH (enhancer-target gene, eQTL) that are scored with AUROC instead of SCC
- Contact maps that have not been generated on the same well-split test partitions — comparing models on different splits invalidates the cross-model SCC table

## Inputs

- Predicted 2D contact maps from the model under evaluation (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, or Akita expert)
- Ground-truth 2D contact maps from the DNALONGBENCH contact_map_prediction test split (Harvard Dataverse DOI 10.7910/DVN/AZM25S)
- Cell-line subset identifier (e.g., HFF) for load_data(root, task_name='contact_map_prediction', subset=..., batch_size=16)

## Outputs

- Per-model, per-cell-line SCC scores for 2D contact map predictions
- Cross-cell-line average SCC per model (Expert Model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS)
- Benchmark table matching Table 4 of the DNALONGBENCH paper

## How to apply

After training or fine-tuning models (CNN baseline, HyenaDNA, Caduceus-Ph, Caduceus-PS) and running inference with the pretrained Akita expert model, load each cell-line subset (e.g., HFF) via dnalongbench's load_data utility with task_name='contact_map_prediction' and batch_size=16 to obtain the test loader. For every model and cell line, compute the Spearman correlation coefficient (SCC) between predicted and ground-truth 2D contact maps on the same test splits, then aggregate a cross-cell-line average over all nine cell types. Tabulate SCC for the five methods and verify the ordering: the expert model should attain the highest average (max reported 0.233), consistent with Table 4 of the DNALONGBENCH paper. SCC is preferred over raw Pearson because contact maps are sparse and rank-based correlation is more robust to the distribution of contact intensities within distance strata.

## Related tools

- **DNALONGBENCH** (Provides the contact_map_prediction dataset, load_data utility, and evaluation harness for computing SCC across cell lines) — https://github.com/ma-compbio/DNALONGBENCH
- **HyenaDNA** (Fine-tuned DNA foundation model whose contact map predictions are scored with SCC) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model whose contact map predictions are scored with SCC)
- **Caduceus-PS** (Fine-tuned DNA foundation model whose contact map predictions are scored with SCC)
- **conda** (Environment management for the dnalongbench Python 3.9 environment used for evaluation)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- Akita expert model attains the highest average SCC across the nine cell lines, reproducing the ordering in Table 4
- Maximum reported SCC reaches approximately 0.233, indicating only moderate correlation is expected for this task
- SCC is computed on identical test splits (well-split train/valid/test files from DOI 10.7910/DVN/AZM25S) for all five models
- Per-cell-line SCC values are reported alongside the cross-cell-line average for all five methods

## Limitations

- SCC values on the contact map task are low (max ~0.233), indicating only moderate correlation even for the best expert model
- The study excluded transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) because quadratic self-attention cost makes long-range tasks infeasible
- Simple fine-tuning may be insufficient for foundation models to outperform specialized expert architectures on this task

## Evidence

- [results] On the contact map prediction task, the Akita expert model achieves the highest stratum-adjusted correlation coefficient (0.233 max), outperforming all DNA foundation models and the CNN baseline: [evidence span withheld — anchor status: not_found]
