---
name: methylation-calibration-curve-fitting
description: Use when bisulfite amplicon/pyrosequencing data include standard controls
  with known methylation percentages (e.g. AMP = 0/25/50/75/100%) and apparent post-PCR
  methylation levels per CpG need bias-corrected calibration curves.
when_to_use_negative:
- The data contain no standard controls with known methylation percentages, so no
  AMP-dependent calibration curve can be derived
- The goal is 'one-at-a-time' CpG calibration or calibration of average methylation
  across CpGs — MethylCal's advantage comes from joint modelling of all CpGs
- Methylation levels are already PCR-bias corrected; only downstream differential
  testing is needed
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: MethylCal
  role: Core R package providing Formatting, MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot,
    MethylCalCorrection and TableFormatting for joint CpG calibration and correction
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend (R-INLA) underlying MethylCal's mixed additive
    calibration models; installed manually from the INLA repository
  repo: http://www.r-inla.org/
- name: R
  role: Runtime environment; MethylCal is distributed as an R package installed via
    devtools::install_github
  repo: https://github.com/lb664/MethylCal
- name: Moskalev's CPR
  role: Baseline cubic polynomial regression (plus its Warnecke extension) fitted
    per assay for goodness-of-fit comparison against MethylCal
- name: WarneckeCalibrationPlot
  role: Plots the Warnecke et al. (1997) hyperbolic baseline calibration for each
    standard-control assay
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:4a5ea864b705c18b6c2f380203de113608d94806268057fad44d1f224d9402df
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# methylation-calibration-curve-fitting

## Summary

Fits Bayesian calibration curves (MethylCal, INLA-based) for PCR-bias correction of bisulfite amplicon sequencing methylation levels, jointly modelling all CpGs within a CGI or DMR, and benchmarks them against the Warnecke hyperbolic and Moskalev CPR baselines.

## When to use

Use when bisulfite amplicon/pyrosequencing data include standard controls with known methylation percentages (e.g. AMP = 0/25/50/75/100%) and apparent post-PCR methylation levels per CpG need bias-corrected calibration curves.

## When NOT to use

- The data contain no standard controls with known methylation percentages, so no AMP-dependent calibration curve can be derived
- The goal is 'one-at-a-time' CpG calibration or calibration of average methylation across CpGs — MethylCal's advantage comes from joint modelling of all CpGs
- Methylation levels are already PCR-bias corrected; only downstream differential testing is needed

## Inputs

- BWS_data: methylation levels of 15 healthy controls and 18 potential Beckwith-Wiedemann syndrome patients (KCNQ1OT1 and H19/IGF2 assays)
- Celiac_data: pyrosequenced methylation levels for eight NFkB-related genes in three target regions (celiac case/control dataset)
- AMP vector of standard-control methylation percentages (e.g. c(0, 25, 50, 75, 100) or c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100))
- Assay/Target name (e.g. 'KCNQ1OT1', 'H19/IGF2') and n_Control/n_Case counts

## Outputs

- Calibration curves per assay with goodness-of-fit measures (mlik, DIC, RSS) for each candidate model
- Calibration plots comparing MethylCal, Warnecke hyperbolic and Moskalev CPR (extension and cubic polynomial regression) fits
- Outlier detection flags with refitted (outlier-free) models
- Corrected methylation levels for cases and controls (clipped to [0%, 100%]), plus a printed list of hyper- and hypo-methylated cases
- Table in the original data format via TableFormatting()

## How to apply

Load the MethylCal R package (which depends on INLA and lattice/latticeExtra) and reshape raw methylation tables with Formatting(), supplying the AMP vector (c(0, 25, 50, 75, 100) for BWS_data; c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for Celiac_data) plus n_Control = 15 and n_Case = 18 where cases are present. For each targeted DMR/CGI assay, fit the Bayesian mixed additive model jointly across all CpGs with MethylCalCalibrationPlot(), recording mlik, DIC and RSS per candidate model and selecting the best model by DIC; optionally plot the Warnecke hyperbolic baseline (WarneckeCalibrationPlot) and Moskalev's two baselines (MoskalevCalibrationPlot) for comparison. Apply the posterior predictive outlier rule (l = Q1 − 1.5*IQR, u = Q3 + 1.5*IQR) via MethylCalCalibrationOutlierPlot() and refit without flagged points when outliers appear. Correct case/control levels with MethylCalCorrection() (or MoskalevCorrection()), classify patients against a ±3SD confidence interval on the healthy controls' distribution, clip corrected degrees to [0%, 100%], and convert back with TableFormatting().

## Related tools

- **MethylCal** (Core R package providing Formatting, MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot, MethylCalCorrection and TableFormatting for joint CpG calibration and correction) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend (R-INLA) underlying MethylCal's mixed additive calibration models; installed manually from the INLA repository) — http://www.r-inla.org/
- **R** (Runtime environment; MethylCal is distributed as an R package installed via devtools::install_github) — https://github.com/lb664/MethylCal
- **Moskalev's CPR** (Baseline cubic polynomial regression (plus its Warnecke extension) fitted per assay for goodness-of-fit comparison against MethylCal)
- **WarneckeCalibrationPlot** (Plots the Warnecke et al. (1997) hyperbolic baseline calibration for each standard-control assay) — https://github.com/lb664/MethylCal

## Examples

```
data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Per-assay mlik, DIC and RSS are reported for every candidate model, and the best model is the DIC-selected one
- All fitted methylation predictions lie within [0%, 100%] and every in-sample RSS value falls in [0, 1]
- Leave-one-out cross-validation (removing a CpG-AMP combination or a whole CpG) yields MSEP for MethylCal several orders of magnitude lower than Moskalev's CPR, with CV-index <= 1
- Exactly two outliers are expected in the H19/IGF2 assay under the l = Q1 − 1.5*IQR / u = Q3 + 1.5*IQR posterior predictive rule, and the refit takes several minutes
- Patient gain/loss-of-methylation calls match the known results: gain of methylation in B5A42 and B5B38 by both methods, with B5B37 additionally flagged by MethylCal in H19/IGF2

## Limitations

- No software version numbers are stated for MethylCal, R or Moskalev's CPR, so exact reproducibility of the reported fits is not guaranteed
- The H19/IGF2 outlier refit is computationally expensive (several minutes), and CpGs 21–23 had to be removed from the PLAGL1 assay by visual inspection
- R-INLA must be installed manually from its own repository before MethylCal can be installed

## Evidence

- [abstract] MethylCal analyses all CpGs jointly: "Here, we present a Bayesian calibration tool, MethylCal, which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR)"
- [methods] MethylCal is INLA-based: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [results] Outlier detection rule: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [methods] Outlier refit workflow: "The function `MethylCalCalibrationOutlierPlot` includes the detection of outliers, where the models are refitted without the detected outliers."
- [readme] Package visualisation capability: "The package permits the visualisation of the calibration curve derived from standard controls with known methylation percentages for a selected assay."
