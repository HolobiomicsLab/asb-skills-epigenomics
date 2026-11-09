---
name: leave-one-out-cross-validation-of-calibration-models
description: Use when you have calibration data on standard controls of known methylation
  percentage (e.g. 0–100% AMP dilutions) measured across multiple CpGs and need to
  compare predictive performance of competing calibration models per assay.
when_to_use_negative:
- Your goal is only in-sample goodness-of-fit visualization of the calibration curve
  (ExploratoryPlot or the CalibrationPlot functions suffice) — cross-validation is
  for out-of-sample predictive comparison
- Your data are case/control methylation levels to be corrected (use MethylCalCorrection
  / MoskalevCorrection rather than the calibration-model CV protocol)
- You have no standard controls of known methylation percentage, so calibration curves
  cannot be fitted or held out
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: MethylCal
  role: Provides MethylCalCalibrationPlot for INLA-based joint Bayesian calibration
    of all CpGs and the data objects (BWS_data, Celiac_data) used in the cross-validation
    comparison
  repo: https://github.com/lb664/MethylCal
- name: Moskalev's CPR
  role: Cubic polynomial regression baseline whose RSS, MSEP and CV-index are compared
    against MethylCal's DIC-selected model
- name: INLA
  role: Bayesian inference backend used to fit MethylCal's calibration models during
    each cross-validation refit
- name: R
  role: Language in which the MethylCal package and the whole CV workflow are implemented
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:40738478ae99a1e45c9f865722318e8f573a0b34521d2f5c0519969db4341586
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# leave-one-out-cross-validation-of-calibration-models

## Summary

Replicates the leave-one-out cross-validation protocol used in Ochoa et al. (2019) to compare MethylCal's INLA-based Bayesian calibration against Moskalev's cubic polynomial regression (CPR) on bisulfite amplicon-sequencing standard controls, using RSS, MSEP and a CV-index as the comparison metrics.

## When to use

Use when you have calibration data on standard controls of known methylation percentage (e.g. 0–100% AMP dilutions) measured across multiple CpGs and need to compare predictive performance of competing calibration models per assay.

## When NOT to use

- Your goal is only in-sample goodness-of-fit visualization of the calibration curve (ExploratoryPlot or the CalibrationPlot functions suffice) — cross-validation is for out-of-sample predictive comparison
- Your data are case/control methylation levels to be corrected (use MethylCalCorrection / MoskalevCorrection rather than the calibration-model CV protocol)
- You have no standard controls of known methylation percentage, so calibration curves cannot be fitted or held out

## Inputs

- Standard-control methylation data of known methylation percentages, formatted with Formatting() — BWS_data with AMP = c(0, 25, 50, 75, 100) or Celiac_data with AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100)
- Eight independent assays: two CpG islands on the SDHC gene promoter and six imprinting DMRs
- R with the MethylCal package (github.com/lb664/MethylCal) and its R-INLA dependency

## Outputs

- Per-assay, per-method in-sample Residual Sum of Squares (RSS) from goodness-of-fit measures
- Per-assay, per-method out-of-sample mean squared prediction error (MSEP) from leave-one-out cross-validation
- Per-assay, per-method CV-index
- A single CSV table with one row per assay and one column block per method

## How to apply

Fit both candidate calibration models to the standard-control data — MethylCal's joint INLA-based Bayesian calibration over all CpGs in a CpG island or imprinting DMR (via MethylCalCalibrationPlot), and Moskalev's CPR per CpG (via MoskalevCalibrationPlot) — extracting each model's in-sample Residual Sum of Squares. Then perform leave-one-out cross-validation in the paper's three scenarios: remove one CpG-AMP data point at a time, remove each CpG one-at-a-time, and remove each AMP separately; at each removal refit both models and compute out-of-sample MSEP and the CV-index on the held-out observation. Assemble per-assay, per-method RSS, MSEP and CV-index into a single comparison table (one row per assay, one column block per method) and use it to judge which calibration approach generalizes better. Cross-validation is the key decision point because in-sample RSS alone can mask differences in out-of-sample prediction of methylation levels not considered in the controlled experiment.

## Related tools

- **MethylCal** (Provides MethylCalCalibrationPlot for INLA-based joint Bayesian calibration of all CpGs and the data objects (BWS_data, Celiac_data) used in the cross-validation comparison) — https://github.com/lb664/MethylCal
- **Moskalev's CPR** (Cubic polynomial regression baseline whose RSS, MSEP and CV-index are compared against MethylCal's DIC-selected model)
- **INLA** (Bayesian inference backend used to fit MethylCal's calibration models during each cross-validation refit)
- **R** (Language in which the MethylCal package and the whole CV workflow are implemented) — https://github.com/lb664/MethylCal

## Examples

```
library(MethylCal); data(BWS_data); data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18)
```

## Evaluation signals

- Every tabulated RSS value falls in [0, 1]
- Every computed CV-index is <= 1
- MethylCal's best DIC-selected model attains a leave-one-out MSEP lower than Moskalev's CPR by several orders of magnitude when cross-validation removes a CpG-AMP combination or a CpG
- The assembled per-assay, per-method table of RSS, MSEP and CV-index matches the paper's reported predictive-measure table
- Cross-validation is executed in all three removal scenarios (CpG-AMP combination, CpG one-at-a-time, AMP separately), not just a single holdout scheme

## Limitations

- CpGs 21–23 had to be removed from the PLAGL1 assay by visual inspection before calibration, so some assays require prior outlier/filtering decisions (e.g. l = Q1 − 1.5IQR, u = Q3 + 1.5IQR) that affect the CV results
- No software version numbers for MethylCal, Moskalev's CPR or R are stated in the paper, so exact numerical reproduction may depend on package versions
- The finding of orders-of-magnitude lower MSEP holds for the paper's standard-control data on the eight assays; it is not demonstrated for arbitrary methylation datasets

## Evidence

- [results] CV removes a CpG-AMP combination: "the cross-validation is performed by removing a data point that corresponds to a specific CpG-AMP combination."
- [results] Goodness-of-fit comparison against CPR: "we checked the goodness of fit of the calibration curves obtained by MethylCal and compared the results with those obtained by Moskalev’s CPR"
- [readme] MethylCal is a Bayesian calibration tool based on INLA: "This package allows the calibration of methylation levels using
**MethylCal**, a Bayesian calibration tool based on INLA"
- [results] Standard controls define the calibration curves: "we derived the calibration curves using five standard controls with known methylation percentages"
