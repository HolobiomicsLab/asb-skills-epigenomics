---
name: random-forest-binding-prediction
description: Use when the goal is to predict or reproduce TF-DNA binding affinities
  from experimental intensity/enrichment data (gcPBM, uPBM, HT-SELEX) using structural/mechanical
  DNA features rather than sequence-only models.
when_to_use_negative:
- The input is already a trained model output or feature table rather than raw experimental
  affinity/intensity data requiring regression
- The task is in vivo binding site identification without accompanying chromatin structure
  data — DNAffinity alone predicts many potential sites with no experimental in vivo
  binding evidence and needs nucleosome positioning maps for in vivo TFBS localization
- Sequence-only deep learning is preferred and structural/mechanical DNA features
  from MD simulations are unavailable
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0654
tools:
- name: DNAffinity
  role: Random-forest regressor trained on MD-derived DNA equilibrium-geometry and
    flexibility features; provides gcpbm_regressor.py, upbm_regressor.py, selex_regressor.py
    and the feature parameter database
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Used in preprocessing to trim uPBM sequences and reduce sequence variability
    before regression
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:a4655e453e7fbf5ae889b1e42a96b145a41a6a37f513ce0caf2a153e7ff81c93
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# random-forest-binding-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a random forest regressor on DNA equilibrium-geometry and flexibility features derived from atomistic MD simulations to reproduce in vitro transcription factor binding affinities (gcPBM, uPBM, HT-SELEX), yielding R2-scored per-protein affinity predictions.

## When to use

Use when the goal is to predict or reproduce TF-DNA binding affinities from experimental intensity/enrichment data (gcPBM, uPBM, HT-SELEX) using structural/mechanical DNA features rather than sequence-only models.

## When NOT to use

- The input is already a trained model output or feature table rather than raw experimental affinity/intensity data requiring regression
- The task is in vivo binding site identification without accompanying chromatin structure data — DNAffinity alone predicts many potential sites with no experimental in vivo binding evidence and needs nucleosome positioning maps for in vivo TFBS localization
- Sequence-only deep learning is preferred and structural/mechanical DNA features from MD simulations are unavailable

## Inputs

- proteins/{protein}/{protein}_{concentration}.txt — concentration-specific gcPBM intensity file
- proteins/{protein}/_freq_matrix_6.txt — 6-mer frequency matrix
- proteins/{protein}/{protein}_training.txt — preprocessed uPBM training file
- proteins/{protein}/{cycle}.txt — preprocessed HT-SELEX cycle file

## Outputs

- Per-protein predicted binding affinities for the held-out 20% test partition
- Per-protein R2 determination coefficients and mean ± standard deviation (e.g. gcpbm_regressor_results.csv)

## How to apply

Prepare per-protein input files under proteins/{protein}/ (e.g. {protein}_{concentration}.txt gcPBM intensity file plus _freq_matrix_6.txt, verifying formatting against package templates). Launch the method-specific regressor script passing the protein name as the 1st argument and, for gcPBM, the concentration as the 2nd argument. The script splits the dataset 80/20, trains a random forest regressor on the 80% partition using DNA equilibrium-geometry and flexibility features at base-pair-step resolution from MD simulations, and computes the determination coefficient R2 on the held-out 20%. For HT-SELEX, first apply SVM to remove cases with inconsistency between enrichments at different cycles; for uPBM, trim sequences based on MEME suite results. Save per-protein predictions and R2 scores (e.g. to gcpbm_regressor_results.csv) and check against reported benchmarks: gcPBM average R2 = 0.93 ± 0.02 with CBF1 at R2 = 0.80, uPBM average R2 = 0.69 ± 0.17.

## Related tools

- **DNAffinity** (Random-forest regressor trained on MD-derived DNA equilibrium-geometry and flexibility features; provides gcpbm_regressor.py, upbm_regressor.py, selex_regressor.py and the feature parameter database) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Used in preprocessing to trim uPBM sequences and reduce sequence variability before regression)

## Examples

```
python gcpbm_regressor.py CBF1 8
```

## Evaluation signals

- Per-protein R2 values on the held-out 20% should aggregate to gcPBM average R2 = 0.93 ± 0.02, with CBF1 reproducing R2 = 0.80
- uPBM runs should reach average R2 = 0.69 ± 0.17; substantial deviation flags preprocessing or split problems
- Input file formatting matches the package templates ({protein}_{concentration}.txt and _freq_matrix_6.txt for gcPBM)
- Results file (e.g. gcpbm_regressor_results.csv) contains per-protein predictions and R2 scores for all proteins (Mad, Max, Myc, CBF1)
- 80/20 train-test split is applied consistently per dataset, with R2 computed between predictions and experimental values on the remaining 20%

## Limitations

- TFBS concept is not uniquely defined — it depends on the intrinsically noisy, low-resolution experimental technique used to detect it, preventing a universal predictor
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity
- DNAffinity predicts many potential binding sites for which no experimental evidence of in vivo binding exists
- The informative parameters depend on the sequence variability explored by the experiment

## Evidence
