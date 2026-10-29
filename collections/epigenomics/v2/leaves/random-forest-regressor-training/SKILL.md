---
name: random-forest-regressor-training
description: Use when k-mer binding data (e.g. HT-SELEX penultimate-cycle affinities,
  uPBM/gcPBM intensities) must be converted into per-protein affinity predictions
  via regression on MD-derived DNA shape features, with R2/Pearson/MSE evaluation.
when_to_use_negative:
- Input is raw sequencing reads or unprocessed peak data — the regressor requires
  preprocessed, template-formatted affinity/k-mer training files
- The goal is de novo motif discovery rather than affinity regression on already-quantified
  binding data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Provides the random forest regressor scripts (selex_regressor.py, upbm_regressor.py,
    gcpbm_regressor.py) and the MD-derived feature parameter database used to train
    and evaluate affinity predictions
  repo: https://github.com/Jalbiti/DNAffinity
- name: scikit-learn
  role: Implements the random forest regressor and the SVM-based check for inconsistent
    HT-SELEX enrichment cases
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:f0ca04d372541d27ff65c1ef5c1ba5f40e731100079c35fa93c954896d58ea1b
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# random-forest-regressor-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a random forest regressor on DNA physical-property features (equilibrium geometry and flexibility from MD simulations) to predict transcription factor binding affinities from in vitro datasets such as HT-SELEX, uPBM, and gcPBM, as implemented in DNAffinity.

## When to use

Use when k-mer binding data (e.g. HT-SELEX penultimate-cycle affinities, uPBM/gcPBM intensities) must be converted into per-protein affinity predictions via regression on MD-derived DNA shape features, with R2/Pearson/MSE evaluation.

## When NOT to use

- Input is raw sequencing reads or unprocessed peak data — the regressor requires preprocessed, template-formatted affinity/k-mer training files
- The goal is de novo motif discovery rather than affinity regression on already-quantified binding data

## Inputs

- HT-SELEX per-protein per-cycle training file: proteins/{protein}/{cycle}.txt (template formatting required)
- uPBM training file: proteins/{protein}/{protein}_training.txt
- gcPBM files: proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt
- MD-derived DNA physical-property parameter database (equilibrium geometry and flexibility at base-pair-step resolution)

## Outputs

- Per-protein predictions on the held-out 20% with R2, Pearson correlation, and MSE metrics
- Dataset-level aggregate metrics (e.g. average R2 for 5-cycle, 7-cycle, and combined HT-SELEX)

## How to apply

Collect the per-protein training files in the required template format (for HT-SELEX, `proteins/{protein}/{cycle}.txt`), pre-filtering TF cases where SVM detects inconsistency between enrichments at different SELEX cycles so suspicious cases do not bias training or testing. Extract DNA physical-property features — equilibrium geometry and flexibility at base-pair-step resolution from the MD-derived parameter database — for the k-mers. Train the random forest regressor on 80% of the k-mer feature vectors against the affinity values (e.g. penultimate-cycle HT-SELEX affinities), invoking `selex_regressor.py` with the protein name and cycle number as arguments. Evaluate predictions on the held-out 20% by computing R2, Pearson correlation, and MSE per protein, then aggregate per-protein R2 values into dataset-level averages (e.g. 5-cycle, 7-cycle, and combined HT-SELEX averages).

## Related tools

- **DNAffinity** (Provides the random forest regressor scripts (selex_regressor.py, upbm_regressor.py, gcpbm_regressor.py) and the MD-derived feature parameter database used to train and evaluate affinity predictions) — https://github.com/Jalbiti/DNAffinity
- **scikit-learn** (Implements the random forest regressor and the SVM-based check for inconsistent HT-SELEX enrichment cases)

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Computed average R2 matches published reference values within the reported standard deviation: 0.63 ± 0.19 (5-cycle HT-SELEX), 0.66 ± 0.19 (combined total)
- 80/20 train/test split is respected: R2 is computed only on the held-out 20% per protein
- Input files match the template formatting required by {method}_regressor.py before running
- SVM pre-filter removed TF cases with inconsistency between enrichments at different SELEX cycles prior to training
- Per-protein R2, Pearson correlation, and MSE are all computed and recorded

## Limitations

- TF cases showing inconsistency between enrichments at different HT-SELEX cycles must be removed, as they are suspicious and would bias training/testing
- The transcription factor binding site concept is not uniquely defined — it depends on the noisy, low-resolution experimental technique used to detect it, preventing a universal predictor
- Random-forest affinity predictions can flag many potential binding sites with no experimental in vivo binding evidence; extrapolating in vitro predictions to in vivo settings is complicated by factors beyond intrinsic affinity

## Evidence
