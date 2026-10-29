---
name: pbm-selex-training-data-preparation
description: Use when raw uPBM, gcPBM, or HT-SELEX data must be formatted into DNAffinity's
  per-protein training files before running upbm_regressor.py, selex_regressor.py,
  or gcpbm_regressor.py.
when_to_use_negative:
- The goal is in vivo TFBS prediction — that requires complementing DNAffinity predictions
  with yeast G1 nucleosome positioning maps, not training-data formatting
- You already have a trained regressor and only want to score affinities or peaks
  (e.g. PB-exo/ChIP-exo peaks along the yeast genome)
- Input data are not from uPBM, gcPBM, or HT-SELEX assays and cannot be cast into
  the required file templates
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: DNAffinity
  role: Provides the regressor scripts (upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py)
    and required training-file formats; tool and feature-parameter database on GitHub
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Used to trim uPBM sequences so the variability of the sequence diminishes
    before training
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:f58b815f7b14eb442482b8ee72d68625f50cf5bb12ef0b99758dfd2a9acd5a0a
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pbm-selex-training-data-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares training data files for the DNAffinity random-forest regressors from three experimental assay types (uPBM, gcPBM, HT-SELEX), applying assay-specific preprocessing (MEME-based trimming, SVM-based inconsistency filtering) so that each {method}_regressor.py can be launched with the correct input files and arguments.

## When to use

Use when raw uPBM, gcPBM, or HT-SELEX data must be formatted into DNAffinity's per-protein training files before running upbm_regressor.py, selex_regressor.py, or gcpbm_regressor.py.

## When NOT to use

- The goal is in vivo TFBS prediction — that requires complementing DNAffinity predictions with yeast G1 nucleosome positioning maps, not training-data formatting
- You already have a trained regressor and only want to score affinities or peaks (e.g. PB-exo/ChIP-exo peaks along the yeast genome)
- Input data are not from uPBM, gcPBM, or HT-SELEX assays and cannot be cast into the required file templates

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM training sequences/affinities)
- proteins/{protein}/{cycle}.txt (HT-SELEX cycle files)
- proteins/{protein}/{protein}_{concentration}.txt (gcPBM data for a given concentration)
- proteins/{protein}/_freq_matrix_6.txt (gcPBM frequency matrix)
- Protein name, HT-SELEX cycle number, gcPBM concentration (regressor CLI arguments)

## Outputs

- Formatted training inputs consumed by {method}_regressor.py
- Trained random-forest regressor per assay
- Per-class feature importances (sequence composition, indirect readout, direct readout) written to feature_importances.csv

## How to apply

First verify the required files per assay against the templates: proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX, and proteins/{protein}/{protein}_{concentration}.txt together with proteins/{protein}/_freq_matrix_6.txt for gcPBM. Apply assay-specific preprocessing: for uPBM trim the sequences based on the MEME suite result so that sequence variability diminishes; for HT-SELEX use SVM to remove cases showing inconsistency between enrichments at different cycles, since these are suspicious. The features fed to the regressors are DNA physical properties at base-pair-step resolution (equilibrium geometry and flexibility per Olson's definitions) derived from MD simulations, aggregated into sequence-composition, indirect-readout, and direct-readout classes. Then launch the corresponding regressor script passing the protein name as the 1st argument, plus the cycle number for selex_regressor.py or the concentration for gcpbm_regressor.py. Extract the random-forest feature importances and aggregate them per feature class to compare how importance shifts between gcPBM, uPBM and HT-SELEX training runs.

## Related tools

- **DNAffinity** (Provides the regressor scripts (upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py) and required training-file formats; tool and feature-parameter database on GitHub) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Used to trim uPBM sequences so the variability of the sequence diminishes before training)

## Examples

```
python upbm_regressor.py Gata4
```

## Evaluation signals

- All required files exist and match the template formatting for the chosen assay before launching the regressor
- HT-SELEX cases with inconsistent enrichments across cycles have been removed by the SVM filtering step before training
- uPBM sequences have been trimmed based on the MEME suite result
- The regressor runs with the protein name as the 1st argument and the cycle number (selex) or concentration (gcPBM) as the 2nd
- Relative ordering/magnitude of the three feature classes (sequence composition, indirect readout, direct readout) differs between uPBM, gcPBM and HT-SELEX runs, consistent with the finding that the most informative parameters depend on the sequence variability explored by the experiment

## Limitations

- Transcription factor binding site concept is not uniquely defined — it depends on the intrinsically noisy, low-resolution experimental technique used to detect it, so training data from different assays yield different informative parameters
- HT-SELEX enrichment values can be inconsistent across cycles; such cases must be filtered out or they will bias training and testing
- The article reports no changelog and does not name validation experiments in the discussion, so expected quantitative thresholds for this comparison are not provided

## Evidence
