---
name: dna-mechanical-property-feature-classification
description: Use when random-forest models have been trained on multiple experimental
  binding datasets (gcPBM, uPBM, HT-SELEX) and you need to compare which DNA-property
  feature classes drive predictive power in each run.
when_to_use_negative:
- You only have a single trained model on one dataset — the skill's purpose is cross-dataset
  comparison of feature-class importance, which requires runs on different experimental
  datasets.
- Input is raw sequence/enrichment data not yet preprocessed into the regressor training
  files (e.g. uPBM sequences not trimmed via MEME suite, HT-SELEX cycles not filtered
  for inconsistent enrichments).
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Provides the random forest regressor scripts (upbm_regressor.py, selex_regressor.py,
    gcpbm_regressor.py) whose feature importances are aggregated into the three feature
    classes
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:4b99c8d26b529c930cd1b66c09700b8f944d4e0eabfd5f25c26ae336a2f229bf
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dna-mechanical-property-feature-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates individual random-forest feature importances from DNAffinity regressors into three feature classes — sequence composition, indirect readout, and direct readout — to determine whether class importance depends on the experimental training dataset (gcPBM, uPBM, HT-SELEX).

## When to use

Use when random-forest models have been trained on multiple experimental binding datasets (gcPBM, uPBM, HT-SELEX) and you need to compare which DNA-property feature classes drive predictive power in each run.

## When NOT to use

- You only have a single trained model on one dataset — the skill's purpose is cross-dataset comparison of feature-class importance, which requires runs on different experimental datasets.
- Input is raw sequence/enrichment data not yet preprocessed into the regressor training files (e.g. uPBM sequences not trimmed via MEME suite, HT-SELEX cycles not filtered for inconsistent enrichments).

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM training data)
- proteins/{protein}/{cycle}.txt (HT-SELEX per-cycle data)
- proteins/{protein}/{protein}_{concentration}.txt (gcPBM concentration data)
- proteins/{protein}/_freq_matrix_6.txt (gcPBM frequency matrix)
- Trained DNAffinity random forest regressors (upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py)

## Outputs

- feature_importances.csv with per-class (sequence composition, indirect readout, direct readout) importance values for uPBM, gcPBM and HT-SELEX runs

## How to apply

Train the DNAffinity random forest separately on uPBM, HT-SELEX, and gcPBM data by launching upbm_regressor.py (protein name as 1st argument), selex_regressor.py (protein name 1st, cycle number 2nd), and gcpbm_regressor.py (protein name 1st, concentration 2nd). Verify the required inputs are present first: proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX, and proteins/{protein}/{protein}_{concentration}.txt plus proteins/{protein}/_freq_matrix_6.txt for gcPBM. Extract the random-forest feature importances from each trained model and aggregate the individual features into the three classes: sequence composition, indirect readout, and direct readout. Write per-class importance values for all methods to feature_importances.csv and compare the relative ordering/magnitude of classes across datasets.

## Related tools

- **DNAffinity** (Provides the random forest regressor scripts (upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py) whose feature importances are aggregated into the three feature classes) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python upbm_regressor.py Gata4; python selex_regressor.py Gata4 4; python gcpbm_regressor.py Gata4 12
```

## Evaluation signals

- feature_importances.csv contains importance values for all three classes (sequence composition, indirect readout, direct readout) for all three methods (uPBM, gcPBM, HT-SELEX).
- The relative ordering/magnitude of the three feature classes differs between the uPBM, gcPBM and HT-SELEX runs, consistent with the criterion that the most informative parameters depend on the sequence variability explored by the experiment.
- Each trained regressor produced a model with importances extractable (i.e. training ran successfully on the correctly formatted training files).
- Sum of aggregated per-class importances equals the total feature importance of the corresponding random forest model (no features lost or double-counted in the class aggregation).

## Limitations

- Feature-class importance depends dramatically on the type of experimental data used for training, so importances from one dataset (e.g. gcPBM) cannot be generalized to another (e.g. HT-SELEX).
- The paper notes that the most informative parameters describing binding mechanisms largely depend on the sequence variability explored by the experiment, so results are dataset-specific by design.
- The concept of transcription factor binding site is not uniquely defined, as it deeply depends on the intrinsically noisy and low-resolution experimental technique used to detect it, limiting interpretation of class importances.

## Evidence
