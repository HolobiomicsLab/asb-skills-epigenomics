---
name: random-forest-feature-importance-extraction
description: Use when comparing how training data source (uPBM vs gcPBM vs HT-SELEX)
  changes the relative importance of DNA feature classes in the DNAffinity random
  forest regressor, or when auditing which physical parameters drive affinity predictions.
when_to_use_negative:
- Input is already an aggregated feature-importance table (e.g. an existing feature_importances.csv)
  — only aggregate from raw per-feature importances of newly trained models
- The goal is affinity prediction accuracy (R2/Pearson/MSE evaluation) rather than
  interpretability of feature-class contributions
- Training data come from techniques outside uPBM/gcPBM/HT-SELEX; the regressor scripts
  and preprocessing formats are specific to those three experimental techniques
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Provides the random forest regressor scripts (upbm_regressor.py, selex_regressor.py,
    gcpbm_regressor.py) trained on MD-derived DNA structural and flexibility parameters
    whose importances are extracted
  repo: https://github.com/Jalbiti/DNAffinity
- name: DNAShapeR
  role: Shape-based ML regressor used as a comparison baseline when benchmarking DNAffinity
    predictions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:02f46bc772df1b142862d43cbef6580687a0712cf795e8d45b98c94a3660b390
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
  - DNAShapeR
  - DNAffinity
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# random-forest-feature-importance-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract and aggregate random-forest feature importances from DNAffinity regressor runs trained on different experimental datasets (uPBM, gcPBM, HT-SELEX), collapsing individual DNA physical-property features into the three feature classes: sequence composition, indirect readout, and direct readout.

## When to use

Use when comparing how training data source (uPBM vs gcPBM vs HT-SELEX) changes the relative importance of DNA feature classes in the DNAffinity random forest regressor, or when auditing which physical parameters drive affinity predictions.

## When NOT to use

- Input is already an aggregated feature-importance table (e.g. an existing feature_importances.csv) — only aggregate from raw per-feature importances of newly trained models
- The goal is affinity prediction accuracy (R2/Pearson/MSE evaluation) rather than interpretability of feature-class contributions
- Training data come from techniques outside uPBM/gcPBM/HT-SELEX; the regressor scripts and preprocessing formats are specific to those three experimental techniques

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM training data)
- proteins/{protein}/{cycle}.txt (HT-SELEX cycle data)
- proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt (gcPBM data)
- Trained random forest regressor models from upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py

## Outputs

- feature_importances.csv with per-class (sequence composition, indirect readout, direct readout) importance values for uPBM, gcPBM and HT-SELEX runs

## How to apply

First verify the training inputs per method are present and correctly formatted: proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX, and proteins/{protein}/{protein}_{concentration}.txt plus proteins/{protein}/_freq_matrix_6.txt for gcPBM. Train the random forest by launching upbm_regressor.py (protein as 1st argument), selex_regressor.py (protein as 1st argument, cycle number as 2nd), and gcpbm_regressor.py (protein as 1st argument, concentration as 2nd). Then extract the feature importances from each trained model and aggregate individual base-pair-step structural/flexibility features into the three classes: sequence composition, indirect readout, and direct readout. Write the per-class importance values for all methods to feature_importances.csv. Validation: confirm the relative ordering/magnitude of the three classes differs across the uPBM, gcPBM and HT-SELEX runs, consistent with the paper's finding that the most informative parameters depend on the sequence variability explored by the experiment.

## Related tools

- **DNAffinity** (Provides the random forest regressor scripts (upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py) trained on MD-derived DNA structural and flexibility parameters whose importances are extracted) — https://github.com/Jalbiti/DNAffinity
- **DNAShapeR** (Shape-based ML regressor used as a comparison baseline when benchmarking DNAffinity predictions)

## Examples

```
python upbm_regressor.py Gata4; python selex_regressor.py Gata4 4; python gcpbm_regressor.py <protein> <concentration>
```

## Evaluation signals

- feature_importances.csv contains per-class importance rows for all three training runs (uPBM, gcPBM, HT-SELEX) covering the same three classes: sequence composition, indirect readout, direct readout
- Class importances sum consistently to 1 within each trained model after aggregation from individual feature importances
- The relative ordering or magnitude of the three feature classes differs between the uPBM, gcPBM and HT-SELEX runs
- Each regressor ran successfully on correctly formatted inputs (e.g. _freq_matrix_6.txt present for gcPBM, cycle number supplied for selex_regressor.py)
- Per-model predictive quality is in the paper's reported ranges (e.g. average R2 = 0.93 ± 0.02 for gcPBM, 0.69 ± 0.17 for uPBM) so importances come from non-degenerate models

## Limitations

- The most informative parameters depend on the sequence variability explored by the experiment, so feature-class importances are dataset-specific and not transferable across experimental techniques
- The concept of a transcription factor binding site is not uniquely defined — it depends on the intrinsically noisy and low-resolution experimental technique used to detect it — so importance comparisons across datasets reflect those technique-specific biases
- HT-SELEX training requires SVM filtering of cases with inconsistency between enrichments at different cycles, as these are suspicious cases that could bias the trained model

## Evidence
