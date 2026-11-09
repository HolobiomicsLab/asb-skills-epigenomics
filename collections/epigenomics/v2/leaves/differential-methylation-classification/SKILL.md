---
name: differential-methylation-classification
description: Use when PCR-bias-corrected (MethylCal or Moskalev's CPR) methylation
  levels for case/control pyrosequencing or bisulfite amplicon data must be called
  as gain/loss of methylation against a healthy-control ±3SD interval, e.g. BWS KCNQ1OT1
  and H19/IGF2 calls.
when_to_use_negative:
- Input is raw observed (uncorrected) methylation data — calibration/correction with
  MethylCal must precede classification, since borderline cases are only resolved
  after PCR-bias correction
- Input is CpG island or DMR standard-control calibration data (five controls at 0/25/50/75/100%)
  — that calls for MethylCalCalibrationPlot/MethylCalCalibrationOutlierPlot, not the
  case/control differential test
- The analysis requires a genome-wide methylation profile rather than a small set
  of targeted bisulfite amplicon/pyrosequencing assays
tools:
- name: MethylCal
  role: R package (INLA-based Bayesian calibration) whose MethylCalCorrection function
    performs the differential methylation test flagging hyper- and hypo-methylated
    cases; calls are stable across the ±3SD and 1% thresholds
  repo: https://github.com/lb664/MethylCal
- name: R
  role: Environment in which the MethylCal package runs (library(MethylCal), Formatting,
    MethylCalCorrection, TableFormatting) and where the ±3SD/1% classification table
    is assembled
- name: Moskalev's CPR
  role: Alternative correction method used as a comparator whose gain/loss-of-methylation
    calls are threshold-sensitive, contrasting with MethylCal's stable calls
- name: INLA
  role: Bayesian inference backend underlying MethylCal's calibration curves on which
    the differential methylation test operates
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:0d2274cf0f5a4c6d32e02e7d11e90cc8df5c25dd3bfaaa38caad58f9f31da3a7
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# differential-methylation-classification

## Summary

Classify case samples as hypermethylated, hypomethylated, or normal at a targeted DMR by comparing PCR-bias-corrected methylation levels against a healthy-control confidence interval, using the differential methylation test built into MethylCal's MethylCalCorrection (R package based on INLA).

## When to use

Use when PCR-bias-corrected (MethylCal or Moskalev's CPR) methylation levels for case/control pyrosequencing or bisulfite amplicon data must be called as gain/loss of methylation against a healthy-control ±3SD interval, e.g. BWS KCNQ1OT1 and H19/IGF2 calls.

## When NOT to use

- Input is raw observed (uncorrected) methylation data — calibration/correction with MethylCal must precede classification, since borderline cases are only resolved after PCR-bias correction
- Input is CpG island or DMR standard-control calibration data (five controls at 0/25/50/75/100%) — that calls for MethylCalCalibrationPlot/MethylCalCalibrationOutlierPlot, not the case/control differential test
- The analysis requires a genome-wide methylation profile rather than a small set of targeted bisulfite amplicon/pyrosequencing assays

## Inputs

- MethylCal-corrected methylation levels for case and control samples per targeted DMR (output of MethylCalCorrection, e.g. KCNQ1OT1 and H19/IGF2 with n_Control = 15, n_Case = 18)
- Formatted case/control methylation dataset (e.g. BWS_data via Formatting() with AMP = c(0, 25, 50, 75, 100))

## Outputs

- Per-patient classification table (hypermethylated / hypomethylated / normal) for each DMR under the ±3SD and 1% thresholds, e.g. bws_patient_methylation_calls.csv
- List of hyper- and hypo-methylated cases printed by the differential methylation test in MethylCalCorrection

## How to apply

Load the MethylCal package in R with the formatted case/control data (e.g. BWS_data: 15 healthy controls, 18 potential BWS patients; AMP = c(0, 25, 50, 75, 100)) and correct methylation levels with MethylCalCorrection for each targeted DMR (KCNQ1OT1, H19/IGF2), which prints the hyper- and hypo-methylated cases from its differential methylation test. Compute the mean and standard deviation of the corrected control methylation levels per DMR and classify each case as hypermethylated, hypomethylated, or normal using the conservative ±3SD control confidence interval (α = 0.0027); clip corrected degrees to [0%, 100%]. For robustness, repeat the calls at a less conservative 1% decision threshold — with MethylCal-corrected levels the patient classifications do not change between ±3SD and 1%, whereas Moskalev's CPR calls are threshold-sensitive. Write the per-patient calls under both thresholds (per assay) to a CSV table such as bws_patient_methylation_calls.csv. Convert the corrected object back to the original tabular format with TableFormatting(corr_data, n_Control = 15, n_Case = 18) before classification.

## Related tools

- **MethylCal** (R package (INLA-based Bayesian calibration) whose MethylCalCorrection function performs the differential methylation test flagging hyper- and hypo-methylated cases; calls are stable across the ±3SD and 1% thresholds) — https://github.com/lb664/MethylCal
- **R** (Environment in which the MethylCal package runs (library(MethylCal), Formatting, MethylCalCorrection, TableFormatting) and where the ±3SD/1% classification table is assembled)
- **Moskalev's CPR** (Alternative correction method used as a comparator whose gain/loss-of-methylation calls are threshold-sensitive, contrasting with MethylCal's stable calls)
- **INLA** (Bayesian inference backend underlying MethylCal's calibration curves on which the differential methylation test operates)

## Examples

```
data = Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18)
```

## Evaluation signals

- The CSV of patient calls exists with one classification per patient per DMR (KCNQ1OT1 and H19/IGF2) under both ±3SD and 1% thresholds
- Hyper-/hypo-methylation calls are identical between the ±3SD and 1% thresholds and match the differential methylation test output printed by MethylCalCorrection
- In the H19/IGF2 assay, gain of methylation is detected in BWS patients B5A42 and B5B38, with MethylCal additionally identifying B5B37 as gaining methylation
- Corrected methylation degrees fed into the classification fall within [0%, 100%]
- Control statistics used for the ±3SD interval are computed only from the 15 healthy controls' corrected levels per DMR

## Limitations

- Classification relies on a parametric ±3SD confidence interval built from a small healthy-control set (15 controls in the BWS application), so control-sample quality drives borderline calls
- MethylCal results depend on prior outlier detection and model refitting (e.g. two outliers expected in H19/IGF2 via the Q1 − 1.5·IQR / Q3 + 1.5·IQR posterior predictive interval)
- No software version numbers for MethylCal or Moskalev's CPR, nor version of R, are stated in the article, complicating exact reproduction

## Evidence

- [readme] If cases are provided, MethylCal performs a differential methylation test to detect hyper- and hypo-methylated samples.: "If cases are provided, MethylCal performs a differential methylation test to detect hyper- and hypo-methylated samples."
- [results] Corrected levels classified using a 3SD confidence interval: [evidence span withheld — anchor status: not_found]
- [abstract] Clearer identification in borderline cases: "The calibration of the methylation levels obtained by MethylCal allows a clearer identification of patients undergoing loss or gain of methylation in borderline cases"
- [methods] Main correction functions: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [readme] TableFormatting converts output: [evidence span withheld — anchor status: external_doc]
