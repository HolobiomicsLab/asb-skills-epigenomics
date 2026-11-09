---
name: dna-sequence-multitask-regression
description: Use when reproducing or benchmarking multi-task DNA regression on DNALONGBENCH
  RSAP (TensorFlow-record train/valid/test splits) and per-sample PCC on human vs
  mouse tracks is the target metric.
when_to_use_negative:
- The task is short-range or classification-based (e.g. enhancer-target gene or eQTL
  AUROC tasks) rather than the RSAP multi-track regression evaluated with PCC
- Only transformer-based DNA foundation models with quadratic self-attention are available,
  since the quadratic cost renders such ultra-long (up to 1 million bp) tasks infeasible
- The goal is sequence generation rather than supervised regression — use generative
  foundation models like GENERator instead
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: conda
  role: Creates the dnalongbench (Python 3.9) and Caduceus environments used to run
    fine-tuning and inference
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline evaluated on the RSAP regression
    task
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline evaluated on the RSAP regression
    task
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline evaluated on the RSAP regression
    task
- name: CNN
  role: Lightweight convolutional baseline fine-tuned on RSAP following experiments/CNN/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:b2d8f7c2e936d4208d871596e024d9488315e7c455a7cc2239e1a59bbf677521
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# dna-sequence-multitask-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tune a lightweight CNN baseline and DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) on the DNALONGBENCH regulatory sequence activity prediction (RSAP) regression task over 5313 human and 1643 mouse tracks, then score per-sample Pearson correlation (PCC). The skill exists because DNALONGBENCH shows current DNA foundation models still lag behind task-specific expert models on long-range regression.

## When to use

Use when reproducing or benchmarking multi-task DNA regression on DNALONGBENCH RSAP (TensorFlow-record train/valid/test splits) and per-sample PCC on human vs mouse tracks is the target metric.

## When NOT to use

- The task is short-range or classification-based (e.g. enhancer-target gene or eQTL AUROC tasks) rather than the RSAP multi-track regression evaluated with PCC
- Only transformer-based DNA foundation models with quadratic self-attention are available, since the quadratic cost renders such ultra-long (up to 1 million bp) tasks infeasible
- The goal is sequence generation rather than supervised regression — use generative foundation models like GENERator instead

## Inputs

- RSAP data from DOI 10.7910/DVN/MNUEZR reformulated into train*/valid*/test*.tfr TensorFlow-record files
- Pretrained HyenaDNA weights from Hugging Face (LongSafari) or the hyena-dna repo checkpoints
- Caduceus-Ph and Caduceus-PS checkpoints with caduceus_env.yml environment spec
- CNN baseline configuration from experiments/CNN/README.md

## Outputs

- Per-model PCC results table with Human, Mouse, and average columns (e.g. rsap_pcc_scores.csv) for CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, and the expert model

## How to apply

Create the dnalongbench conda environment (Python 3.9), clone ma-compbio/DNALONGBENCH with git and pip install it, then download RSAP data from DOI 10.7910/DVN/MNUEZR and reformulate it into train*/valid*/test*.tfr files. Load the splits with the dnalongbench load_data utility to obtain train/valid/test loaders. Fine-tune the CNN baseline, HyenaDNA (from HazyResearch/hyena-dna, installing flash-attention as a submodule inside the activated environment), and Caduceus-Ph/Caduceus-PS (conda env from caduceus_env.yml) following the per-model experiment READMEs. Run inference on the held-out test .tfr files for each of the four models and compute the per-sample Pearson correlation coefficient (PCC) separately for the 5313 human tracks and 1643 mouse tracks, then average over species. The rationale is to compare fine-tuned foundation models against the CNN baseline and the task-specific expert model, which the benchmark shows consistently achieves the highest scores (0.574 average PCC vs 0.347/0.349/0.344 for the foundation models).

## Related tools

- **conda** (Creates the dnalongbench (Python 3.9) and Caduceus environments used to run fine-tuning and inference)
- **HyenaDNA** (Fine-tuned DNA foundation model baseline evaluated on the RSAP regression task) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline evaluated on the RSAP regression task)
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline evaluated on the RSAP regression task)
- **CNN** (Lightweight convolutional baseline fine-tuned on RSAP following experiments/CNN/README.md) — https://github.com/ma-compbio/DNALONGBENCH

## Examples

```
git clone https://github.com/ma-compbio/DNALONGBENCH.git && pip install .
```

## Evaluation signals

- Per-sample PCC is computable for each model separately on the 5313 human tracks and 1643 mouse tracks, and a species-average is derivable
- The reproduced PCC ordering matches the benchmark: the expert model achieves the highest PCC (0.669 Human, 0.479 Mouse, 0.574 average) while the fine-tuned foundation models score below it (averages 0.347–0.349)
- The CNN baseline PCCs (0.538 Human, 0.323 Mouse, 0.431 average) fall between the expert model and the foundation models
- Data splits load correctly via the load_data utility as train*/valid*/test*.tfr before training begins

## Limitations

- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded from the benchmark evaluation
- The quadratic cost of the self-attention mechanism makes ultra-long-range tasks infeasible for standard transformer models
- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures; substantial room remains via novel architectures, fine-tuning strategies, and task-specific training objectives

## Evidence
