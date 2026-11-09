---
name: pearson-correlation-benchmark-comparison
description: Use when a trained TF-affinity model must be evaluated across experimental
  platforms (e.g. HT-SELEX training, uPBM testing) and ranked against published predictors
  via per-TF Pearson correlations.
when_to_use_negative:
- The model is evaluated on the same platform it was trained on (e.g. gcPBM R2 = 0.93
  ± 0.02) rather than in a cross-dataset HT-SELEX-to-uPBM transfer setting
- Input files lack the required template formatting (missing proteins/{protein}/{cycle}.txt
  or {protein}_training.txt), since the regressors depend on specific file layouts
- No competing-method predictions (DeepBind, BEESEM, BindSpace) are available for
  the same TF set, so a like-for-like correlation table cannot be assembled
edam_topics:
- http://edamontology.org/topic_0208
tools:
- name: DNAffinity
  role: Random-forest regressor on DNA physical-property features; trained on HT-SELEX
    and applied to uPBM via selex_regressor.py and upbm_regressor.py
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Motif-based trimming of uPBM sequences to reduce sequence variability before
    evaluation
- name: DeepBind
  role: Competing CNN-based predictor trained and tested on the same TFs; reported
    average Pearson 0.35
- name: BEESEM
  role: Competing method trained and tested on the same TFs; reported average Pearson
    0.36
- name: BindSpace
  role: Competing method trained and tested on the same TFs; reported average Pearson
    0.20
- name: DeepSELEX
  role: Competing shape-based ML predictor; reported average Pearson 0.41 in the HT-SELEX-to-uPBM
    transfer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:6cbce095a8b98b6257d501074aac472a5634c12821d475d86be745cdb41d1326
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
  - BEESEM
  - BindSpace
  - DNAffinity
  - DeepBind
  - DeepSELEX
  - MEME suite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pearson-correlation-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-dataset benchmarking protocol that trains DNAffinity on HT-SELEX per-cycle enrichments, transfers the model to uPBM data, and compares average Pearson correlations against DeepSELEX, BEESEM, DeepBind, and BindSpace on the same TF set.

## When to use

Use when a trained TF-affinity model must be evaluated across experimental platforms (e.g. HT-SELEX training, uPBM testing) and ranked against published predictors via per-TF Pearson correlations.

## When NOT to use

- The model is evaluated on the same platform it was trained on (e.g. gcPBM R2 = 0.93 ± 0.02) rather than in a cross-dataset HT-SELEX-to-uPBM transfer setting
- Input files lack the required template formatting (missing proteins/{protein}/{cycle}.txt or {protein}_training.txt), since the regressors depend on specific file layouts
- No competing-method predictions (DeepBind, BEESEM, BindSpace) are available for the same TF set, so a like-for-like correlation table cannot be assembled

## Inputs

- HT-SELEX per-cycle enrichment files: proteins/{protein}/{cycle}.txt
- uPBM training files: proteins/{protein}/{protein}_training.txt
- Protein/TF name and HT-SELEX cycle number passed as command-line arguments
- Predicted and experimental uPBM affinities per TF (for correlation computation)

## Outputs

- Per-TF Pearson correlation between predicted and experimental uPBM affinities
- Average Pearson correlation per method (DNAffinity, DeepSELEX, BEESEM, DeepBind, BindSpace)
- cross_dataset_correlations.csv (per-method correlation table)

## How to apply

Collect HT-SELEX per-cycle enrichment files at proteins/{protein}/{cycle}.txt and uPBM training files at proteins/{protein}/{protein}_training.txt, verifying formatting against the repository templates. Apply SVM filtering to discard HT-SELEX cases showing inconsistency between enrichments at different cycles, and trim uPBM sequences using the MEME suite motif result to reduce sequence variability. Launch selex_regressor.py with the protein name as the 1st argument and the cycle number as the 2nd argument to extract DNA physical-property features (equilibrium geometry and flexibility at base-pair step resolution from MD simulations) and train the random forest regressor on HT-SELEX affinities; then run upbm_regressor.py to apply the HT-SELEX-trained model to uPBM data. Compute the Pearson correlation between predicted and experimental uPBM affinities per TF, average across TFs, and assemble the per-method table (target: DNAffinity 0.47 vs DeepSELEX 0.41, BEESEM 0.36, DeepBind 0.35, BindSpace 0.20), saving it as cross_dataset_correlations.csv.

## Related tools

- **DNAffinity** (Random-forest regressor on DNA physical-property features; trained on HT-SELEX and applied to uPBM via selex_regressor.py and upbm_regressor.py) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Motif-based trimming of uPBM sequences to reduce sequence variability before evaluation)
- **DeepBind** (Competing CNN-based predictor trained and tested on the same TFs; reported average Pearson 0.35)
- **BEESEM** (Competing method trained and tested on the same TFs; reported average Pearson 0.36)
- **BindSpace** (Competing method trained and tested on the same TFs; reported average Pearson 0.20)
- **DeepSELEX** (Competing shape-based ML predictor; reported average Pearson 0.41 in the HT-SELEX-to-uPBM transfer)

## Examples

```
python selex_regressor.py Gata4 4 && python upbm_regressor.py Gata4
```

## Evaluation signals

- DNAffinity average Pearson correlation in the HT-SELEX-to-uPBM transfer reproduces the reported value of ~0.47
- DNAffinity's average Pearson exceeds DeepSELEX (0.41), BEESEM (0.36), DeepBind (0.35) and BindSpace (0.20) on the same TF set
- Per-TF Pearson correlations are computed between predicted and experimental uPBM affinities for every TF before averaging
- cross_dataset_correlations.csv contains one correlation entry per method with the expected method names and values
- Input files match the repository templates (proteins/{protein}/{cycle}.txt and proteins/{protein}/{protein}_training.txt) before running the regressors

## Limitations

- The transcription factor binding site concept is not uniquely defined, as it depends on the intrinsically noisy and low-resolution experimental technique used to detect it, preventing a universal predictor
- SVM filtering discards HT-SELEX cases with inconsistency between enrichments at different cycles, so the benchmark TF set excludes suspicious cases
- The benchmark is platform-specific: comparisons were run trained and tested on the same TFs; values do not transfer to other dataset pairs without re-running the pipeline

## Evidence

- [other] Average Pearson of 0.47 for HT-SELEX-to-uPBM transfer: [evidence span withheld — anchor status: not_found]
