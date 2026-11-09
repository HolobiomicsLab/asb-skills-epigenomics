---
name: bayesian-inla-model-fitting-in-r
description: Use when bisulfite amplicon sequencing standard-control data (known methylation
  percentages per AMP) must be modelled jointly across CpGs with Bayesian inference
  in R, e.g. deriving PCR-bias calibration curves or correcting case/control methylation
  levels.
when_to_use_negative:
- The analysis goal is one-at-a-time per-CpG calibration or calibration of the average
  methylation level across CpGs — MethylCal's value is joint modelling of all CpGs
  within a CGI/DMR
- 'No INLA installation is possible: R-INLA is a mandatory dependency that must be
  downloaded and installed manually and is not auto-installed like lattice/latticeExtra'
- The task only requires Moskalev's CPR or Warnecke's hyperbolic baseline curves,
  which are provided as separate package functions and do not involve INLA
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: MethylCal
  role: R package exposing the INLA-based Bayesian calibration functions (MethylCalCalibrationPlot,
    MethylCalCalibrationOutlierPlot, MethylCalCorrection) applied to all CpGs of a
    CGI/DMR
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Backend performing approximate Bayesian inference for the latent Gaussian
    random-effects models fitted to the standard-control methylation data
  repo: http://www.r-inla.org/
- name: R
  role: Runtime environment; the skill is invoked by loading the package with library(MethylCal)
- name: devtools
  role: Used to install MethylCal from GitHub via devtools::install_github('lb664/MethylCal',
    build_opts = c('--no-resave-data', '--no-manual'))
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:87a363ae5c97e4d450865466c0b4574f696d947e19bdfbc3c83c8100bc7dc776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - MethylCalCalibrationOutlierPlot
  - MethylCalCalibrationPlot
  - Moskalev's CPR
  - MoskalevCalibrationPlot
  - R
  - WarneckeCalibrationPlot
  - devtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# bayesian-inla-model-fitting-in-r

## Summary

Fit Bayesian latent Gaussian models in R via the INLA engine, as implemented in the MethylCal package, to jointly calibrate methylation levels across all CpGs in a CpG island or DMR and select among candidate random-effects specifications using DIC.

## When to use

Use when bisulfite amplicon sequencing standard-control data (known methylation percentages per AMP) must be modelled jointly across CpGs with Bayesian inference in R, e.g. deriving PCR-bias calibration curves or correcting case/control methylation levels.

## When NOT to use

- The analysis goal is one-at-a-time per-CpG calibration or calibration of the average methylation level across CpGs — MethylCal's value is joint modelling of all CpGs within a CGI/DMR
- No INLA installation is possible: R-INLA is a mandatory dependency that must be downloaded and installed manually and is not auto-installed like lattice/latticeExtra
- The task only requires Moskalev's CPR or Warnecke's hyperbolic baseline curves, which are provided as separate package functions and do not involve INLA

## Inputs

- Packaged standard-control datasets BWS_data and Celiac_data (bisulfite amplicon sequencing methylation levels)
- AMP vectors of actual methylation percentages (e.g. c(0, 25, 50, 75, 100) or c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100))
- n_Control and n_Case counts (e.g. n_Control = 15, n_Case = 18 for BWS; n_Control = 13, n_Case = 34 for celiac)
- Target assay name (e.g. 'KCNQ1OT1', 'H19/IGF2') selecting a CGI or DMR

## Outputs

- Bayesian calibration curves per assay with 95% prediction credible intervals (lattice/latticeExtra plots)
- Per-model goodness-of-fit measures: mlik (marginal likelihood), DIC and RSS
- DIC-selected model per assay among candidate specifications M1–M4
- Corrected methylation levels for case and control samples (via MethylCalCorrection, convertible back with TableFormatting)

## How to apply

Install R-INLA manually from https://inla.r-inla-download.org/R/stable, then install MethylCal with devtools::install_github('lb664/MethylCal', build_opts = c('--no-resave-data', '--no-manual')). Reshape the uploaded data with Formatting(), supplying AMP = c(0, 25, 50, 75, 100) for BWS_data or AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for Celiac_data, plus n_Control/n_Case counts. Fit the four candidate random-effects specifications (AMP effects, crossed CpG effects, RW1 latent Gaussian field over AMP, random CpG intercepts/slopes) jointly across all CpGs of the targeted CGI/DMR with MethylCalCalibrationPlot, recording mlik, DIC and RSS for each model and selecting the model with the lowest DIC. When posterior predictive outlier detection flags points (interval l = Q1 − 1.5IQR, u = Q3 + 1.5IQR), refit with MethylCalCalibrationOutlierPlot excluding those points. Rationale: joint modelling of all CpGs avoids 'one-at-a-time' CpG calibration and yields narrower 95% prediction credible intervals than per-CpG competitors like Moskalev's CPR or Warnecke's hyperbolic model.

## Related tools

- **MethylCal** (R package exposing the INLA-based Bayesian calibration functions (MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot, MethylCalCorrection) applied to all CpGs of a CGI/DMR) — https://github.com/lb664/MethylCal
- **INLA** (Backend performing approximate Bayesian inference for the latent Gaussian random-effects models fitted to the standard-control methylation data) — http://www.r-inla.org/
- **R** (Runtime environment; the skill is invoked by loading the package with library(MethylCal))
- **devtools** (Used to install MethylCal from GitHub via devtools::install_github('lb664/MethylCal', build_opts = c('--no-resave-data', '--no-manual'))) — https://github.com/r-lib/devtools

## Examples

```
library(MethylCal); data(BWS_data); data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18); MethylCalCalibrationPlot(data, Target = "KCNQ1OT1")
```

## Evaluation signals

- Per-model goodness-of-fit metrics (mlik, DIC, RSS) are reported for each of the four candidate specifications, and the selected model is the one with the lowest DIC
- All fitted predictions lie within [0%, 100%] apparent methylation, and in-sample RSS values fall in [0, 1]
- The DIC-selected model per assay matches the model reported in Ochoa et al. (2019) (e.g. model M4 for KCNQ1OT1 and model M3 for H19/IGF2)
- Outlier refits flag points outside the posterior predictive interval l = Q1 − 1.5IQR, u = Q3 + 1.5IQR, and refitting without them (e.g. two outliers in H19/IGF2) reproduces the expected improved fit
- Corrected outputs from TableFormatting() contain one corrected methylation degree per individual and CpG/region, all within the 0–100% range

## Limitations

- No software version numbers are stated for R, INLA or the MethylCal package, so exact reproducibility of DIC/mlik values is not guaranteed
- Refitting after outlier removal can be slow: the H19/IGF2 assay has two outliers and the refit takes several minutes
- Some assays require manual pre-cleaning: CpGs 21–23 were removed from the PLAGL1 assay by visual inspection before fitting
- Supplementary figures supporting model behaviour (e.g. S.3C-E, S.3G-I) are not included in the provided document text

## Evidence

- [readme] This package allows the calibration of methylation levels using MethylCal, a Bayesian calibration tool based on INLA: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [readme] INLA must be installed manually with a dedicated repos URL before MethylCal can be used: [evidence span withheld — anchor status: external_doc]
- [abstract] MethylCal analyses all CpGs jointly within a CGI or DMR, avoiding per-CpG calibration: "which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR), avoiding 'one-at-a-time' CpG calibration"
- [methods] MethylCal is implemented in MethylCalCalibrationPlot and outlier detection refits models without detected outliers: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [results] Outlier detection uses a posterior predictive interval based on the 1.5*IQR rule: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [results] MethylCal interpolates standard controls more precisely than Moskalev's CPR with narrower credible intervals: [evidence span withheld — anchor status: not_found]
