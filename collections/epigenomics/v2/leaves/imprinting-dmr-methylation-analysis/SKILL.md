---
name: imprinting-dmr-methylation-analysis
description: Use when bisulfite amplicon/pyrosequencing methylation data for an imprinted
  DMR (e.g. KCNQ1OT1, H19/IGF2) include standard controls (0%, 25%, 50%, 75%, 100%)
  plus case/control samples and you must correct PCR bias and call hyper/hypo-methylated
  cases.
when_to_use_negative:
- The assay has no standard controls with known methylation percentages (0/25/50/75/100%),
  so no calibration curve can be derived for PCR-bias correction
- The data are genome-wide methylation profiles (e.g. arrays or WGBS) rather than
  targeted bisulfite amplicon/pyrosequencing of a CGI or imprinting DMR — MethylCal
  is designed for targeted assays analysed jointly across CpGs within a DMR
- Only CpG-level 'one-at-a-time' calibration or average-methylation calibration is
  acceptable for the analysis, since MethylCal's joint-DMR approach assumes CpG-level
  variability should be modelled
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: MethylCal
  role: Core R package providing Formatting, calibration/outlier plotting, MethylCalCorrection
    (INLA-based Bayesian calibration and differential methylation test), and TableFormatting
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend (R-INLA) underlying MethylCal's calibration curves;
    installed manually from the INLA repository
- name: R
  role: Runtime environment in which the MethylCal package is loaded and the classification
    workflow is executed
- name: Moskalev's CPR
  role: Alternative correction method used as a comparator to MethylCal's corrected
    methylation levels (MoskalevCorrection / MoskalevCalibrationPlot)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:5f8a7060849ad882e37fa3b62e5e3e9d20472a057e740e6d91713bec865fbe18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# imprinting-dmr-methylation-analysis

## Summary

Reproduce PCR-bias-corrected gain/loss-of-methylation classification of imprinted DMR assays (e.g. KCNQ1OT1 and H19/IGF2 in BWS patients) using the MethylCal R package's INLA-based Bayesian calibration, outlier detection, and 3SD confidence-interval differential methylation testing against healthy controls.

## When to use

Use when bisulfite amplicon/pyrosequencing methylation data for an imprinted DMR (e.g. KCNQ1OT1, H19/IGF2) include standard controls (0%, 25%, 50%, 75%, 100%) plus case/control samples and you must correct PCR bias and call hyper/hypo-methylated cases.

## When NOT to use

- The assay has no standard controls with known methylation percentages (0/25/50/75/100%), so no calibration curve can be derived for PCR-bias correction
- The data are genome-wide methylation profiles (e.g. arrays or WGBS) rather than targeted bisulfite amplicon/pyrosequencing of a CGI or imprinting DMR — MethylCal is designed for targeted assays analysed jointly across CpGs within a DMR
- Only CpG-level 'one-at-a-time' calibration or average-methylation calibration is acceptable for the analysis, since MethylCal's joint-DMR approach assumes CpG-level variability should be modelled

## Inputs

- BWS_data-style methylation table: pyrosequenced/amplicon-sequenced methylation levels for 15 healthy controls and 18 potential BWS patients across targeted DMRs (KCNQ1OT1, H19/IGF2)
- Standard control samples at known methylation percentages AMP = c(0, 25, 50, 75, 100) for deriving calibration curves
- MethylCal R package (github.com/lb664/MethylCal) with R-INLA installed from https://inla.r-inla-download.org/R/stable

## Outputs

- Calibration curve plots and outlier-detection plots per DMR (MethylCalCalibrationPlot / MethylCalCalibrationOutlierPlot)
- Corrected methylation levels for cases and controls per target (corr_data), with printed lists of hyper- and hypo-methylated cases
- Table in the original data format via TableFormatting (output_data)
- Classified patient table of gain/loss-of-methylation calls at the 3SD confidence level (bws_patient_methylation_calls.csv)

## How to apply

Load the MethylCal package (which depends on R-INLA and lattice/latticeExtra) and the BWS_data dataset, then transform the raw methylation table with Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18). Fit INLA-based Bayesian calibration curves for each targeted DMR with MethylCalCalibrationPlot (Target = 'KCNQ1OT1'); for assays with suspect controls use MethylCalCalibrationOutlierPlot (Target = 'H19/IGF2'), which flags outliers via a posterior predictive interval defined by l = Q1 − 1.5IQR and u = Q3 + 1.5IQR, and refit the models without the detected outliers (two outliers are expected in H19/IGF2). Correct case and control apparent methylation levels using MethylCalCorrection (n_Control = 15, n_Case = 18) for each target, which also runs a differential methylation test printing hyper- and hypo-methylated cases, then convert back to the original table format with TableFormatting(corr_data, n_Control = 15, n_Case = 18). Finally classify each patient as gaining or losing methylation by comparing corrected degrees (clipped to [0%, 100%]) against the healthy controls' ±3SD confidence interval, and write the calls to a CSV such as bws_patient_methylation_calls.csv. The rationale is that correcting the PCR-biased apparent methylation levels against the five-standard-control calibration curves gives a clearer separation of borderline patients than raw levels or Moskalev's CPR correction.

## Related tools

- **MethylCal** (Core R package providing Formatting, calibration/outlier plotting, MethylCalCorrection (INLA-based Bayesian calibration and differential methylation test), and TableFormatting) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend (R-INLA) underlying MethylCal's calibration curves; installed manually from the INLA repository)
- **R** (Runtime environment in which the MethylCal package is loaded and the classification workflow is executed)
- **Moskalev's CPR** (Alternative correction method used as a comparator to MethylCal's corrected methylation levels (MoskalevCorrection / MoskalevCalibrationPlot))

## Examples

```
data(BWS_data); data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "H19/IGF2", n_Control = 15, n_Case = 18); output_data = TableFormatting(corr_data, n_Control = 15, n_Case = 18)
```

## Evaluation signals

- In the H19/IGF2 assay, both MethylCal and the comparator method should detect gain of methylation in patients B5A42 and B5B38 (BWS-affected); MethylCal should additionally flag B5B37 as gain of methylation
- MethylCalCalibrationOutlierPlot on H19/IGF2 should report exactly two outliers, and refitted models must exclude them
- Corrected methylation degrees must lie within [0%, 100%] after clipping before classification
- Classification invariants: every healthy control falls inside the ±3SD confidence interval of the control distribution; each of the 18 patients receives exactly one of {gain, loss, no-call} per assay
- TableFormatting(output, n_Control = 15, n_Case = 18) must round-trip the corrected data back into the original BWS_data-style table shape (15 controls + 18 cases)

## Limitations

- No software version numbers are stated for MethylCal, R, or Moskalev's CPR, so exact reproducibility of the published calls may depend on package versions
- No accession or availability information is provided for the BWS case data set itself; users must supply their own case/control methylation table in the same format
- CpGs may need manual removal (e.g. CpGs 21–23 were removed from the PLAGL1 assay by visual inspection), so per-assay data curation is required before calibration
- R-INLA must be installed manually from the INLA repository and installation may take a couple of minutes

## Evidence

- [results] 3SD confidence interval classification on KCNQ1OT1 and H19/IGF2: [evidence span withheld — anchor status: not_found]
- [results] Gain of methylation calls in H19/IGF2: [evidence span withheld — anchor status: not_found]
- [results] Outlier detection via posterior predictive interval: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [results] Calibration from five standard controls: "we derived the calibration curves using five standard controls with known methylation percentages"
- [readme] MethylCalCorrection usage from README example: [evidence span withheld — anchor status: external_doc]
- [readme] INLA-based Bayesian calibration tool: "This package allows the calibration of methylation levels using
**MethylCal**, a Bayesian calibration tool based on INLA"
- [readme] Differential methylation test within MethylCalCorrection: "If cases are provided, MethylCal performs a
differential methylation test to detect hyper- and hypo-methylated
samples."
