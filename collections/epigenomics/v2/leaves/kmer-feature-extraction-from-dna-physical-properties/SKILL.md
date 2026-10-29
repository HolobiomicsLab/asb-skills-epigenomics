---
name: kmer-feature-extraction-from-dna-physical-properties
description: Use when raw k-mer/sequence-affinity data (e.g. HT-SELEX cycle files,
  PBM intensity tables) must be converted into physicochemical feature vectors before
  training an ML affinity regressor.
when_to_use_negative:
- Input is already a feature table of structural parameters rather than raw k-mer/sequence-affinity
  data
- The goal is sequence-only deep learning affinity prediction (e.g. DeepBind-style
  CNNs), where nucleotide one-hot encoding rather than MD-derived DNA shape parameters
  is used
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Provides the MD-derived physical-property parameter database and the regressor
    scripts (selex_regressor.py, upbm_regressor.py, gcpbm_regressor.py) that consume
    the extracted k-mer features
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:c31141ce523ae4dbf269e359da8253a069f5cef26806273f10a00cb360975ef7
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# k-mer feature extraction from DNA physical properties

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes k-mer sequences as numerical feature vectors using DNA equilibrium geometry and flexibility parameters at base-pair-step (bps) resolution, derived from atomistic molecular dynamics simulations, so that a random forest regressor (DNAffinity) can predict transcription factor binding affinities from uPBM, gcPBM and HT-SELEX data.

## When to use

Use when raw k-mer/sequence-affinity data (e.g. HT-SELEX cycle files, PBM intensity tables) must be converted into physicochemical feature vectors before training an ML affinity regressor.

## When NOT to use

- Input is already a feature table of structural parameters rather than raw k-mer/sequence-affinity data
- The goal is sequence-only deep learning affinity prediction (e.g. DeepBind-style CNNs), where nucleotide one-hot encoding rather than MD-derived DNA shape parameters is used

## Inputs

- Preprocessed k-mer/affinity training files (e.g. proteins/{protein}/{cycle}.txt for HT-SELEX, {protein}_training.txt for uPBM, {protein}_{concentration}.txt and _freq_matrix_6.txt for gcPBM)
- MD-derived database of DNA physical-property parameters (equilibrium geometry and flexibility at base-pair-step resolution) from the DNAffinity repository

## Outputs

- Per-k-mer physicochemical feature vectors with associated experimental affinity labels
- Trained random forest regressor predictions with per-protein R2, Pearson correlation and MSE metrics (e.g. selex_r2_metrics.csv)

## How to apply

For each k-mer in the preprocessed training file (e.g. proteins/{protein}/{cycle}.txt for SELEX or {protein}_training.txt for uPBM), look up the MD-derived parameters for the DNA equilibrium geometry and flexibility, as defined by Olson, at base-pair-step resolution. These structural/mechanical values, not the nucleotide identities themselves, become the feature vector inputs. Feed the resulting feature vectors, paired with the experimental affinity values from the penultimate HT-SELEX cycle or PBM intensities, into the random forest regressor (selex_regressor.py / upbm_regressor.py), training on 80% and evaluating R2, Pearson correlation and MSE on the remaining 20%. Ensure input files match the repository templates before feature extraction, since the regressor parses them positionally.

## Related tools

- **DNAffinity** (Provides the MD-derived physical-property parameter database and the regressor scripts (selex_regressor.py, upbm_regressor.py, gcpbm_regressor.py) that consume the extracted k-mer features) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Features trace back one-to-one to k-mers present in the template-formatted input file; no k-mer is dropped or silently padded
- Downstream regressor reproduces published average R2: 0.93 ± 0.02 (gcPBM), 0.69 ± 0.17 (uPBM), and 0.63 ± 0.19 / 0.66 ± 0.19 (HT-SELEX 5-cycle / combined)
- Train/test split is 80/20 and held-out metrics (R2, Pearson, MSE) are computed per protein before aggregation
- For HT-SELEX, features are extracted from penultimate-cycle sequences and suspicious TFs with inconsistent enrichments across cycles have been removed by the SVM pre-filter

## Limitations

- The concept of a transcription factor binding site is not uniquely defined and depends on the noisy, low-resolution experimental technique used to detect it, preventing a universal predictor
- Feature extraction assumes template-compliant input files; the README requires checking 'Make sure you have all the required files (compare formatting with the templates)' before running any regressor
- Extrapolating in vitro-trained features to in vivo settings is complicated by factors beyond intrinsic transcription factor affinity, such as chromatin context

## Evidence
