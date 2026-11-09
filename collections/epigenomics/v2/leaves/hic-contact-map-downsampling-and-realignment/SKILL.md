---
name: hic-contact-map-downsampling-and-realignment
description: Use when a deeply sequenced Hi-C dataset must be reduced to controlled
  sequencing-depth levels to test how data depth affects a structure-prediction or
  interaction-detection model.
when_to_use_negative:
- Input Hi-C data is already low-depth or low-resolution — deepC with chromatin-feature
  pre-seeding is designed to 'predict intricate chromatin interactions even when trained
  on low depth, low resolution Hi-C data', so no controlled downsampling is needed
- The goal is benchmarking deepC against alternative models (e.g. Akita) — the paper
  explicitly defers such comparison to future work
- The task is TAD identification from raw Hi-C (e.g. TopDom domain calling) rather
  than sequence-based interaction prediction
tools:
- name: deepC
  role: TensorFlow deep-learning framework whose models are retrained on downsampled
    5 kb-resolution Hi-C contact maps and used via import deepCregr for chr16/17 predictions
  repo: https://github.com/rschwess/deepC
- name: TensorFlow 2.1
  role: Recommended backend for training the retrained deepC models with GPU support
    (CUDA 10.1); deepC supports v1.8, 1.14 and 2.1
- name: pybedtools
  role: Additional python library installed via pip or bioconda as part of the deepC
    environment setup
  repo: https://github.com/daler/pybedtools
- name: pysam
  role: Additional python library (tested with v0.15.2) required for handling SAM/BAM
    alignments during Hi-C preparation
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:99650c9112472aab799e05b9273cdd5e9818f7b424d4b6883091ce2342e6b282
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow 2.1
  - deepC
  - pybedtools
  - pysam
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Hi-C contact map downsampling and realignment

## Summary

Subsamples deeply sequenced Hi-C data (e.g. GM12878 from GSE63525) to defined valid-contact depths (1 billion, 100 million, 10 million) and realigns/prepares contact matrices with HiC-Pro, so that deepC models can be retrained on lower-depth data to test whether chromatin structure is still learnable.

## When to use

Use when a deeply sequenced Hi-C dataset must be reduced to controlled sequencing-depth levels to test how data depth affects a structure-prediction or interaction-detection model.

## When NOT to use

- Input Hi-C data is already low-depth or low-resolution — deepC with chromatin-feature pre-seeding is designed to 'predict intricate chromatin interactions even when trained on low depth, low resolution Hi-C data', so no controlled downsampling is needed
- The goal is benchmarking deepC against alternative models (e.g. Akita) — the paper explicitly defers such comparison to future work
- The task is TAD identification from raw Hi-C (e.g. TopDom domain calling) rather than sequence-based interaction prediction

## Inputs

- GM12878 Hi-C dataset (GEO accession GSE63525) from the deeply sequenced training set
- Downsampled Hi-C read/contact sets at 1 billion, 100 million and 10 million valid contacts
- Chromatin-feature-seeded pre-trained deepC base model from the deepC repository ./models directory

## Outputs

- HiC-Pro-processed contact matrices at 5 kb resolution for each downsampled depth
- Retrained deepC models (one per depth: 1B, 100M, 10M)
- CSV of mean Pearson correlations between predicted and observed chromatin interaction maps on chr16/17 per model

## How to apply

Start from the deep GM12878 Hi-C training data (GSE63525) and downsample it to 1 billion, 100 million and 10 million valid contacts. Realign and prepare the contact matrices with HiC-Pro following the deepC Methods, producing 5 kb-resolution contact maps suitable for deepC training. Set up the deepC environment first: install TensorFlow with GPU support (TensorFlow 2.1 recommended with CUDA 10.1, choosing the deepC compatibility branch matching the installed version: 1.8, 1.14 or 2.1) plus pybedtools and pysam. Download the pre-trained chromatin-feature-seeded base model from the deepC repository's ./models directory and retrain a 5 kb-resolution deepC model per downsampled depth set, with GPU support (several hours up to days per model). Then import the retrained models via `import deepCregr` and run sequence-based predictions over chr16/17 holdout windows (GPU strongly recommended; CPU-only prediction takes on the order of 2 h versus ~5 min with a GPU) to assess what was retained at each depth.

## Related tools

- **deepC** (TensorFlow deep-learning framework whose models are retrained on downsampled 5 kb-resolution Hi-C contact maps and used via import deepCregr for chr16/17 predictions) — https://github.com/rschwess/deepC
- **TensorFlow 2.1** (Recommended backend for training the retrained deepC models with GPU support (CUDA 10.1); deepC supports v1.8, 1.14 and 2.1)
- **pybedtools** (Additional python library installed via pip or bioconda as part of the deepC environment setup) — https://github.com/daler/pybedtools
- **pysam** (Additional python library (tested with v0.15.2) required for handling SAM/BAM alignments during Hi-C preparation) — https://github.com/pysam-developers/pysam

## Examples

```
pip install pybedtools pysam; import deepCregr
```

## Evaluation signals

- 1B-model mean Pearson correlation on holdout chr16/17 falls around 0.46 (value_in_range)
- 100M-model mean Pearson correlation on holdout chr16/17 falls around 0.4 (value_in_range)
- 10M model shows failure to learn chromatin structure, consistent with Supplementary Figure 8 (output_matches_reference)
- CSV contains per-model (1B/100M/10M) Pearson results computed over chr16/17 windows only
- Contact matrices are prepared at 5 kb resolution with HiC-Pro per the deepC Methods before retraining

## Limitations

- Sequencing depth floor: the 10M-contact downsampled model failed to learn chromatin structure, so depth below ~100M valid contacts is unreliable
- Prediction without a GPU is feasible but takes on the order of 2 h versus ~5 min with a GPU; GPU is essential for training
- deepC was developed under TensorFlow v1.8 and only supports v1.8, 1.14 and 2.1 — other versions have not been tested
- Predictions improve substantially when pre-seeded with hidden layers optimized to predict a compendium of chromatin features; skipping the chromatin-feature-seeded base model degrades results

## Evidence

- [supplementary] Models trained on 1 B and 100 M downsampled GM12878 Hi-C still learned chromatin structure, achieving mean Pearson correlations of 0.46 and 0.4 on holdout chromosomes 16 and 17, while the 10 M model failed to learn.: [evidence span withheld — anchor status: not_found]
