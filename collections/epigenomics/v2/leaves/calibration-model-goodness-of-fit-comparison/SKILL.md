---
name: calibration-model-goodness-of-fit-comparison
description: Use when pyrosequenced or MiSeq methylation data from standard controls
  with known methylation percentages (e.g. AMP 0/25/50/75/100) are available and competing
  PCR-bias calibration models must be fitted and compared per assay.
when_to_use_negative:
- When the goal is correcting observed methylation levels of case/control samples
  with an already-fitted curve — use MoskalevCorrection or MethycalCorrection instead
  of refitting calibration curves.
- When data have not yet been reshaped or explored — run Formatting()/TableFormatting()
  and ExploratoryPlot first, since MethylCal calibration functions require the formatted
  analysis format.
- When only a single CpG is of interest and joint-CpG modelling is unnecessary — the
  skill's advantage comes from jointly modelling all CpGs within a CGI or DMR.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MethylCal
  role: Bayesian mixed additive calibration model fit jointly across all CpGs of an
    assay (MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot)
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Backend for Bayesian inference (mlik/DIC computation) in MethylCal's calibration
    models
- name: Moskalev's CPR
  role: Cubic polynomial regression and Warnecke-extension baselines fitted per assay
    for comparison (MoskalevCalibrationPlot)
- name: WarneckeCalibrationPlot
  role: Fits and plots the Warnecke et al. (1997) hyperbolic calibration baseline
    for a selected Target assay
  repo: https://github.com/lb664/MethylCal
- name: Formatting
  role: Reshapes BWS_data/Celiac_data into the calibration-analysis format with the
    AMP vector of standard-control percentages
  repo: https://github.com/lb664/MethylCal
- name: R
  role: Language in which the MethylCal package and all calibration/comparison functions
    are implemented
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:33d280640ad9db88672ba13f79431a3a9997a75fb8964fb461a604bf46d8a30e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - Formatting
  - INLA
  - MethylCal
  - MethylCalCalibrationOutlierPlot
  - MethylCalCalibrationPlot
  - Moskalev's CPR
  - MoskalevCalibrationPlot
  - R
  - WarneckeCalibrationPlot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# calibration-model-goodness-of-fit-comparison

## Summary

Fits and compares candidate calibration curves (Warnecke hyperbolic, Moskalev cubic polynomial regression, and MethylCal's Bayesian mixed additive model) for bisulfite amplicon-sequencing standard controls, selecting the best model per assay by DIC and validating goodness of fit via mlik, DIC, and RSS.

## When to use

Use when pyrosequenced or MiSeq methylation data from standard controls with known methylation percentages (e.g. AMP 0/25/50/75/100) are available and competing PCR-bias calibration models must be fitted and compared per assay.

## When NOT to use

- When the goal is correcting observed methylation levels of case/control samples with an already-fitted curve — use MoskalevCorrection or MethycalCorrection instead of refitting calibration curves.
- When data have not yet been reshaped or explored — run Formatting()/TableFormatting() and ExploratoryPlot first, since MethylCal calibration functions require the formatted analysis format.
- When only a single CpG is of interest and joint-CpG modelling is unnecessary — the skill's advantage comes from jointly modelling all CpGs within a CGI or DMR.

## Inputs

- Packaged methylation datasets (BWS_data, Celiac_data) or user bisulfite amplicon-sequencing methylation tables
- Vector of standard-control methylation percentages (AMP), e.g. c(0, 25, 50, 75, 100)
- Assay identifiers (Target) for each DMR/CGI, e.g. KCNQ1OT1, H19/IGF2, PLAGL1, SDHC promoter CGIs
- Counts of controls and cases (e.g. n_Control = 15, n_Case = 18) for the BWS example

## Outputs

- Calibration plots per assay for Warnecke hyperbolic, Moskalev CPR, and MethylCal models
- Per-model goodness-of-fit metrics (mlik, DIC, RSS) with the best model selected by DIC
- 95% prediction credible intervals for MethylCal's fitted curves
- Outlier-flagged points under l = Q1 - 1.5*IQR, u = Q3 + 1.5*IQR and refitted plots with/without outliers

## How to apply

Reshape the methylation data with Formatting() (e.g. AMP = c(0, 25, 50, 75, 100) for BWS_data, AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for Celiac_data), then fit the Warnecke et al. (1997) hyperbolic baseline and the two Moskalev et al. (2011) baselines per targeted DMR/CGI assay with WarneckeCalibrationPlot and MoskalevCalibrationPlot. Fit MethylCal's Bayesian mixed additive model jointly across all CpGs of each assay with MethylCalCalibrationPlot, recording mlik, DIC and RSS for each candidate model and selecting the best by DIC. Rationale: joint modelling of all CpGs within a CGI or DMR yields much narrower 95% prediction credible intervals and captures AMP-dependent variability better than one-at-a-time CpG baselines. Where the posterior predictive outlier rule (l = Q1 - 1.5*IQR, u = Q3 + 1.5*IQR) flags points, refit with MethylCalCalibrationOutlierPlot and present plots with and without outliers. Confirm all fitted predictions lie within [0%, 100%] before using the curves to correct case/control samples.

## Related tools

- **MethylCal** (Bayesian mixed additive calibration model fit jointly across all CpGs of an assay (MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot)) — https://github.com/lb664/MethylCal
- **INLA** (Backend for Bayesian inference (mlik/DIC computation) in MethylCal's calibration models)
- **Moskalev's CPR** (Cubic polynomial regression and Warnecke-extension baselines fitted per assay for comparison (MoskalevCalibrationPlot))
- **WarneckeCalibrationPlot** (Fits and plots the Warnecke et al. (1997) hyperbolic calibration baseline for a selected Target assay) — https://github.com/lb664/MethylCal
- **Formatting** (Reshapes BWS_data/Celiac_data into the calibration-analysis format with the AMP vector of standard-control percentages) — https://github.com/lb664/MethylCal
- **R** (Language in which the MethylCal package and all calibration/comparison functions are implemented) — https://github.com/lb664/MethylCal

## Examples

```
library(MethylCal); data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18); MethylCalCalibrationPlot(data, Target = "KCNQ1OT1")
```

## Evaluation signals

- Per-model mlik, DIC and RSS are reported for every candidate model of each of the eight assays, with the best model selected by DIC.
- The DIC-selected MethylCal curve captures the AMP-dependent variability of apparent methylation after PCR better than both the Warnecke hyperbolic and Moskalev CPR baselines.
- MethylCal's 95% prediction credible intervals are much narrower than Moskalev's CPR, and all fitted predictions lie within [0%, 100%].
- Predicted methylation levels closely interpolate the standard-control data points across the AMP grid for each assay.
- Outlier refits (MethylCalCalibrationOutlierPlot) run with the posterior predictive rule l = Q1 - 1.5*IQR, u = Q3 + 1.5*IQR and plots are presented with and without detected outliers.

## Limitations

- No software version numbers are stated for MethylCal, R, or Moskalev's CPR, and no accession/availability information is given for the BWS and celiac case datasets.
- Refitting with MethylCalCalibrationOutlierPlot can be slow — the H19/IGF2 refit with two detected outliers takes several minutes.
- Manual curation may be needed before fitting: CpGs 21-23 were removed from the PLAGL1 assay by visual inspection of the data.
- R-INLA is not installed automatically as a dependency and must be installed manually from the INLA repository before MethylCal can be used.

## Evidence

- [results] Goodness-of-fit comparison with Moskalev baseline: "we checked the goodness of fit of the calibration curves obtained by MethylCal and compared the results with those obtained by Moskalev’s CPR"
- [results] Outlier rule and refitting: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [methods] MethylCal implementation function: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [readme] Package purpose per README: "The package permits the visualisation of the calibration curve derived from standard controls with known methylation percentages for a selected assay."
- [abstract] Joint CpG modelling advantage: "We tested the proposed method on eight independent assays (two CpG islands and six imprinting DMRs) and demonstrated its benefits, including the ability to detect outliers"
