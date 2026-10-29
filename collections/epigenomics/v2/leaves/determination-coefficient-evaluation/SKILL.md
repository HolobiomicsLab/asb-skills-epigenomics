---
name: determination-coefficient-evaluation
description: Use when assessing how well a regression model reproduces experimental
  TF–DNA binding affinities (uPBM, gcPBM, HT-SELEX k-mer data) and you need a comparable
  average R2 benchmark against the DNAffinity paper.
when_to_use_negative:
- Data is classification-only (e.g. binder/non-binder labels) with no continuous affinity
  values to regress, so R2 is not meaningful
- The question is peak or TFBS localization on the genome (PB-exo/ChIP-exo, nucleosome-map-based
  in vivo prediction) rather than affinity-value regression accuracy
- Sequences have not been preprocessed per technique (e.g. uPBM 35-mers not trimmed
  to the MEME motif region, HT-SELEX inconsistent cases not SVM-filtered), so results
  will not be comparable to the published benchmarks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DNAffinity
  role: Random-forest regressor trained on MD-derived DNA physical properties whose
    predictions are scored with R2 on held-out data
  repo: https://github.com/Jalbiti/DNAffinity
- name: scikit-learn
  role: Provides the random forest regressor and metric computation (R2, Pearson,
    MSE) underlying the evaluation
- name: MEME suite
  role: Supplies motif-region trimming of uPBM sequences so the evaluated affinities
    match the paper's uPBM preprocessing
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:db00230b839c2e41d5e62f5f567e5e82350b48ce1b71b8d796ecbeb4de69e7fe
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# determination-coefficient-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate DNAffinity random-forest affinity predictions by splitting each dataset 80/20, training on 80%, and computing the determination coefficient (R2), Pearson correlation, and MSE on the held-out 20%, then aggregating per-TF R2 values into dataset-level averages with standard deviations for comparison against published benchmarks.

## When to use

Use when assessing how well a regression model reproduces experimental TF–DNA binding affinities (uPBM, gcPBM, HT-SELEX k-mer data) and you need a comparable average R2 benchmark against the DNAffinity paper.

## When NOT to use

- Data is classification-only (e.g. binder/non-binder labels) with no continuous affinity values to regress, so R2 is not meaningful
- The question is peak or TFBS localization on the genome (PB-exo/ChIP-exo, nucleosome-map-based in vivo prediction) rather than affinity-value regression accuracy
- Sequences have not been preprocessed per technique (e.g. uPBM 35-mers not trimmed to the MEME motif region, HT-SELEX inconsistent cases not SVM-filtered), so results will not be comparable to the published benchmarks

## Inputs

- Per-TF training files with k-mer sequences and experimental binding affinities (e.g. proteins/{protein}/{protein}_training.txt for uPBM, proteins/{protein}/{cycle}.txt for HT-SELEX)
- Random-forest regressor affinity predictions on the held-out 20% of k-mers
- Experimental affinity values paired one-to-one with the predicted values

## Outputs

- Per-protein determination coefficient (R2), Pearson correlation, and MSE values
- Dataset-level average R2 with standard deviation (e.g. written to selex_r2_metrics.csv)
- Affinity predictions CSV (e.g. selex_affinity_predictions.csv)

## How to apply

Train the random-forest regressor on 80% of the per-protein k-mer feature vectors (MD-derived DNA equilibrium geometry and flexibility properties at base-pair-step resolution) and compute the determination coefficient R2 between predictions and experimental affinity values on the remaining 20%. Repeat per transcription factor and aggregate per-protein R2 into dataset-level averages and standard deviations: 0.93 ± 0.02 for gcPBM, 0.69 ± 0.17 for uPBM (after MEME-suite motif trimming, undersampling, and 10x weighting of PWM-matching sequences), and 0.63 ± 0.19 / 0.71 ± 0.21 / 0.66 ± 0.19 combined for the 5-cycle, 7-cycle, and total HT-SELEX datasets. For HT-SELEX, first apply the SVM discriminant on cross-cycle enrichment correlations to remove inconsistent cases, which raises the average to 0.70 ± 0.14. Compare computed averages against these published references, treating agreement within the reported standard deviation as the acceptance criterion.

## Related tools

- **DNAffinity** (Random-forest regressor trained on MD-derived DNA physical properties whose predictions are scored with R2 on held-out data) — https://github.com/Jalbiti/DNAffinity
- **scikit-learn** (Provides the random forest regressor and metric computation (R2, Pearson, MSE) underlying the evaluation)
- **MEME suite** (Supplies motif-region trimming of uPBM sequences so the evaluated affinities match the paper's uPBM preprocessing)

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Computed average R2 falls within one reported standard deviation of the published benchmark: 0.93 ± 0.02 (gcPBM), 0.69 ± 0.17 (uPBM), 0.66 ± 0.19 combined (HT-SELEX unfiltered)
- SVM-filtered HT-SELEX average R2 reaches ~0.70 ± 0.14 with a dramatic reduction of cases with low R2 compared to the unfiltered 0.66 ± 0.19 baseline
- Per-protein R2, Pearson correlation, and MSE are all written to selex_r2_metrics.csv for every protein evaluated
- Training and testing fractions match the paper's 80%/20% split for each dataset (gcPBM, uPBM, HT-SELEX)
- Input files match the repository template formatting (proteins/{protein}/{protein}_training.txt, proteins/{protein}/{cycle}.txt) before any metrics are computed

## Limitations

- The transcription factor binding site concept is not uniquely defined, depending on the intrinsically noisy, low-resolution experimental technique used to detect it, making a universal predictor impossible
- uPBM performance is sensitive to preprocessing: without motif trimming, undersampling, and 10x PWM-matching weighting, results are not comparable to the 0.69 ± 0.17 benchmark
- HT-SELEX cases with inconsistent enrichments between cycles can bias training/testing unless removed by the SVM quality check

## Evidence
