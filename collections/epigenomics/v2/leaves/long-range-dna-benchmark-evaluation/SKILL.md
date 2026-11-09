---
name: long-range-dna-benchmark-evaluation
description: Use when comparing fine-tuned DNA foundation models against task-specific
  expert models on genomic prediction tasks with long-range dependencies (e.g., TISP,
  contact map prediction, eQTL, enhancer-target gene, regulatory sequence activity),
  or when reproducing PCC/SCC/AUROC benchmark tables.
when_to_use_negative:
- The task involves only short-range DNA sequences without long-range dependencies
  — the benchmark's motivation is that existing benchmarks lack ultra-long tasks like
  contact map prediction
- Transformer-based DNA foundation models such as DNABERT-1/2 or Nucleotide Transformer
  are required, since quadratic self-attention makes these tasks infeasible for them
  and they were excluded from the benchmark
- Only one model is being evaluated with no baseline comparison — the protocol is
  designed for cross-model benchmark comparison against an expert model
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model evaluated on TISP and other long-range tasks
  repo: https://github.com/HazyResearch/hyena-dna?tab=readme-ov-file#dependencies
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model evaluated at multiple context lengths (204,800–409,600
    bp)
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model evaluated on TISP and other long-range tasks
- name: conda
  role: 'Environment management: dnalongbench env with Python 3.9 and caduceus_env
    from caduceus_env.yml'
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:0a20ac3ff4e19bf614dac124cbadc71156aa2f5c9124698cfcd737afe87500d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/HazyResearch/hyena-dna?tab=readme-ov-file#dependencies
  tools:
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# long-range-dna-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A standardized evaluation protocol from DNALONGBENCH for benchmarking DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) against a CNN baseline and task-specific expert models on five long-range genomic prediction tasks with inputs up to 1,000,000 bp.

## When to use

Use when comparing fine-tuned DNA foundation models against task-specific expert models on genomic prediction tasks with long-range dependencies (e.g., TISP, contact map prediction, eQTL, enhancer-target gene, regulatory sequence activity), or when reproducing PCC/SCC/AUROC benchmark tables.

## When NOT to use

- The task involves only short-range DNA sequences without long-range dependencies — the benchmark's motivation is that existing benchmarks lack ultra-long tasks like contact map prediction
- Transformer-based DNA foundation models such as DNABERT-1/2 or Nucleotide Transformer are required, since quadratic self-attention makes these tasks infeasible for them and they were excluded from the benchmark
- Only one model is being evaluated with no baseline comparison — the protocol is designed for cross-model benchmark comparison against an expert model

## Inputs

- DNALONGBENCH task data (.bed / .tfr train*/valid*/test* files) from Dataverse deposits, e.g. doi:10.7910/DVN/VXQKWO (TISP) or doi:10.7910/DVN/AZM25S (contact map prediction)
- Pretrained DNA foundation model checkpoints (HyenaDNA variants from LongSafari on Hugging Face; Caduceus-Ph, Caduceus-PS)
- Task-specific expert model checkpoints (e.g., Puffin for transcription initiation signal prediction)
- Conda environment specifications (caduceus_env.yml) and task experiment configs from experiments/*/README.md

## Outputs

- Per-model benchmark table (e.g., tisp_pcc_results.csv or caduceus_ph_cmp_scc_scores.csv) with per-assay and averaged metric values
- Held-out test-split prediction scores: nucleotide-wise PCC for TISP, SCC for contact map prediction, AUROC for binary classification tasks
- Trained/fine-tuned model checkpoints for CNN, HyenaDNA, Caduceus-Ph, and Caduceus-PS on the chosen task

## How to apply

Create the dnalongbench conda environment with Python 3.9, clone ma-compbio/DNALONGBENCH, and pip install it. Download the task-specific data from the provided Dataverse DOIs (e.g., doi:10.7910/DVN/VXQKWO for TISP, doi:10.7910/DVN/AZM25S for contact map prediction) and load train/valid/test splits via the dnalongbench load_data utility (e.g., task_name='contact_map_prediction', subset='HFF', batch_size=16). Fine-tune each foundation model in its own environment — HyenaDNA from HazyResearch/hyena-dna, Caduceus-Ph/PS in the caduceus_env created from caduceus_env.yml — plus a CNN baseline per experiments/README.md. Compute the task-appropriate metric on held-out test chromosomes (e.g., nucleotide-wise Pearson correlation for TISP on chr8/chr9, Spearman correlation coefficient for contact maps, AUROC for binary classification tasks like eQTL with 450,000-bp inputs). Aggregate per-assay scores into a per-model comparison table and verify that the expert model's score exceeds all foundation models, since DNALONGBENCH's core finding is that foundation models lag behind expert models on long-range dependencies.

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model evaluated on TISP and other long-range tasks) — https://github.com/HazyResearch/hyena-dna?tab=readme-ov-file#dependencies
- **Caduceus-Ph** (Fine-tuned DNA foundation model evaluated at multiple context lengths (204,800–409,600 bp))
- **Caduceus-PS** (Fine-tuned DNA foundation model evaluated on TISP and other long-range tasks)
- **conda** (Environment management: dnalongbench env with Python 3.9 and caduceus_env from caduceus_env.yml)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- The output table contains a metric value for each of the five evaluated methods (expert model, CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS)
- For TISP, the expert model Puffin's average PCC (~0.733) exceeds CNN (0.042), HyenaDNA (0.132), Caduceus-Ph (0.109), and Caduceus-PS (0.108)
- For the Caduceus-Ph contact map ablation, SCC increases monotonically as input context grows from 204,800 to 409,600 bp (200 vs 100 bins), while remaining below the expert model's score
- Held-out evaluation is performed on test chromosomes chr8 and chr9 for TISP, not train/valid splits
- Data loading returns train_loader, valid_loader, and test_loader objects from the load_data utility with the correct task_name and subset

## Limitations

- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because the quadratic cost of self-attention renders long-range tasks infeasible for them
- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures
- Model performance varies substantially across the five tasks, so results on one task (e.g., TISP) do not transfer to others (e.g., enhancer-target gene prediction)

## Evidence
