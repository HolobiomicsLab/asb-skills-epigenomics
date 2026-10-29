---
name: dna-contact-map-prediction-inference
description: Use when evaluating whether DNA foundation models capture long-range
  genomic dependencies via 3D contact map prediction, e.g., reproducing the Figure
  2 case-study comparison of HyenaDNA, Caduceus-Ph, and Akita on chr6/chr3 HFF regions.
when_to_use_negative:
- The task is short-range DNA prediction (e.g., classification of short regulatory
  elements) rather than a 2D structure-related task requiring ultra-long sequences
  like contact map prediction or enhancer-target gene prediction
- You intend to evaluate transformer-based DNA foundation models (DNABERT-1, DNABERT-2,
  Nucleotide Transformer), whose quadratic self-attention cost makes these million-bp
  long-range tasks infeasible
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model whose contact map predictions are compared
    against ground truth on the HFF test regions
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model evaluated on contact map prediction via experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP
    README workflow
- name: Akita
  role: Task-specific expert model for 3D genome organization serving as the reference
    against which foundation models are judged
- name: conda
  role: Environment manager used to create the dnalongbench Python 3.9 environment
    and caduceus_env.yml environment for inference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:8d4dc273b7cf45d47151c1f82aaeb2e65dfe9e09246c25fd3702791cd6129e1a
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dna-contact-map-prediction-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run contact map prediction inference from the DNALONGBENCH benchmark by fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph) and the Akita expert model on held-out HFF test regions, then score agreement with observed 3D genome contact maps using PCC and SCC to assess long-range genomic dependency capture.

## When to use

Use when evaluating whether DNA foundation models capture long-range genomic dependencies via 3D contact map prediction, e.g., reproducing the Figure 2 case-study comparison of HyenaDNA, Caduceus-Ph, and Akita on chr6/chr3 HFF regions.

## When NOT to use

- The task is short-range DNA prediction (e.g., classification of short regulatory elements) rather than a 2D structure-related task requiring ultra-long sequences like contact map prediction or enhancer-target gene prediction
- You intend to evaluate transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer), whose quadratic self-attention cost makes these million-bp long-range tasks infeasible

## Inputs

- Contact Map Prediction dataset from doi:10.7910/DVN/AZM25S (train/valid/test .tfr files, HFF subset, ~1 Mb input sequences)
- Ground-truth 3D genome contact maps for genomic regions chr6:145,205,248-145,614,848 and chr3:139,341,824-139,751,424
- Pretrained/fine-tuned checkpoints for HyenaDNA, Caduceus-Ph, and the Akita expert model

## Outputs

- Predicted 2D contact maps for each model on the two case-study regions
- Per-model, per-region Pearson (PCC) and Spearman (SCC) correlation scores against the observed maps
- Four-panel comparison figure saved as figure_2_contact_maps.png with PCC/SCC values annotated beneath each panel

## How to apply

Create the dnalongbench conda environment with Python 3.9, clone ma-compbio/DNALONGBENCH, and install it with pip. Download the Contact Map Prediction dataset (doi:10.7910/DVN/AZM25S), which stores reformulated train*/valid*/test* .tfr files, and load the held-out test split via dnalongbench.utils.load_data with task_name='contact_map_prediction' and subset='HFF' (batch_size=16 in the reference call). Extract ground-truth contact maps for the two case-study regions (chr6:145,205,248-145,614,848 and chr3:139,341,824-139,751,424), then run inference with HyenaDNA and Caduceus-Ph following the experiments/ READMEs in the DNALONGBENCH repository and with the Akita expert model. Compute Pearson (PCC) and Spearman (SCC) correlations between each predicted map and the observed map per region, and render the four-panel comparison figure with PCC/SCC values beneath each panel using matplotlib.

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model whose contact map predictions are compared against ground truth on the HFF test regions) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model evaluated on contact map prediction via experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP README workflow)
- **Akita** (Task-specific expert model for 3D genome organization serving as the reference against which foundation models are judged)
- **conda** (Environment manager used to create the dnalongbench Python 3.9 environment and caduceus_env.yml environment for inference)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- load_data(root, task_name='contact_map_prediction', subset='HFF') returns non-empty train/valid/test loaders containing .tfr test records
- PCC and SCC are computed between each predicted contact map and the observed map for both case-study regions (chr6 and chr3)
- Validation: the produced figure reproduces the reported qualitative result that the expert model attains the highest contact-map agreement scores across the evaluated models
- The four-panel matplotlib figure is saved as figure_2_contact_maps.png with PCC/SCC values placed beneath each panel

## Limitations

- Current DNA foundation models (HyenaDNA, Caduceus-Ph) show limited capacity to predict domain structures and lag behind expert models like Akita in capturing long-range genomic dependencies
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because the quadratic cost of self-attention renders million-bp tasks infeasible
- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures on this task

## Evidence
