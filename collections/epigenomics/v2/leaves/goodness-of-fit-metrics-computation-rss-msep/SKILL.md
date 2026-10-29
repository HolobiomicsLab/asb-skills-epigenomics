---
name: goodness-of-fit-metrics-computation-rss-msep
description: Use when benchmarking calibration models for methylation standard-control
  data (known AMP percentages such as 0/25/50/75/100%) and you need in-sample fit
  (RSS) plus leave-one-out predictive measures (MSEP, CV-index).
when_to_use_negative:
- When only correcting methylation levels of case/control samples (use MoskalevCorrection
  / MethycalCorrection instead of computing calibration fit metrics)
- When the data are not standard controls with known methylation percentages — RSS/MSEP
  from calibration curves are meaningless for uncontrolled case samples
- When assessing outlier CpGs (use MethylCalCalibrationOutlierPlot with the Q1−1.5IQR
  / Q3+1.5IQR interval rather than fit-metric comparison)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: MethylCal
  role: INLA-based Bayesian joint calibration of all CpGs within a CGI or DMR; its
    goodness-of-fit output supplies RSS, and refits supply CV MSEP/CV-index
  repo: https://github.com/lb664/MethylCal
- name: Moskalev's CPR
  role: Cubic polynomial regression baseline fitted with MoskalevCalibrationPlot;
    RSS/MSEP/CV-index extracted identically for comparison
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Backend for the Bayesian model fitting underlying MethylCal's calibration
    curves
  repo: http://www.r-inla.org/
- name: R
  role: Implementation environment; MethylCal is an R package installed via devtools::install_github
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:e5366ec13b6771e4aff2ac4e946b97ba947324e3e71d805c10fea3008d53f819
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# goodness-of-fit-metrics-computation-rss-msep

## Summary

Computes and compares in-sample residual sum of squares (RSS) and out-of-sample predictive measures (MSEP, CV-index) for methylation calibration curves, as used to benchmark MethylCal against Moskalev's CPR across eight bisulfite-amplicon assays.

## When to use

Use when benchmarking calibration models for methylation standard-control data (known AMP percentages such as 0/25/50/75/100%) and you need in-sample fit (RSS) plus leave-one-out predictive measures (MSEP, CV-index).

## When NOT to use

- When only correcting methylation levels of case/control samples (use MoskalevCorrection / MethycalCorrection instead of computing calibration fit metrics)
- When the data are not standard controls with known methylation percentages — RSS/MSEP from calibration curves are meaningless for uncontrolled case samples
- When assessing outlier CpGs (use MethylCalCalibrationOutlierPlot with the Q1−1.5IQR / Q3+1.5IQR interval rather than fit-metric comparison)

## Inputs

- BWS_data (packaged methylation standard-control data, 5 AMPs: 0, 25, 50, 75, 100; 15 controls, 18 cases)
- Celiac_data (packaged methylation standard-control data, 8 AMPs: 0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100)
- Per-assay CpG-level apparent methylation levels for the two SDHC CGIs and six imprinting DMRs

## Outputs

- Per-assay in-sample RSS values for MethylCal and Moskalev's CPR
- Leave-one-out MSEP and CV-index per method and assay
- A single CSV table with one row per assay and one column block per method

## How to apply

Format each packaged dataset (BWS_data, Celiac_data) with Formatting(), supplying AMP vectors c(0,25,50,75,100) for BWS and c(0,12.5,25,37.5,50,62.5,87.5,100) for celiac. For each of the eight assays (two SDHC-promoter CGIs and six imprinting DMRs), fit MethylCal via MethylCalCalibrationPlot (INLA-based joint Bayesian calibration of all CpGs) and Moskalev's cubic polynomial regression via MoskalevCalibrationPlot, extracting in-sample RSS from each returned goodness-of-fit object. Then perform leave-one-out cross-validation in three scenarios — removing a single CpG-AMP combination, each CpG one-at-a-time, and each AMP separately — refitting both methods per fold and computing out-of-sample MSEP and the CV-index. Assemble per-assay, per-method RSS, MSEP and CV-index into a single CSV table (one row per assay, one column block per method) for direct comparison against the paper's predictive-measure table.

## Related tools

- **MethylCal** (INLA-based Bayesian joint calibration of all CpGs within a CGI or DMR; its goodness-of-fit output supplies RSS, and refits supply CV MSEP/CV-index) — https://github.com/lb664/MethylCal
- **Moskalev's CPR** (Cubic polynomial regression baseline fitted with MoskalevCalibrationPlot; RSS/MSEP/CV-index extracted identically for comparison) — https://github.com/lb664/MethylCal
- **INLA** (Backend for the Bayesian model fitting underlying MethylCal's calibration curves) — http://www.r-inla.org/
- **R** (Implementation environment; MethylCal is an R package installed via devtools::install_github) — https://github.com/lb664/MethylCal

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)
```

## Evaluation signals

- Every tabulated RSS value falls in [0, 1]
- Every computed CV-index is <= 1
- Per-assay, per-method RSS/MSEP/CV-index values reproduce the paper's reported predictive-measure table
- MethylCal's best DIC-selected model yields a leave-one-out MSEP several orders of magnitude lower than Moskalev's CPR when a CpG-AMP combination or a CpG is removed
- The cross-validation covers all three removal scenarios (single CpG-AMP point, each CpG one-at-a-time, each AMP separately)

## Limitations

- No software version numbers are stated for MethylCal, Moskalev's CPR, R, or R-INLA, so exact reproducibility of numeric metrics is not guaranteed
- CpGs 21–23 of the PLAGL1 assay must be removed by visual inspection before fitting; including them changes fit metrics
- Leave-one-out advantage of MethylCal is scenario-dependent: it is strongest when a CpG-AMP combination or whole CpG is held out

## Evidence

- [results] Cross-validation scenarios: "the cross-validation is performed by removing a data point that corresponds to a specific CpG-AMP combination. In the second scenario, each CpG is excluded one-at-a-time"
- [results] Goodness-of-fit comparison against CPR: "we checked the goodness of fit of the calibration curves obtained by MethylCal and compared the results with those obtained by Moskalev’s CPR"
- [methods] MethylCal calibration function: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [readme] Package installation and dependency: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [readme] Data formatting entry point: "data(BWS_data)
AMP = c(0, 25, 50, 75, 100)
data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)"
