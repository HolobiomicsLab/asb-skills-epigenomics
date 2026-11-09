---
name: eqtl-binary-classification-finetuning
description: Use when evaluating whether DNA foundation models (HyenaDNA, Caduceus-Ph,
  Caduceus-PS) can match a task-specific expert model on binary classification of
  long-range eQTL sequences across multiple cell types.
when_to_use_negative:
- The task requires base-pair-resolution regression or 2D structure outputs (e.g.,
  contact map prediction) rather than binary classification — those DNALONGBENCH tasks
  have different output shapes and metrics
- Input sequences are short-range (well below 450,000 bp) — simpler short-range benchmarks
  like Genomic Benchmarks, GUE, or NT Benchmark are more appropriate
- You intend to evaluate transformer-based DNA foundation models (DNABERT-1, DNABERT-2,
  Nucleotide Transformer) — the paper excluded them because quadratic self-attention
  cost renders 450k bp tasks infeasible
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline for the eQTL binary classification
    task, using pretrained weights up to 450k context
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned bidirectional Mamba-based DNA foundation model baseline for eQTL
    prediction
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: Caduceus-PS
  role: Fine-tuned complement-strand Mamba-based DNA foundation model baseline for
    eQTL prediction
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: CNN
  role: Lightweight convolutional baseline fine-tuned per experiments/CNN/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: dnalongbench.utils.load_data
  role: Loads eQTL train/valid/test .tfr splits (batch_size=16) for training and evaluation
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: conda
  role: Creates the dnalongbench (Python 3.9), hyena-dna (Python 3.8), and caduceus
    environments
  repo: https://github.com/ma-compbio/DNALONGBENCH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:33e885f711e2cccda5dbf5b98fcbf9b5d743ef8a928cc8b9e287eafa5d300759
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# eqtl-binary-classification-finetuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the DNALONGBENCH eQTL prediction (eQTLP) benchmark by fine-tuning a CNN baseline, HyenaDNA, Caduceus-Ph, and Caduceus-PS on a binary classification task over 450,000 bp sequences (31,282 samples) and comparing their per-cell-type AUROC against the Enformer Expert Model, which averages 0.681 across nine cell types.

## When to use

Use when evaluating whether DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) can match a task-specific expert model on binary classification of long-range eQTL sequences across multiple cell types.

## When NOT to use

- The task requires base-pair-resolution regression or 2D structure outputs (e.g., contact map prediction) rather than binary classification — those DNALONGBENCH tasks have different output shapes and metrics
- Input sequences are short-range (well below 450,000 bp) — simpler short-range benchmarks like Genomic Benchmarks, GUE, or NT Benchmark are more appropriate
- You intend to evaluate transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) — the paper excluded them because quadratic self-attention cost renders 450k bp tasks infeasible

## Inputs

- eQTL task bed files from https://doi.org/10.7910/DVN/YUP2G5 (450,000 bp input sequences, binary labels, 31,282 samples across nine cell types)
- train/valid/test .tfr splits loadable via dnalongbench.utils.load_data(task_name='eqtl')
- DNALONGBENCH repository experiment configs (experiments/CNN, experiments/HyenaDNA, experiments/Caduceus)
- Pretrained HyenaDNA weights from HuggingFace (e.g., medium-450k) and Caduceus checkpoints

## Outputs

- Fine-tuned CNN, HyenaDNA, Caduceus-Ph, and Caduceus-PS checkpoints for the eQTL task
- Per-cell-type and average AUROC values on the held-out test split (Expert Model reference: 0.639, 0.689, 0.612, 0.710, 0.700, 0.621, 0.683, 0.741, 0.736; average 0.681)
- eqtl_auroc_results.csv summarizing reproduced Table 7 results

## How to apply

Create the dnalongbench conda environment with Python 3.9, clone ma-compbio/DNALONGBENCH and install it with 'pip install .'. Download the eQTL bed files from https://doi.org/10.7910/DVN/YUP2G5, reformulate them into train*/valid*/test*.tfr files, and load them with dnalongbench.utils.load_data(task_name='eqtl', batch_size=16). Fine-tune the lightweight CNN per experiments/CNN/README.md, HyenaDNA in its own environment (PyTorch 1.13, cuda 11.7) per experiments/HyenaDNA/HyenaDNA_ETGP_CMP_eQTLP/README.md, and Caduceus-Ph/Caduceus-PS in the caduceus_env created from caduceus_env.yml per experiments/Caduceus/Caduceus_CMP_eQTLP_ETGP/README.md. Evaluate each fine-tuned model on the held-out test split, compute AUROC per cell type and the average, and write eqtl_auroc_results.csv. The rationale is to test the paper's claim that expert models consistently outperform fine-tuned DNA foundation models on long-range tasks.

## Related tools

- **HyenaDNA** (Fine-tuned DNA foundation model baseline for the eQTL binary classification task, using pretrained weights up to 450k context) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned bidirectional Mamba-based DNA foundation model baseline for eQTL prediction) — https://github.com/ma-compbio/DNALONGBENCH
- **Caduceus-PS** (Fine-tuned complement-strand Mamba-based DNA foundation model baseline for eQTL prediction) — https://github.com/ma-compbio/DNALONGBENCH
- **CNN** (Lightweight convolutional baseline fine-tuned per experiments/CNN/README.md) — https://github.com/ma-compbio/DNALONGBENCH
- **dnalongbench.utils.load_data** (Loads eQTL train/valid/test .tfr splits (batch_size=16) for training and evaluation) — https://github.com/ma-compbio/DNALONGBENCH
- **conda** (Creates the dnalongbench (Python 3.9), hyena-dna (Python 3.8), and caduceus environments) — https://github.com/ma-compbio/DNALONGBENCH

## Examples

```
conda create -n dnalongbench python=3.9 -y && conda activate dnalongbench && pip install . && python -c "from dnalongbench.utils import load_data; train,valid,test = load_data(root='data/eqtl', task_name='eqtl', batch_size=16)"
```

## Evaluation signals

- Reproduced Table 7 AUROC values confirm the Expert Model (Enformer) attains the highest AUROC on the eQTL task in every one of the nine cell types
- Expert Model average AUROC of 0.681 is the highest mean across the five compared methods
- Each per-cell-type AUROC falls in a plausible range (~0.55–0.75); Expert Model values range from 0.612 to 0.741
- eqtl_auroc_results.csv contains nine per-cell-type AUROC entries plus an average for each fine-tuned model
- Test AUROC is computed strictly on the held-out test split loaded via load_data, not the validation split

## Limitations

- The study excluded transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) because the quadratic cost of self-attention makes 450k bp tasks infeasible
- Simple fine-tuning may be insufficient for foundation models to outperform highly specialized expert architectures; substantial room remains via novel architectures, advanced fine-tuning strategies, and task-specific training objectives
- HyenaDNA hardware requirements scale with context length — 450k-length models (medium-450k) require large GPUs; 1M-token inference needs an A100 (paid Colab tier) or own machine

## Evidence

- [other] Expert Model AUROC values and average: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark scope: [evidence span withheld — anchor status: not_found]
- [readme] HyenaDNA setup steps: [evidence span withheld — anchor status: external_doc]
