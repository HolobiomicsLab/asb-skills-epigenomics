---
name: benchmark-results-tabulation-across-cell-lines
description: Use when comparing multiple model classes (CNN baseline, DNA foundation
  models, task-specific expert) on the same prediction task across several biological
  subsets (e.g., cell lines) and a per-subset plus averaged score table is required.
when_to_use_negative:
- The task is a short-range DNA prediction task (e.g., classification with AUROC metric)
  rather than a 2D contact map regression evaluated with stratum-adjusted correlation
  coefficient
- Only one model or one cell line is being evaluated, so a cross-model, cross-cell-line
  comparison table is not applicable
- The inputs are raw .tfr files that have not yet been run through model inference;
  tabulation requires predicted maps first
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: DNALONGBENCH
  role: Provides the load_data utility and benchmark splits for contact map prediction
    used to generate predictions per cell line before tabulation
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: HyenaDNA
  role: Fine-tuned DNA foundation model whose per-cell-line SCC scores are tabulated
    alongside the other models
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model included as a row in the SCC benchmark table
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model included as a row in the SCC benchmark table
- name: conda
  role: Creates the dnalongbench environment (Python 3.9) in which predictions and
    SCC computation are run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:c3eb627b1567ffa5ba70fdcb6f1b3369c2c1e8d2826f78b15a39333a6fa445c6
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# benchmark-results-tabulation-across-cell-lines

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulate stratum-adjusted correlation coefficient (SCC) scores for all evaluated models (expert model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) per cell line and as a cross-cell-line average, reproducing the contact map prediction benchmark table (Table 4) from DNALONGBENCH.

## When to use

Use when comparing multiple model classes (CNN baseline, DNA foundation models, task-specific expert) on the same prediction task across several biological subsets (e.g., cell lines) and a per-subset plus averaged score table is required.

## When NOT to use

- The task is a short-range DNA prediction task (e.g., classification with AUROC metric) rather than a 2D contact map regression evaluated with stratum-adjusted correlation coefficient
- Only one model or one cell line is being evaluated, so a cross-model, cross-cell-line comparison table is not applicable
- The inputs are raw .tfr files that have not yet been run through model inference; tabulation requires predicted maps first

## Inputs

- Per-cell-line predicted 2D contact maps from each model (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, Akita expert)
- True 2D contact maps from the Contact Map Prediction test splits (Harvard Dataverse DOI 10.7910/DVN/AZM25S)
- Test loaders from dnalongbench load_data (task_name='contact_map_prediction', subset per cell line, batch_size=16)

## Outputs

- Table of SCC scores: five models (Expert Model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) × nine cell types, plus cross-cell-line averages (Table 4 reproduction)
- Per-cell-line SCC values identifying the top-performing model (max SCC 0.233, Akita expert)

## How to apply

After running inference with each model (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, and the pretrained Akita expert) on the identical test split of each cell-line subset (e.g., HFF loaded via dnalongbench load_data with task_name='contact_map_prediction', batch_size=16), compute the Spearman correlation coefficient (SCC) between predicted and true 2D contact maps for every model-subset pair. Organize scores into a matrix with models as rows and cell lines as columns, then append a cross-cell-line average column for each model. Verify that the task-specific expert model attains the highest average SCC (reported max 0.233), consistent with the article's finding that expert models consistently outperform DNA foundation models, and that only moderate correlation is achieved. Finally, cross-check the tabulated values against the published table (Table 4) to confirm reproduction.

## Related tools

- **DNALONGBENCH** (Provides the load_data utility and benchmark splits for contact map prediction used to generate predictions per cell line before tabulation) — https://github.com/ma-compbio/DNALONGBENCH.git
- **HyenaDNA** (Fine-tuned DNA foundation model whose per-cell-line SCC scores are tabulated alongside the other models) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model included as a row in the SCC benchmark table)
- **Caduceus-PS** (Fine-tuned DNA foundation model included as a row in the SCC benchmark table)
- **conda** (Creates the dnalongbench environment (Python 3.9) in which predictions and SCC computation are run)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- Each model-subset cell in the table contains a valid SCC value in [−1, 1]; all nine cell lines are populated for all five models
- The expert model row has the highest cross-cell-line average SCC, matching the published finding (max 0.233)
- Tabulated averages reproduce Table 4 within numerical tolerance when re-run on the same test splits
- SCC values are computed on identical test splits for all models, confirming no data leakage across train/valid/test

## Limitations

- Performance indicates only moderate correlation (max SCC 0.233), so the table demonstrates relative ranking rather than high absolute predictive accuracy
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded from evaluation, limiting the generality of model comparisons
- Simple fine-tuning may be insufficient for foundation models to outperform specialized expert architectures, so differences in the table may reflect fine-tuning strategy rather than architecture alone

## Evidence

- [results] On the contact map prediction task, the Akita expert model achieves the highest stratum-adjusted correlation coefficient (0.233 max), outperforming all DNA foundation models and the CNN baseline: [evidence span withheld — anchor status: not_found]
- [methods] load_data invocation: "train_loader, valid_loader, test_loader = load_data(root=root, task_name = 'contact_map_prediction', subset='HFF', batch_size=16)"
- [intro] expert models outperform foundation models: "while DNA foundation models capture long-range dependencies to some extent, expert models consistently outperform them across all tasks"
- [discussion] expert models highest scores: "The benchmarking results consistently showed that expert models achieved the highest scores across all tasks."
- [readme] README task listing: "DNALongBench is a benchmark of realistic and biologically meaningful genomic DNA prediction tasks that require long-range sequence input and involve long-range dependencies."
