---
name: calibrated-methylation-data-analysis
description: Use when analyzing bisulfite amplicon sequencing methylation data (e.g.
  BWS KCNQ1OT1 / H19/IGF2 DMR assays) where PCR bias must be corrected before gain/loss-of-methylation
  calls are made for patient samples.
when_to_use_negative:
- Data are already corrected methylation levels or come from a platform without a
  PCR-bias problem (e.g. epigenome-wide arrays) — recalibration adds no value
- No standard controls with known methylation percentages (0/25/50/75/100%) are available,
  so calibration curves cannot be derived
- The goal is one-at-a-time CpG calibration or calibration of the average methylation
  level, which MethylCal is explicitly designed to avoid
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: MethylCal
  role: Bayesian calibration tool (based on INLA) that derives assay calibration curves,
    corrects methylation levels in case/control samples, and performs the differential
    methylation test
  repo: https://github.com/lb664/MethylCal
- name: R
  role: Implementation language; the skill is executed as an R package workflow (Formatting,
    MethylCalCalibrationPlot, MethylCalCorrection, TableFormatting)
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend used by MethylCal for fitting calibration models
- name: Moskalev's CPR
  role: Alternative threshold-sensitive correction method used as a comparator when
    validating that MethylCal-based classifications are stable across significance
    thresholds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:83063ab2d9631ad053ed557eb4da87b445995fc7ef3ef0de8744578a6600cb6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# calibrated-methylation-data-analysis

## Summary

Applies the MethylCal Bayesian calibration method (R package, INLA-based) to correct PCR-biased methylation levels from bisulfite amplicon sequencing, then classifies case samples as hyper-, hypo-, or normally methylated against healthy-control reference intervals.

## When to use

Use when analyzing bisulfite amplicon sequencing methylation data (e.g. BWS KCNQ1OT1 / H19/IGF2 DMR assays) where PCR bias must be corrected before gain/loss-of-methylation calls are made for patient samples.

## When NOT to use

- Data are already corrected methylation levels or come from a platform without a PCR-bias problem (e.g. epigenome-wide arrays) — recalibration adds no value
- No standard controls with known methylation percentages (0/25/50/75/100%) are available, so calibration curves cannot be derived
- The goal is one-at-a-time CpG calibration or calibration of the average methylation level, which MethylCal is explicitly designed to avoid

## Inputs

- Methylation level matrix for a targeted DMR assay (e.g. BWS_data: KCNQ1OT1 and H19/IGF2 assays, 15 controls, 18 cases)
- Vector of standard control AMPs with known methylation percentages: c(0, 25, 50, 75, 100)
- Number of control and case samples (n_Control = 15, n_Case = 18)

## Outputs

- MethylCal-corrected methylation levels per CpG-AMP combination (via equations (9) and (10))
- Differential methylation test output classifying each case as hypermethylated, hypomethylated, or normal
- Formatted table of corrected methylation levels (TableFormatting output)
- CSV comparison of patient classifications across significance thresholds (±3SD vs 1%)

## How to apply

Load the methylation data (e.g. BWS_data with 15 healthy controls and 18 potential BWS patients) and standard control AMPs (0, 25, 50, 75, 100%) into R and transform it with Formatting(). Calibrate each assay using MethylCalCalibrationPlot (optionally MethylCalCalibrationOutlierPlot to detect and refit without outliers flagged via the posterior predictive interval l = Q1 − 1.5·IQR, u = Q3 + 1.5·IQR), then correct observed methylation levels with MethylCalCorrection, which also runs a differential methylation test when cases are supplied. Classify patients as hyper-, hypo-, or normally methylated using the ±3SD control interval (α = 0.0027); validate threshold robustness by repeating classification at the less conservative 1% (α = 0.01) level and confirming calls are unchanged. Convert output with TableFormatting() to the original data format for downstream comparison against uncorrected or Moskalev-corrected calls.

## Related tools

- **MethylCal** (Bayesian calibration tool (based on INLA) that derives assay calibration curves, corrects methylation levels in case/control samples, and performs the differential methylation test) — https://github.com/lb664/MethylCal
- **R** (Implementation language; the skill is executed as an R package workflow (Formatting, MethylCalCalibrationPlot, MethylCalCorrection, TableFormatting)) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend used by MethylCal for fitting calibration models)
- **Moskalev's CPR** (Alternative threshold-sensitive correction method used as a comparator when validating that MethylCal-based classifications are stable across significance thresholds)

## Examples

```
corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Corrected methylation levels for the 15 healthy controls yield a mean ± SD reference interval that reproduces the differential methylation test printed by MethylCalCorrection
- Patient classifications under the ±3SD (α = 0.0027) rule are identical to those under the 1% (α = 0.01) threshold — no call changes for KCNQ1OT1 or H19/IGF2
- Formatting() and TableFormatting() round-trip preserves the row count of 15 controls + 18 cases with the same sample identifiers
- Outlier detection via the posterior predictive interval (Q1 − 1.5·IQR, Q3 + 1.5·IQR) removes only CpGs flagged as outliers, and refitting without them changes calibration curves only in borderline CpGs

## Limitations

- No software version numbers are stated for MethylCal or Moskalev's CPR, and no version of R or the MethylCal package is given
- No accession or availability information is provided for the BWS case data set or the celiac case/control data set
- R-INLA must be installed manually (install.packages('INLA', repos = 'https://inla.r-inla-download.org/R/stable')) and may take a couple of minutes
- Some CpGs may need manual removal (e.g. CpGs 21–23 removed from the PLAGL1 assay by visual inspection), introducing subjective judgment

## Evidence

- [methods] The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`.: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [results] The observed methylation levels of 15 healthy controls and 18 potential BWS patients were corrected using the calibration curves obtained by MethylCal and Moskalev’s CPR: "The observed methylation levels of 15 healthy controls and 18 potential BWS patients were corrected using the calibration curves obtained by MethylCal and Moskalev’s CPR"
- [readme] If cases are provided, MethylCal performs a differential methylation test to detect hyper- and hypo-methylated samples.: "If cases are provided, MethylCal performs a
differential methylation test to detect hyper- and hypo-methylated samples."
- [readme] data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18): "data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)"
- [methods] The function `MethylCalCalibrationOutlierPlot` includes the detection of outliers, where the models are refitted without the detected outliers.: "The function `MethylCalCalibrationOutlierPlot` includes the detection of outliers, where the models are refitted without the detected outliers."
