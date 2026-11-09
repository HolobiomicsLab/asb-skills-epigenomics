---
name: dna-physical-property-feature-preparation
description: Use when training or validating DNAffinity regressors on in vitro binding
  data (gcPBM, uPBM, HT-SELEX) and you must assemble the per-protein input files and
  6-mer feature inputs correctly.
when_to_use_negative:
- The goal is in vivo TFBS identification on yeast genome peaks (PB-exo/ChIP-exo)
  rather than in vitro affinity regression — that downstream application assumes a
  regressor already trained (e.g. CBF1 R2 = 0.80)
- Input data are HT-SELEX enrichments inconsistent across cycles, which must first
  be filtered with SVM before training/testing
- Sequences are uPBM data that have not yet been trimmed based on the MEME suite result
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3125
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Supplies the MD-derived equilibrium geometry and flexibility feature database
    and the {method}_regressor.py scripts that consume the prepared per-protein input
    files
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Used to trim uPBM sequences prior to feature preparation, reducing sequence
    variability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:13f2258d1d242a5ddfbee5361a7237b73e050ea50554e14dc696e44aee051ae9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Jalbiti/DNAffinity
  repo_url: https://github.com/Jalbiti/DNAffinity
  tools:
  - DNAffinity
  - MEME suite
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3125
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dna-physical-property-feature-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare the DNAffinity input files and physical-property features (DNA equilibrium geometry and flexibility from MD simulations) needed to train the random-forest affinity regressor for a transcription factor, e.g. on gcPBM data for Mad, Max, Myc and CBF1.

## When to use

Use when training or validating DNAffinity regressors on in vitro binding data (gcPBM, uPBM, HT-SELEX) and you must assemble the per-protein input files and 6-mer feature inputs correctly.

## When NOT to use

- The goal is in vivo TFBS identification on yeast genome peaks (PB-exo/ChIP-exo) rather than in vitro affinity regression — that downstream application assumes a regressor already trained (e.g. CBF1 R2 = 0.80)
- Input data are HT-SELEX enrichments inconsistent across cycles, which must first be filtered with SVM before training/testing
- Sequences are uPBM data that have not yet been trimmed based on the MEME suite result

## Inputs

- gcPBM concentration-specific intensity file per protein: proteins/{protein}/{protein}_{concentration}.txt
- 6-mer frequency matrix per protein: proteins/{protein}/_freq_matrix_6.txt
- uPBM training file: proteins/{protein}/{protein}_training.txt
- HT-SELEX cycle file: proteins/{protein}/{cycle}.txt
- DNA equilibrium geometry and flexibility feature parameters from the DNAffinity GitHub repository

## Outputs

- Trained random-forest regressor per protein/dataset
- Per-protein predictions and R2 scores (e.g. gcpbm_regressor_results.csv)
- Determination coefficient (R2) between predictions and experimental affinities on the held-out 20%

## How to apply

For each protein (e.g. Mad, Max, Myc, CBF1), place the required files in proteins/{protein}/: for gcPBM this means the concentration-specific intensity file {protein}_{concentration}.txt plus the 6-mer frequency matrix _freq_matrix_6.txt, checking formatting against the package templates. Launch the appropriate {method}_regressor.py, passing the protein name as the 1st argument and, for gcpbm_regressor.py, the concentration as the 2nd argument. The regressor describes DNA conformation at base-pair-step resolution using equilibrium geometry and flexibility parameters derived from atomistic molecular dynamics simulations, and trains a random forest on 80% of the data. Decision point: use gcpbm_regressor.py for gcPBM, upbm_regressor.py for uPBM ({protein}_training.txt), and selex_regressor.py for HT-SELEX ({cycle}.txt, cycle number as 2nd argument). Correctness is judged by the R2 computed on the held-out 20% (gcPBM target: average 0.93 ± 0.02; CBF1: R2 = 0.80).

## Related tools

- **DNAffinity** (Supplies the MD-derived equilibrium geometry and flexibility feature database and the {method}_regressor.py scripts that consume the prepared per-protein input files) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Used to trim uPBM sequences prior to feature preparation, reducing sequence variability)

## Examples

```
python gcpbm_regressor.py CBF1 8B1
```

## Evaluation signals

- Per-protein R2 on the held-out 20% aggregates to mean ± SD ≈ 0.93 ± 0.02 across gcPBM-trained transcription factors
- CBF1 gcPBM regression reproduces R2 = 0.80
- All required files present with formatting matching the package templates: {protein}_{concentration}.txt and _freq_matrix_6.txt for gcpbm_regressor.py
- Results file gcpbm_regressor_results.csv contains per-protein predictions and R2 scores
- 80/20 train/test split applied consistently as in the article's Materials and Methods

## Limitations

- The concept of a transcription factor binding site is not uniquely defined — it depends on the intrinsically noisy, low-resolution experimental technique used to detect it, so a universal predictor is impossible
- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists; extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity
- Regressors are dataset-specific: gcpbm_regressor.py, upbm_regressor.py and selex_regressor.py each require different file layouts and second arguments (concentration vs cycle number)

## Evidence
