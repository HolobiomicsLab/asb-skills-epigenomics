---
name: regression-model-evaluation-r2
description: Use when a binding-affinity regression model (e.g. DNAffinity on gcPBM/uPBM/HT-SELEX
  data) needs quantitative validation of prediction quality via R2 on a held-out test
  partition.
when_to_use_negative:
- The task is classification or TFBS identification rather than reproducing quantitative
  binding intensities with a regressor
- The dataset is already a set of model scores with no experimental reference values
  to compute R2 against
- Sequence data has not been preprocessed (e.g. HT-SELEX cases with inconsistent enrichment
  between cycles not yet removed by SVM, or uPBM sequences not trimmed by MEME suite)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Random-forest regressor trained on DNA equilibrium-geometry and flexibility
    features from MD simulations; its gcpbm_regressor.py script is evaluated by computing
    R2 on the held-out 20% split
  repo: https://github.com/Jalbiti/DNAffinity
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:fd297c4f864b8668472f7fc5d972594af949bf7a3c0bcdb115a989905f51d4a6
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# regression-model-evaluation-r2

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate a trained DNAffinity random-forest regressor by computing the determination coefficient (R2) between predictions and experimental binding intensities on a held-out 20% test split, following the paper's 80/20 training/testing protocol for datasets such as gcPBM.

## When to use

Use when a binding-affinity regression model (e.g. DNAffinity on gcPBM/uPBM/HT-SELEX data) needs quantitative validation of prediction quality via R2 on a held-out test partition.

## When NOT to use

- The task is classification or TFBS identification rather than reproducing quantitative binding intensities with a regressor
- The dataset is already a set of model scores with no experimental reference values to compute R2 against
- Sequence data has not been preprocessed (e.g. HT-SELEX cases with inconsistent enrichment between cycles not yet removed by SVM, or uPBM sequences not trimmed by MEME suite)

## Inputs

- Trained DNAffinity random-forest regressor (e.g. gcpbm_regressor.py) with equilibrium-geometry and flexibility features
- gcPBM concentration-specific intensity file proteins/{protein}/{protein}_{concentration}.txt
- 6-mer frequency matrix proteins/{protein}/_freq_matrix_6.txt
- 80/20 train/test split of the binding dataset (gcPBM, uPBM, or HT-SELEX intensities)

## Outputs

- Determination coefficient (R2) between predictions and experimental values on the held-out 20%
- Per-protein predictions saved to gcpbm_regressor_results.csv
- Aggregated mean ± standard deviation of R2 across proteins/datasets

## How to apply

After training the random-forest regressor (using DNA equilibrium-geometry and flexibility features from MD simulations) on 80% of the data, compute the determination coefficient (R2) between predictions and the experimental values on the remaining 20%, as described in the paper's Materials and Methods. For gcPBM, run this per protein (e.g. Mad, Max, Myc, CBF1) using gcpbm_regressor.py with the protein name as the 1st argument and the concentration as the 2nd, after verifying that proteins/{protein}/{protein}_{concentration}.txt and proteins/{protein}/_freq_matrix_6.txt match the package template formatting. Aggregate per-protein R2 values as mean ± standard deviation and compare against the paper's benchmarks (gcPBM average R2 = 0.93 ± 0.02; CBF1 R2 = 0.80). Save per-protein predictions and R2 scores to gcpbm_regressor_results.csv. Judging correctness: per-protein R2 values should fall within the reported ranges (e.g. CBF1 ≈ 0.80), and the cross-dataset averages should be near 0.93 ± 0.02 for gcPBM and 0.69 ± 0.17 for uPBM.

## Related tools

- **DNAffinity** (Random-forest regressor trained on DNA equilibrium-geometry and flexibility features from MD simulations; its gcpbm_regressor.py script is evaluated by computing R2 on the held-out 20% split) — https://github.com/Jalbiti/DNAffinity

## Examples

```
python gcpbm_regressor.py CBF1 <concentration>
```

## Evaluation signals

- Per-protein gcPBM R2 values reproduce the paper's benchmarks: average R2 = 0.93 ± 0.02 across gcPBM-trained TFs and CBF1 R2 = 0.80
- uPBM evaluation yields average R2 = 0.69 ± 0.17 when applied with the same protocol
- The evaluation uses an 80% training / 20% testing split with R2 computed only on the held-out 20%
- Per-protein predictions and R2 scores are persisted to gcpbm_regressor_results.csv
- Input files ({protein}_{concentration}.txt and _freq_matrix_6.txt) match the formatting of the package templates before running

## Limitations

- The concept of a transcription factor binding site is not uniquely defined, as it depends on the intrinsically noisy and low-resolution experimental technique used to detect it, preventing a universal predictor
- R2 on in vitro data does not guarantee in vivo transferability: other factors besides intrinsic transcription factor affinity affect in vivo binding
- DNAffinity predicts many potential binding sites where no experimental evidence of in vivo binding exists

## Evidence
