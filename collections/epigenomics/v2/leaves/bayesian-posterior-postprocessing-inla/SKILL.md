---
name: bayesian-posterior-postprocessing-inla
description: Use when you have fitted MethylCal/INLA calibration models per assay
  (e.g. from BWS standard controls at 0/25/50/75/100% methylation) and must numerically
  convert observed apparent methylation levels into corrected methylation degrees.
when_to_use_negative:
- You need the analytic hyperbolic calibration/inversion of Warnecke et al. (1997)
  rather than MethylCal's spline-based numeric inversion
- You only need calibration-curve visualisation or goodness-of-fit comparison (use
  MethylCalCalibrationPlot / MoskalevCalibrationPlot) rather than correcting case/control
  methylation levels
- Calibration models have not yet been fitted with INLA — the interpolant requires
  posterior means of the AMP random effects from fitted models
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MethylCal
  role: Package whose fitted calibration models are post-processed; MethycalCorrection/MethylCalCorrection
    implements the spline interpolation and optimize()-based inversion of equation
    (10)
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend supplying the posterior means of the AMP random
    effects used to build the cubic-spline interpolant
  repo: http://www.r-inla.org/
- name: R
  role: Implementation language; optimize() performs the numeric minimisation of the
    objective function
- name: Moskalev's CPR
  role: 'Baseline corrector: inversion of the cubic polynomial regression (equation
    (9)) computed on the same fitted controls'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:e1b96561b6521a0decde58e79695ed0ea85f20d1c6329e5bc34e209886bd0639
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bayesian-posterior-postprocessing-inla

## Summary

Post-process fitted INLA posterior distributions from MethylCal's Bayesian methylation calibration: build a cubic-spline interpolant of the posterior means of the AMP random effects and numerically invert it (via optimize() minimisation of objective function (10)) to recover PCR-bias-corrected methylation degrees that cannot be obtained analytically.

## When to use

Use when you have fitted MethylCal/INLA calibration models per assay (e.g. from BWS standard controls at 0/25/50/75/100% methylation) and must numerically convert observed apparent methylation levels into corrected methylation degrees.

## When NOT to use

- You need the analytic hyperbolic calibration/inversion of Warnecke et al. (1997) rather than MethylCal's spline-based numeric inversion
- You only need calibration-curve visualisation or goodness-of-fit comparison (use MethylCalCalibrationPlot / MoskalevCalibrationPlot) rather than correcting case/control methylation levels
- Calibration models have not yet been fitted with INLA — the interpolant requires posterior means of the AMP random effects from fitted models

## Inputs

- Fitted MethylCal INLA model objects per assay (from standard control samples)
- Formatted methylation data produced by Formatting(), e.g. BWS_data with AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18
- Observed apparent methylation levels for case/control samples (e.g. KCNQ1OT1 and H19/IGF2 BWS assays, celiac NFkB-related gene targets)

## Outputs

- Corrected methylation degrees per CpG-AMP combination for each case/control sample
- Moskalev CPR-corrected methylation degrees (equation (9)) as baseline comparison
- Table-formatted output (via TableFormatting) matching the uploaded data set format

## How to apply

Load the fitted INLA calibration models and the Formatting()-processed methylation data (e.g. BWS_data with AMP = c(0, 25, 50, 75, 100)). For each CpG-AMP combination, fit a cubic-spline interpolant to the posterior means of the AMP random effects across the actual methylation percentages, so that the calibration curve can predict the apparent level for any candidate actual methylation percentage. Then, for each observed apparent methylation level, use R's optimize() to minimise objective function (10) — the discrepancy between the observed level and the spline-predicted calibration curve — yielding the corrected methylation degree, which unlike Warnecke's hyperbolic method has no analytic inverse. Compute Moskalev's CPR baseline (equation (9), inversion of the cubic polynomial regression) on the same fitted controls for comparison. Verify that every corrected methylation degree falls within [0%, 100%].

## Related tools

- **MethylCal** (Package whose fitted calibration models are post-processed; MethycalCorrection/MethylCalCorrection implements the spline interpolation and optimize()-based inversion of equation (10)) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend supplying the posterior means of the AMP random effects used to build the cubic-spline interpolant) — http://www.r-inla.org/
- **R** (Implementation language; optimize() performs the numeric minimisation of the objective function)
- **Moskalev's CPR** (Baseline corrector: inversion of the cubic polynomial regression (equation (9)) computed on the same fitted controls)

## Examples

```
corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Every corrected methylation degree lies within [0%, 100%]
- Recovered corrected values match the package's MethycalCorrection()/MoskalevCorrection() outputs on the same fitted models within numerical tolerance
- The optimize() minimisation converges for every CpG-AMP combination (no non-finite or boundary-stuck solutions)
- Moskalev-corrected (equation (9)) and MethylCal-corrected (equation (10)) results are both produced on the same fitted controls, enabling baseline comparison
- Output can be round-tripped through TableFormatting() into the same format as the uploaded data set

## Limitations

- No software version numbers are stated for MethylCal or Moskalev's CPR, limiting reproducibility of the numeric inversion
- The correction is only defined relative to fitted calibration curves from standard controls (0%, 25%, 50%, 75%, 100%); it requires prior INLA model fitting and cannot run on raw methylation data alone
- Outliers detected by the posterior predictive interval (l = Q1 − 1.5IQR, u = Q3 + 1.5IQR) or removed CpGs (e.g. CpGs 21–23 in PLAGL1) affect the fitted posteriors and hence the spline and inversion

## Evidence

- [methods] MethylCal is a Bayesian tool based on INLA that post-processes posterior means of AMP random effects: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [results] Equations (9) and (10) correct the apparent methylation level after PCR for each CpG-AMP combination: "For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR."
- [methods] Package functions MoskalevCorrection and MethycalCorrection perform the correction of case/control methylation levels: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [readme] README example showing MethylCalCorrection applied to formatted BWS data with AMP levels 0–100: [evidence span withheld — anchor status: external_doc]
- [readme] README states MethylCal is a Bayesian calibration tool based on INLA with correction of methylation levels in case and control samples: "This package allows the calibration of methylation levels using
**MethylCal**, a Bayesian calibration tool based on INLA"
