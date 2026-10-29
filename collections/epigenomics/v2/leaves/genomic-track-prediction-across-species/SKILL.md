---
name: genomic-track-prediction-across-species
description: Use when reproducing or extending the RSAP PCC benchmark table in DNALONGBENCH,
  i.e., when fine-tuned DNA foundation models must be evaluated against expert and
  CNN baselines on human and mouse regulatory activity tracks.
when_to_use_negative:
- Task is a short-range DNA benchmark (e.g., GenomicBenchmarks, GUE, NT Benchmark)
  that does not require long-range dependencies up to 1 million bp
- Goal is base-pair-resolution or 2D-structure tasks (e.g., contact map prediction,
  enhancer-target gene prediction) rather than regulatory activity regression
- Only a single model or a single species is needed without the cross-model, cross-species
  benchmark comparison
edam_operation: http://edamontology.org/operation_0480
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: HyenaDNA
  role: DNA foundation model fine-tuned on the RSAP regression task using the HazyResearch/hyena-dna
    training pipeline (PyTorch Lightning + Hydra configs)
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Mamba-based DNA foundation model variant fine-tuned on RSAP via the Caduceus
    conda environment (caduceus_env.yml)
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: Caduceus-PS
  role: Mamba-based DNA foundation model variant (pretrain-on-species flavor) fine-tuned
    on RSAP alongside Caduceus-Ph
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: CNN
  role: Lightweight convolutional baseline fine-tuned on RSAP following experiments/CNN/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: conda
  role: Environment manager used to create the dnalongbench (Python 3.9) and Caduceus
    environments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:efee03b22e9593d374b69c9249a4bb3a6f678733c057f0bcfc866338c66416bc
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
  edam_operation: http://edamontology.org/operation_0480
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# genomic-track-prediction-across-species

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the DNALONGBENCH regulatory sequence activity prediction (RSAP) benchmark by fine-tuning a CNN baseline and the DNA foundation models HyenaDNA, Caduceus-Ph, and Caduceus-PS, then computing per-sample Pearson correlation coefficients (PCC) on 5313 human and 1643 mouse test tracks to compare against the task-specific expert model.

## When to use

Use when reproducing or extending the RSAP PCC benchmark table in DNALONGBENCH, i.e., when fine-tuned DNA foundation models must be evaluated against expert and CNN baselines on human and mouse regulatory activity tracks.

## When NOT to use

- Task is a short-range DNA benchmark (e.g., GenomicBenchmarks, GUE, NT Benchmark) that does not require long-range dependencies up to 1 million bp
- Goal is base-pair-resolution or 2D-structure tasks (e.g., contact map prediction, enhancer-target gene prediction) rather than regulatory activity regression
- Only a single model or a single species is needed without the cross-model, cross-species benchmark comparison

## Inputs

- RSAP dataset from DOI 10.7910/DVN/MNUEZR reformulated into train*/valid*/test*.tfr TensorFlow-record files
- Pretrained HyenaDNA weights from HuggingFace (LongSafari)
- Caduceus-Ph and Caduceus-PS checkpoints
- Test split .tfr files for 5313 human tracks and 1643 mouse tracks

## Outputs

- rsap_pcc_scores.csv with per-model Pearson correlation columns for Human, Mouse, and species average
- Per-model test-set predictions on human and mouse tracks

## How to apply

Create the dnalongbench conda environment (Python 3.9 via 'conda create -n dnalongbench python=3.9 -y'), clone ma-compbio/DNALONGBENCH and install it with pip, then download the RSAP data from DOI 10.7910/DVN/MNUEZR and reformulate it into train*/valid*/test*.tfr TensorFlow-record files. Set up the HyenaDNA environment from HazyResearch/hyena-dna (installing flash-attention as a submodule inside the activated environment) and create the Caduceus environment with 'conda env create -f caduceus_env.yml'. Load the RSAP train/valid/test splits via the dnalongbench load_data utility, then fine-tune the CNN baseline, HyenaDNA, Caduceus-Ph, and Caduceus-PS on the RSAP regression task following the per-model experiment READMEs. Run inference on the held-out test .tfr files and compute the per-sample PCC separately for the 5313 human tracks and 1643 mouse tracks, then average over species. Write the per-model Human/Mouse/average table to rsap_pcc_scores.csv; the expert model should achieve the highest PCC (0.669 Human, 0.479 Mouse, 0.574 average) with the foundation models below it (averages 0.347, 0.349, 0.344).

## Related tools

- **HyenaDNA** (DNA foundation model fine-tuned on the RSAP regression task using the HazyResearch/hyena-dna training pipeline (PyTorch Lightning + Hydra configs)) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Mamba-based DNA foundation model variant fine-tuned on RSAP via the Caduceus conda environment (caduceus_env.yml)) — https://github.com/ma-compbio/DNALONGBENCH
- **Caduceus-PS** (Mamba-based DNA foundation model variant (pretrain-on-species flavor) fine-tuned on RSAP alongside Caduceus-Ph) — https://github.com/ma-compbio/DNALONGBENCH
- **CNN** (Lightweight convolutional baseline fine-tuned on RSAP following experiments/CNN/README.md) — https://github.com/ma-compbio/DNALONGBENCH
- **conda** (Environment manager used to create the dnalongbench (Python 3.9) and Caduceus environments)

## Examples

```
python -m train wandb=null experiment=hg38/genomic_benchmark_scratch  # fine-tuning entry point from hyena-dna; RSAP splits loaded via dnalongbench load_data(root=root, task_name='regulatory_sequence_activity_prediction', subset='Human', batch_size=16)
```

## Evaluation signals

- Per-sample PCC is computable for each model on both 5313 human tracks and 1643 mouse tracks from the held-out test .tfr files
- Expert model achieves the highest PCC (0.669 Human, 0.479 Mouse, 0.574 average), confirming the benchmark finding that expert models consistently outperform foundation models
- Fine-tuned foundation-model averages fall near the reported values: HyenaDNA 0.347, Caduceus-Ph 0.349, Caduceus-PS 0.344
- CNN baseline reproduces PCC of 0.538 Human, 0.323 Mouse, 0.431 average, above the foundation models
- rsap_pcc_scores.csv contains Human, Mouse, and average columns for all four fine-tuned models

## Limitations

- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures on long-range tasks
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) were excluded because the quadratic cost of self-attention renders long-range tasks infeasible for them
- Model performance varies substantially across tasks, so RSAP PCC results should not be generalized to other DNALONGBENCH tasks

## Evidence

- [readme] HyenaDNA repo clone with submodules and flash-attention install: [evidence span withheld — anchor status: external_doc]
