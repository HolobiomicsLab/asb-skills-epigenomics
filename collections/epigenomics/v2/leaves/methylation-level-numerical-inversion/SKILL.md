---
name: methylation-level-numerical-inversion
description: Use when fitted MethylCal INLA calibration models and formatted BWS-style
  methylation data are available and the apparent methylation level of case/control
  samples must be converted back to a corrected methylation degree per CpG-AMP combination.
when_to_use_negative:
- Only the calibration curves themselves need to be visualised or goodness-of-fit
  checked — use MethylCalCalibrationPlot rather than the correction step
- Apparent methylation levels come from a platform without AMP calibration controls
  (no 0%/25%/50%/75%/100% standards), so no spline-based inversion can be anchored
- The analysis goal is outlier detection or exploratory plotting of standard control
  samples (ExploratoryPlot, MethylCalCalibrationOutlierPlot), not correction of case/control
  levels
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: MethylCal
  role: Provides MethylCalCorrection()/MethycalCorrection() which implement the spline-based
    numerical inversion of equation (10) and the fitted calibration curves being inverted
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian backend producing the INLA model objects whose posterior means of
    the AMP random effects (ηi) feed the cubic-spline interpolant
- name: R
  role: Implementation language; optimize() is used to minimize the objective function
    (10) for each CpG-AMP combination
- name: Moskalev's CPR
  role: 'Baseline corrector: inversion of the cubic polynomial regression (equation
    9) computed on the same fitted controls'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:625ed90f664bbcbbff17a3e3b38a588834f2cb0c53185ca8e4cf6308de683242
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# methylation-level-numerical-inversion

## Summary

Recovers the PCR-bias corrected methylation degree from fitted MethylCal calibration curves by numerically inverting equation (10): a cubic-spline interpolant of the posterior means of the AMP random effects (ηi) is built over the five standard control percentages (0%, 25%, 50%, 75%, 100%) and R's optimize() minimizes the discrepancy between observed and spline-predicted apparent methylation, with Moskalev's CPR inversion (equation 9) as the baseline corrector.

## When to use

Use when fitted MethylCal INLA calibration models and formatted BWS-style methylation data are available and the apparent methylation level of case/control samples must be converted back to a corrected methylation degree per CpG-AMP combination.

## When NOT to use

- Only the calibration curves themselves need to be visualised or goodness-of-fit checked — use MethylCalCalibrationPlot rather than the correction step
- Apparent methylation levels come from a platform without AMP calibration controls (no 0%/25%/50%/75%/100% standards), so no spline-based inversion can be anchored
- The analysis goal is outlier detection or exploratory plotting of standard control samples (ExploratoryPlot, MethylCalCalibrationOutlierPlot), not correction of case/control levels

## Inputs

- Fitted MethylCal calibration models (INLA model objects) from the standard controls
- Formatted methylation data produced by Formatting(), e.g. BWS_data with AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18
- Observed apparent methylation levels per CpG-AMP combination for case and control samples

## Outputs

- Corrected methylation degrees per CpG-AMP combination for case/control samples, all within [0%, 100%]
- Table of corrected values re-formatted via TableFormatting() to the uploaded data format
- Baseline corrected values from Moskalev's CPR inversion (equation 9) for comparison

## How to apply

Load the fitted MethylCal calibration models (INLA model objects) together with the Formatting()-processed BWS data. Construct a cubic-spline interpolant of the posterior means of the AMP random effects (ηi) across the actual methylation percentages (0%, 25%, 50%, 75%, 100% for the BWS assays), enabling prediction of η(xj) for any actual methylation percentage. For each CpG-AMP combination, use R's optimize() to minimize the discrepancy between the observed apparent methylation level and the spline-predicted calibration curve (equation 10); unlike Warnecke's hyperbolic method this correction cannot be obtained analytically. Implement equation (9) — inversion of Moskalev's cubic polynomial regression — on the same fitted controls as the baseline corrector. Run both correctors on the case/control samples using the package's MethycalCorrection/MoskalevCorrection functions and verify that every corrected methylation degree lies within [0%, 100%].

## Related tools

- **MethylCal** (Provides MethylCalCorrection()/MethycalCorrection() which implement the spline-based numerical inversion of equation (10) and the fitted calibration curves being inverted) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian backend producing the INLA model objects whose posterior means of the AMP random effects (ηi) feed the cubic-spline interpolant)
- **R** (Implementation language; optimize() is used to minimize the objective function (10) for each CpG-AMP combination)
- **Moskalev's CPR** (Baseline corrector: inversion of the cubic polynomial regression (equation 9) computed on the same fitted controls)

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Every corrected methylation degree falls within [0%, 100%]
- Recovered corrected methylation degrees match the package's MethylCalCorrection()/MoskalevCorrection() outputs on the same fitted models within numerical tolerance
- Corrections are produced for every CpG-AMP combination present in the formatted data (no missing combinations)
- Corrected levels of the 15 healthy controls and 18 potential BWS patients on the KCNQ1OT1 and H19/IGF2 assays reproduce the 3SD classification of borderline loss/gain of methylation cases
- Both correctors (spline-based equation 10 and Moskalev's equation 9) run on the same fitted controls so their outputs are directly comparable

## Limitations

- Unlike Warnecke's hyperbolic method, the MethylCal correction has no closed-form solution and depends on optimize()-based numerical inversion, so results carry numerical tolerance rather than exactness
- The spline interpolant is anchored only at the five standard control percentages (0%, 25%, 50%, 75%, 100%); accuracy at actual methylation percentages not in the controlled experiment relies on the spline's predictive extrapolation
- No software version numbers for MethylCal or Moskalev's CPR are stated in the article, and no accession is provided for the BWS or celiac case/control datasets, limiting exact reproduction

## Evidence

- [results] equations used for numerical inversion of apparent methylation: "For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR."
- [methods] package functions performing the correction: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [readme] concrete invocation on the BWS dataset from the README: [evidence span withheld — anchor status: external_doc]
- [intro] control design anchoring the spline interpolant: "we designed eight independent assays in two CGIs located on SDHC gene promoter and six imprinted DMRs"
