---
name: transcription-factor-affinity-regression
description: Use when experimental TF-DNA binding data (uPBM training files, gcPBM
  concentration files, HT-SELEX per-cycle count files) must be converted into per-sequence
  affinity predictions with an R2-validated random-forest regressor.
when_to_use_negative:
- 'When extrapolating in vitro predictions to in vivo TFBS identification: this regressor
  alone does not model chromatin — combine it with yeast G1 nucleosome positioning
  maps instead'
- When the input is already a trained model or affinity prediction table rather than
  raw per-cycle/per-concentration sequence count files
- For experimental techniques other than uPBM, gcPBM and HT-SELEX, which the provided
  regressors are not configured to parse
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Provides the upbm_regressor.py, selex_regressor.py and gcpbm_regressor.py
    scripts that train the random-forest regressor on uPBM, HT-SELEX and gcPBM data
    respectively
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Used to trim uPBM sequences so that sequence variability is reduced before
    regressor training
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:13b9cfb25466d485dd7a56e6d721993bca5088cb1de10cd805bacbb64aab8ab2
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transcription-factor-affinity-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the DNAffinity random-forest regressor on uPBM, gcPBM or HT-SELEX in vitro binding data using DNA structural/mechanical features derived from atomistic molecular dynamics simulations, to produce per-sequence transcription factor binding affinity predictions evaluated by R2 on a held-out 20% split.

## When to use

Use when experimental TF-DNA binding data (uPBM training files, gcPBM concentration files, HT-SELEX per-cycle count files) must be converted into per-sequence affinity predictions with an R2-validated random-forest regressor.

## When NOT to use

- When extrapolating in vitro predictions to in vivo TFBS identification: this regressor alone does not model chromatin — combine it with yeast G1 nucleosome positioning maps instead
- When the input is already a trained model or affinity prediction table rather than raw per-cycle/per-concentration sequence count files
- For experimental techniques other than uPBM, gcPBM and HT-SELEX, which the provided regressors are not configured to parse

## Inputs

- proteins/{protein}/{protein}_training.txt (uPBM preprocessed training file)
- proteins/{protein}/{cycle}.txt (HT-SELEX per-cycle sequence count file)
- proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt (gcPBM data)

## Outputs

- Per-sequence affinity predictions (selex_affinity_predictions.csv)
- Determination coefficient R2 on the held-out 20% of sequences
- Random-forest feature importances aggregatable into sequence composition, indirect readout and direct readout classes

## How to apply

First verify the required input files exist and match the repository templates: proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX, and proteins/{protein}/{protein}_{concentration}.txt plus proteins/{protein}/_freq_matrix_6.txt for gcPBM. Launch the corresponding regressor (upbm_regressor.py, selex_regressor.py, gcpbm_regressor.py) passing the protein name as the 1st argument and, for selex/gcpbm, the cycle number or concentration as the 2nd argument; run without arguments to execute the bundled test case. The regressor trains a random forest on 80% of the sequences using descriptors of DNA equilibrium geometry and flexibility at base-pair-step resolution from MD simulations. For HT-SELEX, compute correlations of selected sequence counts across cycles and train an SVM discriminant to flag inconsistent enrichment cases before retraining on the filtered set. Evaluate the determination coefficient R2 (plus Pearson correlation and MSE) between predictions and experimental affinities on the remaining 20% held-out sequences and write predictions to selex_affinity_predictions.csv.

## Related tools

- **DNAffinity** (Provides the upbm_regressor.py, selex_regressor.py and gcpbm_regressor.py scripts that train the random-forest regressor on uPBM, HT-SELEX and gcPBM data respectively) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Used to trim uPBM sequences so that sequence variability is reduced before regressor training)

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Average R2 should fall in the published ranges: 0.93 ± 0.02 for gcPBM, 0.69 ± 0.17 for uPBM, and ~0.66 ± 0.19 unfiltered HT-SELEX (rising to 0.70 ± 0.14 after SVM filtering)
- Verify the 80/20 split: regressor trained on 80% of sequences, R2 computed only on the remaining 20%
- Check that predictions are written to selex_affinity_predictions.csv with correct formatting
- Feature importances should aggregate into the three classes (sequence composition, indirect readout, direct readout) and their relative ordering should differ between uPBM, gcPBM and HT-SELEX runs
- For HT-SELEX, confirm SVM-flagged inconsistent-cycle cases were removed before retraining (dramatic reduction of low-R2 cases)

## Limitations

- The transcription factor binding site concept is not uniquely defined and depends on the noisy, low-resolution experimental technique used to detect it, so no universal predictor exists
- Extrapolating in vitro predictions to in vivo settings requires factors beyond intrinsic transcription factor affinity (e.g. chromatin structure)
- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists
- The relative importance of feature classes depends dramatically on the type of experimental data used for training, so results are not transferable across uPBM, gcPBM and HT-SELEX

## Evidence
