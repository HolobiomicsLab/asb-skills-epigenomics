---
name: posterior-predictive-outlier-detection
description: Use when calibrating pyrosequenced/amplicon methylation levels with MethylCal
  and suspect aberrant standard-control observations at some CpG-AMP combinations
  that would bias the fitted calibration curve.
when_to_use_negative:
- The data are not standard controls with known actual methylation percentages (AMP)
  — the posterior predictive interval must be estimated from the calibration controls
- You are applying Warnecke or Moskalev calibration instead of the MethylCal Bayesian
  method, as outlier detection and refitting is a feature of MethylCalCalibrationOutlierPlot
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: MethylCal
  role: R package providing MethylCalCalibrationOutlierPlot, which detects outliers
    via the posterior predictive interval and refits the Bayesian calibration models
    without them
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend underlying the MethylCal calibration models that
    generate the posterior predictive distribution
- name: R
  role: Runtime environment in which the MethylCal package and its outlier-detection
    function are executed
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:a9fe80225e1b2e90fe393d32d46dc79a04aeacc9c1712fafa1f79b7597b9f659
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# posterior-predictive-outlier-detection

## Summary

Detect aberrant standard-control data points during MethylCal Bayesian calibration using the posterior predictive interval defined by l = Q1 − 1.5·IQR and u = Q3 + 1.5·IQR, then refit the calibration models without the detected outliers so PCR-bias correction is not distorted.

## When to use

Use when calibrating pyrosequenced/amplicon methylation levels with MethylCal and suspect aberrant standard-control observations at some CpG-AMP combinations that would bias the fitted calibration curve.

## When NOT to use

- The data are not standard controls with known actual methylation percentages (AMP) — the posterior predictive interval must be estimated from the calibration controls
- You are applying Warnecke or Moskalev calibration instead of the MethylCal Bayesian method, as outlier detection and refitting is a feature of MethylCalCalibrationOutlierPlot

## Inputs

- Formatted MethylCal data object produced by Formatting() from pyrosequenced methylation levels of standard control samples with known AMP percentages (e.g. AMP = c(0, 12.5, 25, 37.5, 50, 62.5, 87.5, 100) for Celiac_data)
- Target assay/region identifier (e.g. one of the celiac susceptibility target regions NFKBIA, RELA, TNFAIP3)

## Outputs

- Calibration plot highlighting detected outliers relative to the posterior predictive interval [l, u]
- Refitted MethylCal calibration models (without detected outliers) usable by MethylCalCorrection

## How to apply

Fit the MethylCal Bayesian calibration (INLA-based) on the standard controls with known actual methylation percentages (AMP) using MethylCalCalibrationOutlierPlot for the chosen assay (e.g. NFKBIA, RELA, TNFAIP3 in the celiac disease data). The function computes the posterior predictive interval with l = Q1 − 1.5·IQR and u = Q3 + 1.5·IQR, flags observed methylation levels falling outside [l, u] as outliers, and automatically refits the models excluding those data points. Inspect the returned outlier plot to confirm which CpG-AMP combinations were removed before proceeding to MethylCalCorrection. Rationale: MethylCal analyses jointly all CpGs within a CGI or DMR, so a few extreme control points can distort the shared calibration curve; refitting without outliers improves the accuracy of subsequent methylation-level correction.

## Related tools

- **MethylCal** (R package providing MethylCalCalibrationOutlierPlot, which detects outliers via the posterior predictive interval and refits the Bayesian calibration models without them) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend underlying the MethylCal calibration models that generate the posterior predictive distribution)
- **R** (Runtime environment in which the MethylCal package and its outlier-detection function are executed) — https://github.com/lb664/MethylCal

## Examples

```
data(Celiac_data); data = Formatting(Celiac_data, AMP = c(0,12.5,25,37.5,50,62.5,87.5,100), n_Control = 13, n_Case = 34); MethylCalCalibrationOutlierPlot(data, Target = "NFKBIA")
```

## Evaluation signals

- Outliers flagged are exactly the standard-control points outside [Q1 − 1.5·IQR, Q3 + 1.5·IQR] of the posterior predictive interval for their CpG-AMP combination
- The refitted calibration models exclude the flagged CpG-AMP combinations and the calibration plot no longer shows those points in the fit
- Subsequent MethylCalCorrection on the celiac cohort (13 controls, 34 cases) yields corrected methylation degrees within the 0–100% range per individual and CpG/region
- Corrected control/case distributions remain consistent with the corrected levels displayed in the paper's celiac disease figures

## Limitations

- No software version numbers are stated for MethylCal or R, so reproducibility of the exact posterior predictive interval may vary across versions
- Outlier detection depends on the quality and design of the standard-control experiment (e.g. the eight AMP values 0–100% for Celiac_data); sparse or missing AMP levels weaken the posterior predictive estimate
- The IQR-based rule is a fixed heuristic (1.5·IQR fence); borderline control points just inside the interval are retained even if visually suspect

## Evidence

- [results] Outlier detection via posterior predictive interval: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [methods] Refitting after outlier detection: "The function `MethylCalCalibrationOutlierPlot` includes the detection of outliers, where the models are refitted without the detected outliers."
- [abstract] Joint CpG analysis rationale: "which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR)"
- [methods] Bayesian backend: "a Bayesian calibration tool based on INLA"
- [other] Celiac data AMP grid: [evidence span withheld — anchor status: not_found]
