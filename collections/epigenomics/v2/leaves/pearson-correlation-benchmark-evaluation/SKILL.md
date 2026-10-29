---
name: pearson-correlation-benchmark-evaluation
description: Use when evaluating fine-tuned DNA foundation models on the DNALONGBENCH
  RSAP regression task and reporting per-sample PCC for Human, Mouse, and average
  against the expert-model benchmark.
when_to_use_negative:
- The task is one of the other DNALONGBENCH tasks (enhancer-target gene interaction,
  eQTL, contact map, or transcription initiation signals) scored by AUROC or other
  metrics rather than PCC regression.
- A task-specific expert model result is already available and only qualitative comparison
  is needed — the reproduction workflow (fine-tuning four models) is unnecessary.
- Input data are short-range DNA prediction tasks from benchmarks like Genomic Benchmarks,
  NT Benchmark, GUE, or BEND, which lack the long-range dependencies RSAP requires.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: HyenaDNA
  role: DNA foundation model fine-tuned on the RSAP regression task and scored by
    per-sample PCC
  repo: https://github.com/HazyResearch/hyena-dna
- name: Caduceus-Ph
  role: DNA foundation model fine-tuned on RSAP following experiments/Caduceus_RSAP_TISP/README.md;
    PCC evaluated per track
- name: Caduceus-PS
  role: DNA foundation model fine-tuned on RSAP; its average PCC (0.344) is compared
    against the expert model
- name: CNN
  role: Lightweight convolutional baseline fine-tuned per experiments/CNN/README.md
    on the RSAP regression task
- name: conda
  role: 'Environment management: dnalongbench env (Python 3.9) plus HyenaDNA and Caduceus
    environments for reproducible fine-tuning and inference'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:931102c8f9081fe5b6e024de2b5cf9eeaf25842e3cc869ec2be70ae374dd954a
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pearson-correlation-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the DNALONGBENCH regulatory sequence activity prediction (RSAP) benchmark table by fine-tuning a CNN baseline and three DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) and scoring them with per-sample Pearson correlation coefficients (PCC) on 5313 human and 1643 mouse tracks.

## When to use

Use when evaluating fine-tuned DNA foundation models on the DNALONGBENCH RSAP regression task and reporting per-sample PCC for Human, Mouse, and average against the expert-model benchmark.

## When NOT to use

- The task is one of the other DNALONGBENCH tasks (enhancer-target gene interaction, eQTL, contact map, or transcription initiation signals) scored by AUROC or other metrics rather than PCC regression.
- A task-specific expert model result is already available and only qualitative comparison is needed — the reproduction workflow (fine-tuning four models) is unnecessary.
- Input data are short-range DNA prediction tasks from benchmarks like Genomic Benchmarks, NT Benchmark, GUE, or BEND, which lack the long-range dependencies RSAP requires.

## Inputs

- RSAP train/valid/test TensorFlow-record (.tfr) files from DOI 10.7910/DVN/MNUEZR
- Fine-tuned model checkpoints (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS) and their test-set predictions

## Outputs

- rsap_pcc_scores.csv with per-model PCC values in Human, Mouse, and average columns
- Per-sample Pearson correlation coefficients for each model over 5313 human tracks and 1643 mouse tracks

## How to apply

Create the dnalongbench conda environment (Python 3.9), clone ma-compbio/DNALONGBENCH, install with pip, and download the RSAP data from DOI 10.7910/DVN/MNUEZR, reformulating it into train*/valid*/test*.tfr files. Set up the HyenaDNA environment from HazyResearch/hyena-dna (installing flash-attention inside the activated environment) and create the Caduceus environment from caduceus_env.yml. Load the RSAP train/valid/test TensorFlow-record splits via the dnalongbench load_data utility, then fine-tune each model: the CNN baseline, HyenaDNA, Caduceus-Ph, and Caduceus-PS following the experiment READMEs. Run inference on the held-out test .tfr files and compute the per-sample PCC separately for the 5313 human tracks and 1643 mouse tracks, then average over species. Validate by checking that the task-specific expert model achieves the highest PCC (0.669 Human, 0.479 Mouse, 0.574 average) while fine-tuned foundation models score below it.

## Related tools

- **HyenaDNA** (DNA foundation model fine-tuned on the RSAP regression task and scored by per-sample PCC) — https://github.com/HazyResearch/hyena-dna
- **Caduceus-Ph** (DNA foundation model fine-tuned on RSAP following experiments/Caduceus_RSAP_TISP/README.md; PCC evaluated per track)
- **Caduceus-PS** (DNA foundation model fine-tuned on RSAP; its average PCC (0.344) is compared against the expert model)
- **CNN** (Lightweight convolutional baseline fine-tuned per experiments/CNN/README.md on the RSAP regression task)
- **conda** (Environment management: dnalongbench env (Python 3.9) plus HyenaDNA and Caduceus environments for reproducible fine-tuning and inference)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='regulatory_sequence_activity_prediction', subset='Human', batch_size=16)
```

## Evaluation signals

- Per-sample PCC is computable for each model separately on all 5313 human tracks and 1643 mouse tracks.
- The expert model achieves the highest PCC (0.669 Human, 0.479 Mouse, 0.574 average), above the CNN (0.538, 0.323, 0.431).
- Fine-tuned foundation-model average PCC values fall in the ~0.344–0.349 range (HyenaDNA 0.347, Caduceus-Ph 0.349, Caduceus-PS 0.344).
- The output table contains Human, Mouse, and average columns for all five models (CNN, HyenaDNA, Caduceus-Ph, Caduceus-PS, expert) in rsap_pcc_scores.csv.
- The .tfr splits were reformulated correctly so train*/valid*/test* files load via the dnalongbench load_data utility without errors.

## Limitations

- Fine-tuned DNA foundation models (HyenaDNA, Caduceus-Ph, Caduceus-PS) consistently lag behind the task-specific expert model on RSAP.
- Transformer-based DNA foundation models such as DNABERT-1, DNABERT-2, and Nucleotide Transformer are excluded because the quadratic cost of self-attention makes long-range tasks infeasible for them.
- Simple fine-tuning may not be sufficient for foundation models to outperform highly specialized expert architectures.

## Evidence

- [other] expert model highest PCC on RSAP: [evidence span withheld — anchor status: not_found]
- [readme] hyena-dna clone command: [evidence span withheld — anchor status: external_doc]
