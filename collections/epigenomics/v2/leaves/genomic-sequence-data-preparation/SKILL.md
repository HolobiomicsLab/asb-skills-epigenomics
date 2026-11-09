---
name: genomic-sequence-data-preparation
description: Use when a training run for deepC (or a similar sequence-based chromatin
  interaction model) needs genome sequence windows plus 5 kb Hi-C skeleton targets
  assembled from raw genome assemblies, BED intervals, and Hi-C contact data.
when_to_use_negative:
- A trained deepC checkpoint already exists and the goal is running predictions —
  deepC predictions are feasible without GPU and do not require re-preparing training
  data
- The model of interest is Akita or another non-deepC architecture whose input and
  target formats differ from deepC's 5 kb skeleton scheme
- Hi-C data comes from a tissue/cell type for which no matching chromatin feature
  compendium or reference genome preparation is available
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3292
tools:
- name: deepC
  role: Framework containing the core python code, R scripts and tutorials for preparing
    Hi-C data and sequence inputs into deepC training format
  repo: https://github.com/rschwess/deepC
- name: pysam
  role: Python module for reading and manipulating sequence/alignment files (SAM/BAM,
    FASTA) when extracting genome sequence for training windows
  repo: https://github.com/pysam-developers/pysam
- name: pybedtools
  role: Interval arithmetic over BED tracks to construct the 5 kb skeleton bins and
    coordinate sequence windows with Hi-C targets
- name: TensorFlow
  role: Deep learning backend (v1.8, 1.14 or 2.1) whose docker containers package
    matching CUDA/cuDNN for GPU training of prepared data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0960-3
    title: 'DeepC: predicting 3D genome folding using megabase-scale transfer learning'
  source_content_hash: sha256:997d4a6c08fe6b35be991c621822bf0d2d7a5e22a4a346941f38a6936ee7f1c1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rschwess/deepC
  tools:
  - TensorFlow
  - deepC
  - pybedtools
  - pysam
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genomic-sequence-data-preparation

## Summary

Prepare megabase-scale DNA sequence inputs and Hi-C-derived training targets for deepC's transfer learning models: install pysam and pybedtools alongside TensorFlow, clone the deepC repository, and derive 5 kb skeleton target tracks from Hi-C data such as Rao et al. (GSE63525) for GM12878 and K562.

## When to use

Use when a training run for deepC (or a similar sequence-based chromatin interaction model) needs genome sequence windows plus 5 kb Hi-C skeleton targets assembled from raw genome assemblies, BED intervals, and Hi-C contact data.

## When NOT to use

- A trained deepC checkpoint already exists and the goal is running predictions — deepC predictions are feasible without GPU and do not require re-preparing training data
- The model of interest is Akita or another non-deepC architecture whose input and target formats differ from deepC's 5 kb skeleton scheme
- Hi-C data comes from a tissue/cell type for which no matching chromatin feature compendium or reference genome preparation is available

## Inputs

- Reference genome assembly (FASTA) accessible via pysam
- Genomic interval files (BED) for tiling 5 kb Hi-C skeleton bins
- Hi-C contact maps from Rao et al. (GEO GSE63525) for GM12878 and K562
- ENCODE/Roadmap chromatin feature datasets (~936 datasets) for the pre-training phase
- DeepC repository cloned from github.com/rschwess/deepC

## Outputs

- Megabase-scale one-hot/encoded DNA sequence windows matching deepC's expected input resolution
- 5 kb Hi-C skeleton target tracks per cell type (GM12878, K562)
- Chromatin feature training matrices for the DeepSEA-adapted pre-training network
- Environment with TensorFlow (GPU), pysam, pybedtools, and the correct deepC compatibility branch

## How to apply

Set up a GPU-enabled TensorFlow environment using the official TensorFlow docker containers (TensorFlow 2.1 recommended; deepC was developed under v1.8 and also supports 1.14), which package the matching CUDA and cuDNN versions. Clone the deepC GitHub repository and select the compatibility branch matching the installed TensorFlow release (e.g. the tensorflow2.1plus_compatibility_version branch for 2.1+). Install pysam and pybedtools via pip or bioconda (pysam tested with v0.15.2) plus bedtools, so genome FASTA/BAM access and interval arithmetic are available. Use pybedtools to tile the genome into the 5 kb bins that define the Hi-C skeleton targets and use pysam to retrieve the corresponding megabase-scale DNA sequence windows feeding the ~60M-parameter conv-plus-dilated-convolution network (dilated convolutions adapted from wavenet). Pair the sequence windows with tissue-specific targets (Rao et al. Hi-C, GEO GSE63525, for GM12878 and K562) or the ENCODE/Roadmap compendium of ~936 chromatin feature datasets when training the DeepSEA-adapted chromatin feature network used for transfer learning pre-seeding.

## Related tools

- **deepC** (Framework containing the core python code, R scripts and tutorials for preparing Hi-C data and sequence inputs into deepC training format) — https://github.com/rschwess/deepC
- **pysam** (Python module for reading and manipulating sequence/alignment files (SAM/BAM, FASTA) when extracting genome sequence for training windows) — https://github.com/pysam-developers/pysam
- **pybedtools** (Interval arithmetic over BED tracks to construct the 5 kb skeleton bins and coordinate sequence windows with Hi-C targets)
- **TensorFlow** (Deep learning backend (v1.8, 1.14 or 2.1) whose docker containers package matching CUDA/cuDNN for GPU training of prepared data)

## Examples

```
pip install pysam; pip install pybedtools
```

## Evaluation signals

- Installed TensorFlow version is one of the tested releases (v1.8, 1.14, 2.1) and the deepC branch matches it
- pysam (v0.15.2 tested) and pybedtools import successfully and pybedtools can locate a compatible bedtools binary
- Tiled target bins tile the genome without gaps/overlaps at 5 kb resolution and sequence window coordinates match bin coordinates on the same genome build
- Downstream interaction network checkpoints load as valid HDF5 files with a parameter count within tolerance of ~60M
- Training loss decreases across logged epochs when the prepared data is fed to the GPU-enabled training run

## Limitations

- deepC was developed under TensorFlow v1.8; other versions besides 1.14 and 2.1 have not been tested, so data-prep code may break on untested releases
- GPU support is essential for training (predictions run without GPU but are significantly slower), so CPU-only environments cannot complete the training phase
- The interaction network was trained on 5 kb Hi-C skeleton targets from only GM12878 and K562; other cell types require regenerating targets from their own Hi-C data

## Evidence
