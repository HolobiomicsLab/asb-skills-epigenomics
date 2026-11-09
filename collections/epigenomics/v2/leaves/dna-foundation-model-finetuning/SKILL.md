---
name: dna-foundation-model-finetuning
description: Use when evaluating how a pretrained DNA foundation model captures long-range
  genomic dependencies (up to 1,000,000 bp) on a supervised regression or classification
  task with train/valid/test splits, and you need reproducible per-model benchmark
  scores.
when_to_use_negative:
- The task input is short-range (well below the 1 million bp context DNALONGBENCH
  targets) — benchmarks limited to short-range tasks are explicitly out of scope for
  this suite
- The goal is to evaluate an expert/task-specific model (e.g. Enformer, Akita, Puffin,
  ABC) rather than fine-tune a foundation model
- You intend to use a quadratic self-attention transformer (DNABERT-1, DNABERT-2,
  Nucleotide Transformer) on ultra-long inputs — the article notes this quadratic
  cost renders such tasks infeasible for these models
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: HyenaDNA
  role: Long-range DNA foundation model (pretrained up to 1M tokens at single nucleotide
    resolution on hg38) fine-tuned on DNALONGBENCH tasks; environment built from its
    README with flash-attention
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Mamba-based DNA foundation model fine-tuned in the caduceus_env environment
    for CMP, eQTLP, ETGP, RSAP, and TISP tasks
- name: Caduceus-PS
  role: Pretrained-Caduceus variant fine-tuned alongside Caduceus-Ph on the same benchmark
    tasks
- name: conda
  role: Environment management for the dnalongbench (Python 3.9) and caduceus_env
    environments used in fine-tuning
- name: DNALONGBENCH
  role: Benchmark package providing load_data utility, task data splits, and experiment
    READMEs for fine-tuning each model
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_003
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:f8e8d39f2f3ffdb5e72af8439bbb5703fc99fbdaa7229a72dd744c683de2df80
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
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# DNA foundation model fine-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes pretrained DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) on the five DNALONGBENCH long-range genomics tasks (ETGP, eQTL, CMP, RSAP, TISP) so their performance can be compared against task-specific expert models and a CNN baseline.

## When to use

Use when evaluating how a pretrained DNA foundation model captures long-range genomic dependencies (up to 1,000,000 bp) on a supervised regression or classification task with train/valid/test splits, and you need reproducible per-model benchmark scores.

## When NOT to use

- The task input is short-range (well below the 1 million bp context DNALONGBENCH targets) — benchmarks limited to short-range tasks are explicitly out of scope for this suite
- The goal is to evaluate an expert/task-specific model (e.g. Enformer, Akita, Puffin, ABC) rather than fine-tune a foundation model
- You intend to use a quadratic self-attention transformer (DNABERT-1, DNABERT-2, Nucleotide Transformer) on ultra-long inputs — the article notes this quadratic cost renders such tasks infeasible for these models

## Inputs

- Task data as train/valid/test .tfr (TensorFlow-record) splits from Harvard Dataverse deposits (ETGP, eQTL, CMP, RSAP, TISP tasks)
- Pretrained HyenaDNA weights from HuggingFace (LongSafari, e.g. medium-450k, large-1m)
- Caduceus-Ph and Caduceus-PS checkpoints via the caduceus_env.yml environment
- dnalongbench load_data loaders with task_name, subset (e.g. 'HFF'), and batch_size=16

## Outputs

- Per-model metric tables (AUROC/AUPRC for ETGP/eQTL, SCC for CMP, PCC for RSAP/TISP) as CSV files such as eqtl_auroc_results.csv, rsap_pcc_scores.csv, tisp_pcc_results.csv, etgp_benchmark_scores.csv, caduceus_ph_cmp_scc_scores.csv
- Fine-tuned model checkpoints and inference predictions on held-out test chromosomes or splits

## How to apply

Set up the task-specific environment: create the dnalongbench conda environment with Python 3.9 and install the ma-compbio/DNALONGBENCH package with pip, then create the HyenaDNA environment from HazyResearch/hyena-dna (PyTorch 1.13, CUDA 11.7, flash-attention installed as a submodule with 'pip install -e . --no-build-isolation') and the Caduceus environment via 'conda env create -f caduceus_env.yml'. Download the task data from its Harvard Dataverse DOI (e.g. 10.7910/DVN/MNUEZR for RSAP, 10.7910/DVN/AZM25S for CMP, 10.7910/DVN/CTEQXX for ETGP), reformulate it into train*/valid*/test*.tfr files, and load loaders with dnalongbench load_data (task_name, subset, batch_size=16). Fine-tune each model following the task-specific experiment README (e.g. experiments/HyenaDNA/HyenaDNA_RSAP_TISP/README.md, experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md), choosing context length to match the task input (450,000 bp for ETGP/eQTL; 100,000 bp for TISP; for CMP ablations use 409,600/307,200/204,800 bp). Run inference on held-out test splits (e.g. chr8 and chr9 for TISP) and compute the task metric: AUROC/AUPRC for ETGP and eQTL, SCC for contact maps, nucleotide-wise or per-sample PCC for RSAP and TISP. The rationale is that DNALONGBENCH shows simple fine-tuning may not be sufficient for foundation models to outperform specialized expert architectures, so scores should be tabulated per model and checked against the paper's ordering.

## Related tools

- **HyenaDNA** (Long-range DNA foundation model (pretrained up to 1M tokens at single nucleotide resolution on hg38) fine-tuned on DNALONGBENCH tasks; environment built from its README with flash-attention) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Mamba-based DNA foundation model fine-tuned in the caduceus_env environment for CMP, eQTLP, ETGP, RSAP, and TISP tasks)
- **Caduceus-PS** (Pretrained-Caduceus variant fine-tuned alongside Caduceus-Ph on the same benchmark tasks)
- **conda** (Environment management for the dnalongbench (Python 3.9) and caduceus_env environments used in fine-tuning)
- **DNALONGBENCH** (Benchmark package providing load_data utility, task data splits, and experiment READMEs for fine-tuning each model) — https://github.com/ma-compbio/DNALONGBENCH.git

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- The output table contains a metric value for each of the five models (Expert Model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) with per-assay and average entries
- For TISP on test chromosomes chr8 and chr9, average PCC should show the expert Puffin model (0.733) well above HyenaDNA (0.132), Caduceus-Ph (0.109), and Caduceus-PS (0.108)
- For ETGP, reproduced AUROC values should match Expert Model 0.926, CNN 0.797, HyenaDNA 0.828, Caduceus-Ph 0.826, Caduceus-PS 0.821
- For the CMP context-length ablation, Caduceus-Ph SCC should increase monotonically from 204,800 bp (100 bins) to 409,600 bp (200 bins) while staying below the expert model
- Foundation-model predictions on RSAP/TISP can be inspected for underpredicted signal intensities or missed peaks, a documented failure mode of fine-tuned foundation models

## Limitations

- Expert models consistently outperformed the fine-tuned foundation models across all benchmark tasks, so fine-tuning alone is not expected to close the gap
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because the quadratic self-attention cost renders long-range tasks infeasible
- Simple fine-tuning may not be sufficient to outperform highly specialized expert architectures; improvements require novel architectures, fine-tuning strategies, and task-specific training objectives
- Model performance varies substantially across tasks, so scores on one task (e.g. eQTL AUROC ~0.68) do not transfer to others

## Evidence

- [readme] HyenaDNA setup from README: [evidence span withheld — anchor status: external_doc]
