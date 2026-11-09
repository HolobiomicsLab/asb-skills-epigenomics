---
name: bisulfite-pcr-bias-correction
description: Use when bisulfite-amplicon/pyrosequencing methylation data include standard
  controls at known methylation percentages (e.g. 0%, 25%, 50%, 75%, 100%) and apparent
  methylation levels must be corrected for PCR bias before case/control classification.
when_to_use_negative:
- The data lack standard controls with known methylation percentages, so no calibration
  curve can be derived from the AMP levels (0%, 25%, 50%, 75%, 100% style series).
- Only average methylation across CpGs is available and per-CpG resolution is absent,
  which prevents the joint all-CpG modelling that MethylCal requires.
- The goal is exploratory visualisation of control methylation levels only (ExploratoryPlot),
  not correction of case/control levels.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MethylCal
  role: Performs the Bayesian calibration curve fitting, outlier detection, and PCR-bias
    correction of methylation levels via MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot,
    MethylCalCorrection and utility functions Formatting/TableFormatting
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Backend for fitting the Bayesian mixed additive calibration model (R-INLA
    dependency that must be installed manually)
- name: R
  role: Implementation language; optimize() numerically minimizes equation (10) to
    invert the calibration curve
- name: Moskalev's CPR
  role: Baseline corrector (equation 9, inversion of cubic polynomial regression)
    exposed as MoskalevCalibrationPlot and MoskalevCorrection
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:17ea5ded2ca695973cff6ef930a321f1ede94ecde3eb9d7625a259104b8cbde5
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
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# bisulfite-pcr-bias-correction

## Summary

Corrects apparent methylation levels measured by bisulfite amplicon sequencing for PCR amplification bias, using MethylCal's Bayesian mixed additive calibration model (fit with INLA) built from five standard controls at known methylation percentages, with Moskalev's CPR and Warnecke's hyperbola as baselines.

## When to use

Use when bisulfite-amplicon/pyrosequencing methylation data include standard controls at known methylation percentages (e.g. 0%, 25%, 50%, 75%, 100%) and apparent methylation levels must be corrected for PCR bias before case/control classification.

## When NOT to use

- The data lack standard controls with known methylation percentages, so no calibration curve can be derived from the AMP levels (0%, 25%, 50%, 75%, 100% style series).
- Only average methylation across CpGs is available and per-CpG resolution is absent, which prevents the joint all-CpG modelling that MethylCal requires.
- The goal is exploratory visualisation of control methylation levels only (ExploratoryPlot), not correction of case/control levels.

## Inputs

- BWS_data-style methylation table: apparent methylation levels per CpG for 15 healthy controls, 18 potential cases, and 5 standard controls at AMP percentages (e.g. 0, 25, 50, 75, 100)
- Vector of actual methylation percentages (AMP) of the standard controls
- Names of the targeted assays/DMRs to correct (e.g. KCNQ1OT1, H19/IGF2)
- n_Control and n_Case sample counts (15 and 18 for the BWS dataset)

## Outputs

- Fitted INLA-based Bayesian calibration model per assay with mlik, DIC and RSS goodness-of-fit metrics
- Corrected methylation degrees per CpG–sample, all within [0%, 100%]
- Printed list of hyper- and hypo-methylated cases from the differential methylation test
- Table in the original data format via TableFormatting (e.g. bws_patient_methylation_calls.csv)

## How to apply

Reshape the methylation table with Formatting() into the MethylCal analysis format, passing the actual AMP vector (e.g. c(0, 25, 50, 75, 100) for BWS_data) and the counts of controls and cases (15 and 18 for the BWS data). Fit the Bayesian calibration model jointly across all CpGs of each targeted DMR/CGI with MethylCalCalibrationPlot (best model selected by DIC among mlik, DIC and RSS metrics), and refit with MethylCalCalibrationOutlierPlot when the posterior predictive outlier rule (l = Q1 − 1.5·IQR, u = Q3 + 1.5·IQR) flags points — e.g. two outliers are expected in the H19/IGF2 assay. Because MethylCal's correction has no analytic solution, fit a cubic-spline interpolation of the posterior means of the AMP random effects (ηi) and, for each CpG–AMP combination, minimize equation (10) with R's optimize() to invert the observed apparent methylation level, yielding the corrected methylation degree; Moskalev's correction (equation 9, inversion of a cubic polynomial regression) is the analytic baseline. Apply MethylCalCorrection (or MoskalevCorrection) to correct case/control levels for each Target (e.g. 'KCNQ1OT1', 'H19/IGF2'), clipping corrected degrees to [0%, 100%], then convert back with TableFormatting. The rationale is that MethylCal jointly models all CpGs within a CGI/DMR, capturing AMP-dependent variability with much narrower 95% prediction credible intervals than one-at-a-time CpG calibration.

## Related tools

- **MethylCal** (Performs the Bayesian calibration curve fitting, outlier detection, and PCR-bias correction of methylation levels via MethylCalCalibrationPlot, MethylCalCalibrationOutlierPlot, MethylCalCorrection and utility functions Formatting/TableFormatting) — https://github.com/lb664/MethylCal
- **INLA** (Backend for fitting the Bayesian mixed additive calibration model (R-INLA dependency that must be installed manually))
- **R** (Implementation language; optimize() numerically minimizes equation (10) to invert the calibration curve)
- **Moskalev's CPR** (Baseline corrector (equation 9, inversion of cubic polynomial regression) exposed as MoskalevCalibrationPlot and MoskalevCorrection)

## Examples

```
data(BWS_data); data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- All corrected methylation degrees fall within [0%, 100%].
- Per-assay goodness-of-fit metrics (mlik, DIC, RSS) are reported for each candidate model and the best model is selected by DIC; the MethylCal curve interpolates standard-control data more precisely than Moskalev's CPR with narrower 95% prediction credible intervals.
- Numerical re-implementation of the optimize()-based inversion matches MethylCalCorrection()/MoskalevCorrection() outputs within numerical tolerance on the same fitted models.
- The H19/IGF2 outlier refit removes two outliers under the l = Q1 − 1.5·IQR / u = Q3 + 1.5·IQR posterior predictive rule.
- On the BWS H19/IGF2 assay, corrected levels reproduce gain-of-methylation calls for B5A42 and B5B38, with MethylCal additionally flagging B5B37.

## Limitations

- No software version numbers are stated for MethylCal, R, or Moskalev's CPR, complicating exact reproduction.
- The MethylCal correction cannot be obtained analytically and requires numerical inversion (cubic-spline interpolation of ηi plus optimize()), whereas Warnecke's hyperbolic method has a closed form.
- Outlier refits can be slow (the H19/IGF2 refit takes several minutes); some CpGs may need manual removal by visual inspection (CpGs 21–23 were removed from PLAGL1).
- No accession or availability information is provided for the BWS case dataset or the celiac case/control dataset beyond the packaged BWS_data/Celiac_data.

## Evidence

- [methods] Bayesian calibration tool based on INLA: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
- [results] Correction of observed methylation degree based on estimated calibration curves using equations (9) and (10): "For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR."
- [results] Outlier detection using posterior predictive interval: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [abstract] MethylCal analyses all CpGs jointly, avoiding one-at-a-time calibration: "which can analyse jointly all CpGs within a CpG island (CGI) or a Differentially Methylated Region (DMR), avoiding 'one-at-a-time' CpG calibration"
- [readme] README example showing the correction invocation and output reshaping: [evidence span withheld — anchor status: external_doc]
