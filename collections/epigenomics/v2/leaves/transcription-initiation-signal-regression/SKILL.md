---
name: transcription-initiation-signal-regression
description: Use when benchmarking 100,000-bp DNA sequence models on nucleotide-wise
  signal-intensity regression for transcription initiation, or when verifying that
  DNA foundation models underperform a task-specific expert model on TISP.
when_to_use_negative:
- The task is binary or short-range classification (e.g. GenomicBenchmarks-style tasks)
  rather than base-pair-resolution regression of long-range signal intensities.
- You only need qualitative model rankings without reproducing the PCC benchmark table
  — the DNALONGBENCH paper's published averages (Puffin 0.733 vs. CNN 0.042) may suffice
  as reference values.
- Transformer-based DNA foundation models with quadratic self-attention (DNABERT-1,
  DNABERT-2, Nucleotide Transformer) are the models of interest — the paper excludes
  them because the quadratic cost makes 100k-bp+ tasks infeasible.
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Puffin
  role: Task-specific expert model serving as the reference TISP predictor to compare
    against (average PCC 0.733)
- name: HyenaDNA
  role: Fine-tuned DNA foundation model baseline evaluated on TISP; run via the HazyResearch/hyena-dna
    codebase following experiments/HyenaDNA_RSAP_TISP/README.md
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: Fine-tuned DNA foundation model baseline evaluated on TISP inside the caduceus_env
    conda environment
- name: Caduceus-PS
  role: Fine-tuned DNA foundation model baseline evaluated on TISP inside the caduceus_env
    conda environment
- name: CNN
  role: Convolutional neural network baseline trained on 100,000-bp sequences with
    10 channel-wise outputs following experiments/CNN/README.md
  repo: https://github.com/ma-compbio/DNALONGBENCH
- name: conda
  role: Environment manager for creating the dnalongbench (Python 3.9) and caduceus_env
    environments
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:31eb4507270ebaad68b8e095acdc32581616115a94e7ec0e9b0fba9391d12c85
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
  - Puffin
  - conda
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# transcription-initiation-signal-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the DNALONGBENCH transcription initiation signal prediction (TISP) nucleotide-wise regression benchmark, comparing the expert Puffin model against a CNN baseline and three fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) using per-assay Pearson correlation on held-out test chromosomes.

## When to use

Use when benchmarking 100,000-bp DNA sequence models on nucleotide-wise signal-intensity regression for transcription initiation, or when verifying that DNA foundation models underperform a task-specific expert model on TISP.

## When NOT to use

- The task is binary or short-range classification (e.g. GenomicBenchmarks-style tasks) rather than base-pair-resolution regression of long-range signal intensities.
- You only need qualitative model rankings without reproducing the PCC benchmark table — the DNALONGBENCH paper's published averages (Puffin 0.733 vs. CNN 0.042) may suffice as reference values.
- Transformer-based DNA foundation models with quadratic self-attention (DNABERT-1, DNABERT-2, Nucleotide Transformer) are the models of interest — the paper excludes them because the quadratic cost makes 100k-bp+ tasks infeasible.

## Inputs

- TISP bed files (Transcription Initiation Signal Prediction data) from Dataverse deposit doi:10.7910/DVN/VXQKWO
- 100,000-bp genomic DNA sequences (hg38) reformulated into train*/valid*/test*.tfr files
- train/valid/test chromosome splits with chr8 and chr9 reserved as held-out test chromosomes
- Pretrained model checkpoints: HyenaDNA weights from Hugging Face (LongSafari), Caduceus-Ph, Caduceus-PS

## Outputs

- Per-assay nucleotide-wise Pearson correlation (PCC) scores for each of the five models (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, Puffin)
- Average PCC per model aggregated across assays
- tisp_pcc_results.csv containing the per-model comparison table
- Nucleotide-wise predicted signal-intensity tracks (10 channel-wise outputs) from each model on test chromosomes

## How to apply

Create the dnalongbench conda environment with Python 3.9 (conda create -n dnalongbench python=3.9 -y), clone ma-compbio/DNALONGBENCH and pip install it, then download the TISP bed files from the Dataverse deposit at https://doi.org/10.7910/DVN/VXQKWO. Reformulate the bed data into train*/valid*/test*.tfr files and load the splits with the dnalongbench load_data utility using the transcription initiation signal task name. Train and run inference with the CNN baseline per experiments/CNN/README.md on 100,000-bp sequences with 10 channel-wise outputs; fine-tune HyenaDNA from the HazyResearch/hyena-dna codebase (PyTorch 1.13, conda env with Python 3.8+) per experiments/HyenaDNA_RSAP_TISP/README.md; and fine-tune Caduceus-Ph and Caduceus-PS in the caduceus_env environment per experiments/Caduceus_RSAP_TISP/README.md. Compute nucleotide-wise Pearson correlation (PCC) per assay on held-out test chromosomes chr8 and chr9, average per-assay PCC per model, and write the comparison table to tisp_pcc_results.csv.

## Related tools

- **Puffin** (Task-specific expert model serving as the reference TISP predictor to compare against (average PCC 0.733))
- **HyenaDNA** (Fine-tuned DNA foundation model baseline evaluated on TISP; run via the HazyResearch/hyena-dna codebase following experiments/HyenaDNA_RSAP_TISP/README.md) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (Fine-tuned DNA foundation model baseline evaluated on TISP inside the caduceus_env conda environment)
- **Caduceus-PS** (Fine-tuned DNA foundation model baseline evaluated on TISP inside the caduceus_env conda environment)
- **CNN** (Convolutional neural network baseline trained on 100,000-bp sequences with 10 channel-wise outputs following experiments/CNN/README.md) — https://github.com/ma-compbio/DNALONGBENCH
- **conda** (Environment manager for creating the dnalongbench (Python 3.9) and caduceus_env environments)

## Examples

```
conda create -n dnalongbench python=3.9 -y; train_loader, valid_loader, test_loader = load_data(root=root, task_name='transcription_initiation_signal', batch_size=16)
```

## Evaluation signals

- tisp_pcc_results.csv contains a PCC value for each of the five models (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, Puffin).
- Both per-assay PCC entries and the average PCC entry are present for every model in the output table.
- The relative ordering matches the paper's finding: Puffin (average PCC 0.733) outperforms HyenaDNA (0.132), Caduceus-Ph (0.109), Caduceus-PS (0.108), and CNN (0.042).
- Per-assay PCC is computed only on held-out test chromosomes chr8 and chr9, not on train/valid chromosomes.
- Foundation-model predictions can be checked for the paper's noted failure modes: underprediction of signal intensities or missed peaks relative to ground truth tracks.

## Limitations

- Simple fine-tuning may be insufficient for DNA foundation models to outperform specialized expert architectures; foundation models underpredict signal intensities or miss peaks on TISP.
- Transformer-based DNA foundation models (DNABERT-1, DNABERT-2, Nucleotide Transformer) are excluded because the quadratic cost of self-attention renders 100k-bp+ tasks infeasible.
- Model performance varies substantially across benchmark tasks, so TISP results may not transfer to other DNALONGBENCH tasks such as enhancer-target gene prediction or contact map prediction.

## Evidence
