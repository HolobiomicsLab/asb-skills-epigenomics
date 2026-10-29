---
name: joint-cpg-dmr-calibration-modeling
description: Use when calibrating bisulfite amplicon methylation levels across all
  CpGs in a CGI or DMR and needing to choose among MethylCal's four candidate random-effects
  models (M1–M4) per assay via the Deviance Information Criterion.
when_to_use_negative:
- When CpG-by-CpG ('one-at-a-time') calibration or a single curve per average methylation
  level is desired — MethylCal's purpose is joint modeling of all CpGs within a CGI/DMR
- When using Warnecke et al. (1997) or Moskalev et al. (2011) calibration approaches
  is required — those are separate functions (WarneckeCalibrationPlot, MoskalevCalibrationPlot),
  not the M1–M4 random-effects DIC selection
- When data are case/control samples needing methylation correction only — use MoskalevCorrection
  or MethycalCorrection after calibration, not a new model selection run
tools:
- name: MethylCal
  role: R package providing Formatting() to restructure standard-control data and
    MethylCalCalibrationPlot() to fit the four random-effects specifications (M1–M4)
    and return mlik, DIC, and RSS
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference engine underlying MethylCal's latent Gaussian model fitting;
    must be installed manually before MethylCal
- name: R
  role: Environment in which MethylCal is written and executed; loaded via library(MethylCal)
    after devtools::install_github
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:5fd255692b25e0fb87a0118555fde4350052a3f88972696cbd8616d764496e3d
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# joint-cpg-dmr-calibration-modeling

## Summary

Fits MethylCal's four random-effects regression specifications (M1: AMP effects, M2: crossed CpG effects, M3: RW1 latent Gaussian field over AMP, M4: random CpG intercepts/slopes) jointly across all CpGs in a CGI/DMR using INLA, and selects the best calibration model per assay by lowest Deviance Information Criterion (DIC).

## When to use

Use when calibrating bisulfite amplicon methylation levels across all CpGs in a CGI or DMR and needing to choose among MethylCal's four candidate random-effects models (M1–M4) per assay via the Deviance Information Criterion.

## When NOT to use

- When CpG-by-CpG ('one-at-a-time') calibration or a single curve per average methylation level is desired — MethylCal's purpose is joint modeling of all CpGs within a CGI/DMR
- When using Warnecke et al. (1997) or Moskalev et al. (2011) calibration approaches is required — those are separate functions (WarneckeCalibrationPlot, MoskalevCalibrationPlot), not the M1–M4 random-effects DIC selection
- When data are case/control samples needing methylation correction only — use MoskalevCorrection or MethycalCorrection after calibration, not a new model selection run

## Inputs

- Standard-control methylation datasets with known methylation percentages (packaged BWS_data and Celiac_data)
- Vector of actual methylation/AMP percentages per dataset (BWS: 0, 25, 50, 75, 100; Celiac: 0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100)
- Number of control and case samples (e.g. n_Control = 15, n_Case = 18 for the BWS example)
- List of eight independent assays (two CGIs on the SDHC promoter and six imprinting DMRs) with their CpG positions

## Outputs

- Per-assay goodness-of-fit measures for each of the four fitted models: mlik, DIC, and RSS
- Assay-to-selected-model mapping table (e.g. dic_model_selection.csv) with eight rows and a lowest-DIC model assignment per assay
- Estimated calibration curves per assay with fitted MethylCal model

## How to apply

Restructure the standard-control data with Formatting() at the actual methylation percentages — 0/25/50/75/100% AMP for BWS_data and 0/12.5/25/37.5/50/62.5/87.5/100% AMP for Celiac_data. For each of the eight independent assays (two CpG islands on the SDHC gene promoter and six imprinting DMRs), fit the four candidate specifications of equations 2–5 jointly across all CpGs in the targeted CGI/DMR using MethylCalCalibrationPlot, which performs Bayesian inference with INLA. Record the goodness-of-fit measures returned per estimated model: mlik (marginal likelihood), DIC, and RSS. For each assay, select the model with the lowest DIC as the preferred calibration model. Validate the per-assay selection against Ochoa et al. (2019), where M4 was reported for KCNQ1OT1 and M3 for H19/IGF2, and write the eight-row assay-to-model mapping (e.g. dic_model_selection.csv) with a non-empty model assignment per row.

## Related tools

- **MethylCal** (R package providing Formatting() to restructure standard-control data and MethylCalCalibrationPlot() to fit the four random-effects specifications (M1–M4) and return mlik, DIC, and RSS) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference engine underlying MethylCal's latent Gaussian model fitting; must be installed manually before MethylCal)
- **R** (Environment in which MethylCal is written and executed; loaded via library(MethylCal) after devtools::install_github)

## Examples

```
library(MethylCal); data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)
```

## Evaluation signals

- Each assay's DIC-selected model reproduces the preferred model reported in Ochoa et al. (2019): model M4 for KCNQ1OT1 and model M3 for H19/IGF2
- The output mapping contains exactly eight assay rows (two CpG islands and six imprinting DMRs), each with a non-empty model assignment
- For every assay and model (M1–M4), all three goodness-of-fit measures (mlik, DIC, RSS) are recorded; the selected model has the lowest DIC among the four
- Data were formatted at the correct actual methylation percentages (five standard controls for BWS; eight AMPs for the celiac data) via Formatting() before fitting
- INLA inference completes without error for each MethylCalCalibrationPlot fit, confirming valid latent Gaussian model specifications

## Limitations

- No software version numbers are stated for MethylCal, R, or Moskalev's CPR; INLA must be downloaded and installed manually from its own repository before MethylCal works
- Outliers may distort fits; MethylCalCalibrationOutlierPlot refits models without detected outliers (posterior predictive interval l = Q1 − 1.5IQR, u = Q3 + 1.5IQR), and some CpGs may need removal by visual inspection (e.g. CpGs 21–23 removed from the PLAGL1 assay)
- Goodness-of-fit selection relies on DIC (with mlik and RSS as alternatives); supplementary figures referenced in the article are not included in the provided document text, limiting direct comparison

## Evidence

- [methods] MethylCalCalibrationPlot implements the method: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [methods] INLA-based Bayesian calibration: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [results] Five standard controls with known methylation percentages: "we derived the calibration curves using five standard controls with known methylation percentages"
- [intro] Eight independent assays design: "we designed eight independent assays in two CGIs located on SDHC gene promoter and six imprinted DMRs"
- [abstract] Joint analysis of all CpGs avoids one-at-a-time calibration: "which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR), avoiding 'one-at-a-time' CpG calibration"
- [readme] README installation requires manual INLA install: [evidence span withheld — anchor status: external_doc]
