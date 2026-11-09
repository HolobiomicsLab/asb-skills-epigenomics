---
name: deviance-information-criterion-model-selection
description: Decision support for choosing among MethylCal's four model specifications
  (equations 2–5) per assay when calibrating apparent methylation levels from standard
  controls at known percentages (e.g. 0/25/50/75/100% or 0–100% at 12.5% steps).
when_to_use_negative:
- The goal is outlier detection or model refitting without detected outliers — use
  MethylCalCalibrationOutlierPlot with the l = Q1 − 1.5IQR, u = Q3 + 1.5IQR posterior
  predictive interval instead
- The aim is correction of methylation levels in case/control samples (use MoskalevCorrection
  or MethycalCorrection), not selection of the calibration model itself
- Data are average methylation levels or one-at-a-time CpG calibrations; MethylCal's
  premise is joint analysis of all CpGs within a CGI or DMR
tools:
- name: MethylCal
  role: R package providing Formatting() to restructure standard-control data and
    MethylCalCalibrationPlot to fit the four random-effects models and return mlik,
    DIC and RSS
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend used by MethylCalCalibrationPlot to estimate the
    candidate models from which DIC is computed
- name: R
  role: Runtime environment in which the MethylCal package is loaded and the per-assay
    model comparison is executed
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:c1e72e4e39e406f18185cef16259f442b9275875a4600da09e5e05b3fc7c325d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# deviance-information-criterion-model-selection

## Summary

Selects the best-fitting MethylCal random-effects calibration model (M1–M4: AMP effects, crossed CpG effects, RW1 latent Gaussian field, random CpG intercepts/slopes) for each bisulfite amplicon sequencing assay by comparing DIC values returned by INLA-based inference on standard-control methylation data.

## When to use

Decision support for choosing among MethylCal's four model specifications (equations 2–5) per assay when calibrating apparent methylation levels from standard controls at known percentages (e.g. 0/25/50/75/100% or 0–100% at 12.5% steps).

## When NOT to use

- The goal is outlier detection or model refitting without detected outliers — use MethylCalCalibrationOutlierPlot with the l = Q1 − 1.5IQR, u = Q3 + 1.5IQR posterior predictive interval instead
- The aim is correction of methylation levels in case/control samples (use MoskalevCorrection or MethycalCorrection), not selection of the calibration model itself
- Data are average methylation levels or one-at-a-time CpG calibrations; MethylCal's premise is joint analysis of all CpGs within a CGI or DMR

## Inputs

- BWS_data standard-control methylation dataset (15 controls, 18 potential BWS cases, KCNQ1OT1 and H19/IGF2 assays) at AMPs 0/25/50/75/100%
- Celiac_data standard-control methylation dataset (eight NFkB-related genes) at AMPs 0/12.5/25/37.5/50/62.5/87.5/100%
- R-formatted data produced by Formatting() with the AMP vector matching the controlled experiment

## Outputs

- Per-assay mlik, DIC and RSS goodness-of-fit values for each of the four MethylCal models (M1–M4)
- A mapping of each of the eight assays to its DIC-selected model (e.g. dic_model_selection.csv with eight assay rows)

## How to apply

Load the packaged BWS_data or Celiac_data standard-control datasets and restructure them with MethylCal's Formatting() at the actual methylation percentages used in the controlled experiment (BWS: 0/25/50/75/100%; Celiac: 0/12.5/25/37.5/50/62.5/87.5/100%). For each of the eight independent assays (two CGIs on the SDHC promoter and six imprinting DMRs), fit all four candidate random-effects specifications jointly across all CpGs of the targeted CGI/DMR using MethylCalCalibrationPlot, which performs Bayesian inference with R-INLA. Record the goodness-of-fit measures returned for each fit — mlik (marginal likelihood), DIC (Deviance Information Criterion) and RSS (Residual Sum of Squares). For each assay, select the model with the lowest DIC as the preferred calibration model, and cross-check the selection against the models reported in Ochoa et al. (2019), where model M4 was selected for KCNQ1OT1 and model M3 for H19/IGF2.

## Related tools

- **MethylCal** (R package providing Formatting() to restructure standard-control data and MethylCalCalibrationPlot to fit the four random-effects models and return mlik, DIC and RSS) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend used by MethylCalCalibrationPlot to estimate the candidate models from which DIC is computed)
- **R** (Runtime environment in which the MethylCal package is loaded and the per-assay model comparison is executed) — https://github.com/lb664/MethylCal

## Examples

```
library(MethylCal); data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)
```

## Evaluation signals

- The DIC-selected model for KCNQ1OT1 is M4 and for H19/IGF2 is M3, matching the preferred models reported in Ochoa et al. (2019)
- The output mapping contains exactly eight assay rows (two CGIs plus six imprinting DMRs), each with a non-empty model assignment among M1–M4
- Every fit returns all three goodness-of-fit measures (mlik, DIC, RSS); the selected model must have the lowest DIC among the four for that assay
- Input data were formatted with Formatting() using the exact AMP vectors of the controlled experiment (BWS: 0/25/50/75/100%; Celiac: 0/12.5/25/37.5/50/62.5/87.5/100%)

## Limitations

- No software version numbers are stated for MethylCal, R or INLA, so exact numerical DIC reproducibility may vary across versions
- Model fits rely on R-INLA, which must be installed manually (install.packages('INLA', repos = 'https://inla.r-inla-download.org/R/stable')) and may take a couple of minutes
- CpG exclusions (e.g. CpGs 21–23 removed from PLAGL1 by visual inspection) can change which model is selected, so the filtering history of each assay must be fixed before comparing DIC

## Evidence

- [methods] MethylCalCalibrationPlot fits candidate models: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [methods] MethylCal is an R package based on INLA: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [results] Goodness of fit of calibration curves is checked: "we checked the goodness of fit of the calibration curves obtained by MethylCal and compared the results with those obtained by Moskalev’s CPR"
- [discussion] MethylCal availability and implementation language: "Written in R and available as an R package on https://github.com/lb664/MethylCal."
- [readme] README documents installation and standard-control calibration context: "The package permits the visualisation of the calibration curve derived from standard controls with known methylation percentages for a selected assay."
