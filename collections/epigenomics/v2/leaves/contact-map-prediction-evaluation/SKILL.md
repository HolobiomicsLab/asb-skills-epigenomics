---
name: contact-map-prediction-evaluation
description: Use when benchmarking DNA sequence models on 2D genome-organization prediction
  from long-range inputs (up to ~450 kb) and comparing them against an expert model
  with SCC on test splits of the Harvard Dataverse CMP deposit.
when_to_use_negative:
- Task output is binary classification (e.g. enhancer-target gene or eQTL, 450,000
  bp input, AUROC metric) rather than 2D contact map regression
- Sequence context is short-range; existing benchmarks (Genomic Benchmarks, NT Benchmark,
  GUE) cover short-range tasks where CMP's long-range setup is unnecessary
- No access to the CMP Dataverse deposit or the Akita expert checkpoint — the expert-model
  comparison column cannot be reproduced
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DNALONGBENCH
  role: Benchmark package providing the load_data utility for the contact_map_prediction
    task and train/valid/test .tfr splits
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: HyenaDNA
  role: DNA foundation model fine-tuned on the CMP regression task
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: DNA foundation model fine-tuned on CMP; used for the context-length ablation
    (200/150/100 bins)
- name: Caduceus-PS
  role: DNA foundation model fine-tuned on the CMP task
- name: Akita expert model
  role: Task-specific expert model run in pretrained mode on the same CMP test splits
    to provide the upper-bound SCC
- name: conda
  role: Environment manager for dnalongbench (python=3.9) and Caduceus (caduceus_env.yml)
    environments
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:c9c44d9e546711d526fe985c700d199b6be532614d0325b44505250ad6fa2e27
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
  - Akita expert model
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# contact-map-prediction-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS), a CNN baseline, and the Akita expert model on the DNALONGBENCH 2D contact map prediction (CMP) task using stratum-adjusted Spearman correlation coefficient (SCC) across human cell lines.

## When to use

Use when benchmarking DNA sequence models on 2D genome-organization prediction from long-range inputs (up to ~450 kb) and comparing them against an expert model with SCC on test splits of the Harvard Dataverse CMP deposit.

## When NOT to use

- Task output is binary classification (e.g. enhancer-target gene or eQTL, 450,000 bp input, AUROC metric) rather than 2D contact map regression
- Sequence context is short-range; existing benchmarks (Genomic Benchmarks, NT Benchmark, GUE) cover short-range tasks where CMP's long-range setup is unnecessary
- No access to the CMP Dataverse deposit or the Akita expert checkpoint — the expert-model comparison column cannot be reproduced

## Inputs

- Contact Map Prediction dataset from Harvard Dataverse DOI 10.7910/DVN/AZM25S (well-split train/valid/test files)
- Cell-line subsets (e.g. HFF) loaded via dnalongbench load_data (task_name='contact_map_prediction', batch_size=16)
- Pretrained Akita expert model checkpoint
- Caduceus conda environment defined in caduceus_env.yml (experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP)

## Outputs

- SCC scores per model and cell line on the CMP test split, plus cross-cell-line averages (Table 4 style)
- Context-length ablation table of Caduceus-Ph SCC at 204,800 / 307,200 / 409,600 bp (e.g. caduceus_ph_cmp_scc_scores.csv)
- Predicted vs. ground-truth 2D contact maps for the test split

## How to apply

Set up the dnalongbench conda environment (Python 3.9), clone ma-compbio/DNALONGBENCH, and pip install it. Download the Contact Map Prediction dataset (DOI 10.7910/DVN/AZM25S), which ships well-split train/valid/test files, and load each cell line subset (e.g. 'HFF') with the dnalongbench load_data utility using task_name='contact_map_prediction' and batch_size=16. Fine-tune the foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) and train the lightweight CNN baseline following the READMEs under experiments/CNN and the Caduceus CMP_eQTLP_ETGP folders; for context-length ablations, run Caduceus-Ph at 409,600 bp (200 bins), 307,200 bp (150 bins), and 204,800 bp (100 bins). Run the pretrained Akita expert model on the same test splits. Compute Spearman correlation coefficient (SCC) between predicted and ground-truth contact maps for every model and cell line, plus the cross-cell-line average, and tabulate results (Expert Model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS).

## Related tools

- **DNALONGBENCH** (Benchmark package providing the load_data utility for the contact_map_prediction task and train/valid/test .tfr splits) — https://github.com/ma-compbio/DNALONGBENCH.git
- **HyenaDNA** (DNA foundation model fine-tuned on the CMP regression task) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (DNA foundation model fine-tuned on CMP; used for the context-length ablation (200/150/100 bins))
- **Caduceus-PS** (DNA foundation model fine-tuned on the CMP task)
- **Akita expert model** (Task-specific expert model run in pretrained mode on the same CMP test splits to provide the upper-bound SCC)
- **conda** (Environment manager for dnalongbench (python=3.9) and Caduceus (caduceus_env.yml) environments)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- SCC values exist for all five methods (Expert Model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) for each cell line and the cross-cell-line average
- The Akita expert model attains the highest average SCC, matching Table 4 (max SCC ≈ 0.233), with all scores indicating only moderate correlation
- Context-length ablation reproduces the trend of Tables S6–S11: Caduceus-Ph SCC increases monotonically from 204,800 bp to 409,600 bp while staying below the expert model
- load_data returns three distinct loaders (train_loader, valid_loader, test_loader) for the requested subset, confirming well-split data ingestion
- Environment checks: dnalongbench env created with python=3.9; Caduceus env created from caduceus_env.yml without dependency conflicts

## Limitations

- Expert models consistently outperform DNA foundation models across all benchmark tasks, so CMP results should not be read as evidence that foundation models match specialized architectures
- Quadratic self-attention cost makes long-range tasks infeasible for transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer), which were excluded from the evaluation
- Simple fine-tuning may be insufficient for foundation models to outperform specialized expert architectures on CMP
- Overall SCC values indicate only moderate correlation, limiting absolute interpretability of any single score

## Evidence
