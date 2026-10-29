---
name: finetuned-foundation-model-inference
description: Use when evaluating whether pre-trained DNA foundation models (HyenaDNA,
  Caduceus variants) capture long-range genomic dependencies on tasks like contact
  map prediction, eQTL, enhancer-target gene, or regulatory sequence activity prediction.
when_to_use_negative:
- The task is short-range (well under 450,000 bp input length) or lacks long-range
  dependencies — DNALONGBENCH targets dependencies up to 1 million base pairs and
  existing short-range benchmarks (Genomic Benchmarks, GUE, BEND) suffice
- A transformer-based DNA foundation model (DNABERT-1, DNABERT-2, Nucleotide Transformer)
  is the intended model — the quadratic cost of self-attention renders ultra-long-range
  tasks like contact map prediction infeasible for these models
- You need a production-grade 3D genome predictor rather than a benchmark comparison
  — the task-specific expert model (e.g., Akita) is the stronger choice and foundation
  models are expected to lag behind it
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model whose inference is run on the held-out test
    split following experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model; inference follows experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md
- name: Akita
  role: Task-specific expert model for 3D contact map prediction, run in parallel
    as the reference baseline that foundation models are compared against
- name: conda
  role: Environment manager used to create the dnalongbench Python 3.9 environment
    and Caduceus environment from caduceus_env.yml
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:121914c723110706d9af435d60094b34e3b142357ef52294a28886804c8f837b
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
  - Akita
  - Caduceus-Ph
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# finetuned-foundation-model-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) on DNALONGBENCH long-range prediction tasks and compare their predictions against task-specific expert models to quantify how well foundation models capture long-range genomic dependencies.

## When to use

Use when evaluating whether pre-trained DNA foundation models (HyenaDNA, Caduceus variants) capture long-range genomic dependencies on tasks like contact map prediction, eQTL, enhancer-target gene, or regulatory sequence activity prediction.

## When NOT to use

- The task is short-range (well under 450,000 bp input length) or lacks long-range dependencies — DNALONGBENCH targets dependencies up to 1 million base pairs and existing short-range benchmarks (Genomic Benchmarks, GUE, BEND) suffice
- A transformer-based DNA foundation model (DNABERT-1, DNABERT-2, Nucleotide Transformer) is the intended model — the quadratic cost of self-attention renders ultra-long-range tasks like contact map prediction infeasible for these models
- You need a production-grade 3D genome predictor rather than a benchmark comparison — the task-specific expert model (e.g., Akita) is the stronger choice and foundation models are expected to lag behind it

## Inputs

- DNALONGBENCH task datasets (e.g., Contact Map Prediction from https://doi.org/10.7910/DVN/AZM25S) reformulated as train*/valid*/test*.tfr files
- Fine-tuned model checkpoints for HyenaDNA, Caduceus-Ph, or Caduceus-PS
- Held-out test split loaded via load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
- Ground-truth contact maps for specific genomic regions (e.g., chr6:145,205,248-145,614,848 and chr3:139,341,824-139,751,424)

## Outputs

- Model predictions for each held-out test region (2D contact maps for CMP, binary scores for classification tasks)
- PCC/SCC correlation values comparing each model's predicted map to the observed map
- Comparison figures (e.g., figure_2_contact_maps.png) annotating per-model PCC/SCC beneath each panel

## How to apply

Set up the dnalongbench conda environment (Python 3.9) by cloning ma-compbio/DNALONGBENCH and installing it with pip, then download the task dataset (e.g., Contact Map Prediction from https://doi.org/10.7910/DVN/AZM25S) and load the held-out test split via dnalongbench.utils.load_data with task_name='contact_map_prediction' and subset='HFF'. Run inference with the fine-tuned foundation model checkpoints by following the per-model experiment READMEs (e.g., experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md for HyenaDNA and experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md for Caduceus-Ph), and in parallel run the task-specific expert model (e.g., Akita for contact maps). Score predictions against ground truth with task-appropriate metrics — Pearson (PCC) and Spearman (SCC) correlations for 2D contact maps, AUROC for binary classification tasks like enhancer-target gene and eQTL. Interpret results relative to the benchmark's central finding that expert models consistently outperform the fine-tuned foundation models, so foundation-model scores below the expert baseline are the expected outcome rather than a pipeline failure.

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model whose inference is run on the held-out test split following experiments/HyenaDNA_ETGP_CMP_eQTLP/README.md) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model; inference follows experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md)
- **Akita** (Task-specific expert model for 3D contact map prediction, run in parallel as the reference baseline that foundation models are compared against)
- **conda** (Environment manager used to create the dnalongbench Python 3.9 environment and Caduceus environment from caduceus_env.yml)

## Examples

```
python -c "from dnalongbench.utils import load_data; train_loader, valid_loader, test_loader = load_data(root='data', task_name='contact_map_prediction', subset='HFF', batch_size=16)"
```

## Evaluation signals

- The loaded test split comes from load_data with task_name='contact_map_prediction' and subset='HFF', and the ground-truth maps correspond to the reported regions chr6:145,205,248-145,614,848 and chr3:139,341,824-139,751,424
- PCC and SCC are computed between each model's predicted map and the observed map for every region, yielding one score pair per model per region
- The expert model (Akita) attains the highest contact-map agreement scores across the evaluated models — this qualitative ordering should be reproduced
- Foundation-model scores below the expert baseline are expected and consistent with the paper's finding that 'expert models consistently outperform them across all tasks'
- The rendered figure (figure_2_contact_maps.png) shows four panels with PCC/SCC values annotated beneath each panel

## Limitations

- Current DNA foundation models still lag behind expert models in capturing long-range genomic dependencies; simple fine-tuning may not be sufficient to outperform highly specialized expert architectures
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded from the evaluation because quadratic self-attention cost makes ultra-long-range tasks infeasible
- Model performance varies substantially across the five tasks, so results on one task (e.g., contact map prediction) do not transfer to others (e.g., eQTL AUROC)

## Evidence
