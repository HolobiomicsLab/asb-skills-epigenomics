---
name: pwm-weighted-high-affinity-sequence-annotation
description: Use when preparing uPBM training files (35-mer sequences with experimental
  affinities, e.g. proteins/{protein}/{protein}_training.txt) for per-TF affinity
  regression where sequence variability must be reduced and PWM-matching high-affinity
  k-mers emphasized.
when_to_use_negative:
- Input is gcPBM or HT-SELEX data — these have their own preprocessing (e.g. gcpbm_regressor.py
  needs `{protein}_{concentration}.txt` and `_freq_matrix_6.txt`; selex needs `{cycle}.txt`
  files and SVM filtering of inconsistent enrichment cycles)
- The goal is in vivo binding site prediction, where DNAffinity predictions must instead
  be combined with yeast G1-phase nucleosome maps
- The dataset lacks a PWM/MEME motif definition, so high-affinity sequences cannot
  be identified for weighting
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Random-forest regressor trained on MD-derived DNA equilibrium geometry and
    flexibility features; run via `upbm_regressor.py` on the trimmed, PWM-weighted
    uPBM data
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Provides the motif used to trim uPBM 35-mers to the motif region, reducing
    sequence variability before training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:95c3edc5a1e4c04c087ddccb94333e8eb81656b52bc5191eda01423d1e40bc0e
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# PWM-weighted high-affinity sequence annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A uPBM-specific preprocessing and sample-weighting step for DNAffinity: 35-mer oligos are trimmed to the MEME-derived motif region, and sequences matching the PWM-derived high-affinity motif are assigned 10x weight (with undersampling of low-affinity k-mers) before training the random-forest regressor.

## When to use

Use when preparing uPBM training files (35-mer sequences with experimental affinities, e.g. proteins/{protein}/{protein}_training.txt) for per-TF affinity regression where sequence variability must be reduced and PWM-matching high-affinity k-mers emphasized.

## When NOT to use

- Input is gcPBM or HT-SELEX data — these have their own preprocessing (e.g. gcpbm_regressor.py needs `{protein}_{concentration}.txt` and `_freq_matrix_6.txt`; selex needs `{cycle}.txt` files and SVM filtering of inconsistent enrichment cycles)
- The goal is in vivo binding site prediction, where DNAffinity predictions must instead be combined with yeast G1-phase nucleosome maps
- The dataset lacks a PWM/MEME motif definition, so high-affinity sequences cannot be identified for weighting

## Inputs

- Per-TF uPBM training file `proteins/{protein}/{protein}_training.txt` with 35-mer sequences and experimental binding affinities
- MEME suite motif result for the TF (used to define the trim region)
- PWM-derived high-affinity motif for the TF

## Outputs

- Trimmed, undersampled, PWM-weighted training set per TF
- Per-TF random-forest regressor predictions on the held-out 20% of sequences
- Per-TF and average determination coefficient (R2) across the 66 mouse TFs

## How to apply

Load the per-TF uPBM training file `proteins/{protein}/{protein}_training.txt` containing 35-mers with experimental binding affinities for each of the 66 mouse TFs. Trim each sequence to the motif region using the MEME suite result, which reduces sequence variability before feature extraction. Balance the training set by undersampling low-affinity k-mers and assigning 10x weight to sequences matching the PWM-derived high-affinity motif. Train the DNAffinity random-forest regressor on MD-derived DNA equilibrium geometry and flexibility features using 80% of the trimmed, weighted sequences, then compute R2 on the held-out 20%. Repeat for each TF and compare the average R2 against the reported benchmark of 0.69 ± 0.17.

## Related tools

- **DNAffinity** (Random-forest regressor trained on MD-derived DNA equilibrium geometry and flexibility features; run via `upbm_regressor.py` on the trimmed, PWM-weighted uPBM data) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Provides the motif used to trim uPBM 35-mers to the motif region, reducing sequence variability before training)

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- Average R2 across the 66 mouse TFs should approximate the reported 0.69 ± 0.17
- Training must use 80% of data and R2 must be computed on the remaining 20% for each TF
- uPBM sequences must be trimmed to the MEME motif region before feature extraction
- PWM-matching high-affinity sequences carry 10x weight and low-affinity k-mers are undersampled
- Input file must match the template format `proteins/{protein}/{protein}_training.txt` required by `upbm_regressor.py`

## Limitations

- uPBM reproduction quality (R2 = 0.69 ± 0.17) is notably lower than gcPBM (R2 = 0.93 ± 0.02), reflecting greater noise/variability in uPBM data
- The transcription factor binding site concept is not uniquely defined — it depends on the noisy, low-resolution experimental technique used, so performance is dataset-dependent
- Extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic TF affinity, and DNAffinity predicts many potential binding sites without experimental in vivo evidence

## Evidence
