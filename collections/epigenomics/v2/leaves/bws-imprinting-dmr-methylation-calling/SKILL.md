---
name: bws-imprinting-dmr-methylation-calling
description: Use when bisulfite amplicon methylation data from cases and controls
  at imprinted DMRs (e.g., KCNQ1OT1, H19/IGF2) need PCR-bias-corrected patient calls
  (gain/loss of methylation), or when assessing sensitivity of those calls to the
  decision threshold (±3SD vs 1%).
when_to_use_negative:
- The methylation levels are uncorrected observed (apparent) values — apply MethylCal
  calibration (calibration curves from the five standard controls at 0%, 25%, 50%,
  75%, 100%) first, since the classification is defined on corrected levels
- The goal is comparing calibration methods themselves (e.g., MethylCal vs Moskalev's
  CPR vs Warnecke) rather than making patient calls — use the calibration/correction
  comparison workflow instead
- Data are from CpG islands (e.g., the SDHC promoter CGIs) or non-imprinting assays
  in the celiac data set — this skill is specific to the BWS imprinting DMR classification
  task
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: MethylCal
  role: R package providing MethylCalCorrection, which corrects observed methylation
    levels via calibration curves and performs the differential methylation test used
    to flag hyper-/hypo-methylated patients
  repo: https://github.com/lb664/MethylCal
- name: R
  role: Environment in which the MethylCal package runs and where control mean/SD,
    ±3SD vs 1% classification, and the comparison CSV are computed
- name: INLA
  role: R-INLA dependency backing the Bayesian calibration models of MethylCal
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:7e27c619403e4117e5fb3ee687015d6515affea8d3e4456a5c5e2df5a31ac4eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bws-imprinting-dmr-methylation-calling

## Summary

Calls hyper- and hypo-methylation in Beckwith-Wiedemann syndrome (BWS) patients by classifying MethylCal-corrected methylation levels at the KCNQ1OT1 and H19/IGF2 imprinting DMRs against a healthy-control reference interval (±3SD), and tests whether calls are robust to relaxing the significance threshold to α = 0.01.

## When to use

Use when bisulfite amplicon methylation data from cases and controls at imprinted DMRs (e.g., KCNQ1OT1, H19/IGF2) need PCR-bias-corrected patient calls (gain/loss of methylation), or when assessing sensitivity of those calls to the decision threshold (±3SD vs 1%).

## When NOT to use

- The methylation levels are uncorrected observed (apparent) values — apply MethylCal calibration (calibration curves from the five standard controls at 0%, 25%, 50%, 75%, 100%) first, since the classification is defined on corrected levels
- The goal is comparing calibration methods themselves (e.g., MethylCal vs Moskalev's CPR vs Warnecke) rather than making patient calls — use the calibration/correction comparison workflow instead
- Data are from CpG islands (e.g., the SDHC promoter CGIs) or non-imprinting assays in the celiac data set — this skill is specific to the BWS imprinting DMR classification task

## Inputs

- MethylCal-corrected methylation levels for the KCNQ1OT1 and H19/IGF2 assays (15 healthy controls and 18 BWS patients) from a MethylCalCorrection run
- BWS_data data set (pyrosequenced methylation levels) from the MethylCal R package
- AMP vector of standard control methylation percentages: c(0, 25, 50, 75, 100)

## Outputs

- Per-patient classification table (hypermethylated / hypomethylated / normal) for KCNQ1OT1 and H19/IGF2 under both the ±3SD and 1% thresholds
- CSV table listing each patient's classification under both thresholds per targeted DMR
- Control reference statistics (mean and SD of corrected methylation levels of the 15 healthy controls per DMR)

## How to apply

Load the MethylCal-corrected methylation levels produced by MethylCalCorrection (e.g., data(BWS_data); Formatting with AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18; then MethylCalCorrection with Target = "KCNQ1OT1" or "H19/IGF2"). Compute the mean and standard deviation of the corrected levels of the 15 healthy controls per DMR, then classify each of the 18 patients as hypermethylated, hypomethylated, or normal using the differential methylation test: outside the conservative ±3SD control interval (α = 0.0027) indicates gain/loss of methylation. Repeat the classification with the less restrictive 1% (α = 0.01) threshold and assemble a CSV table of each patient's classification under both thresholds for KCNQ1OT1 and H19/IGF2. The rationale is that MethylCal calibration permits a clearer identification of loss/gain of methylation in borderline cases; validation requires that hyper-/hypo-methylation calls be identical between the ±3SD and 1% thresholds, matching the differential methylation test output printed by MethylCalCorrection.

## Related tools

- **MethylCal** (R package providing MethylCalCorrection, which corrects observed methylation levels via calibration curves and performs the differential methylation test used to flag hyper-/hypo-methylated patients) — https://github.com/lb664/MethylCal
- **R** (Environment in which the MethylCal package runs and where control mean/SD, ±3SD vs 1% classification, and the comparison CSV are computed)
- **INLA** (R-INLA dependency backing the Bayesian calibration models of MethylCal)

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- The hyper-/hypo-methylation calls are identical between the ±3SD and 1% thresholds for MethylCal-corrected levels (threshold-robustness check)
- The classification matches the differential methylation test output printed by MethylCalCorrection
- Formatting of inputs matches the expected structure: Formatting(BWS_data, AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18), with exactly 15 controls and 18 cases
- Output of MethylCalCorrection round-trips through TableFormatting into the same data format as the uploaded data set
- Each of the 18 patients receives exactly one of: hypermethylated, hypomethylated, or normal, per DMR

## Limitations

- Threshold sensitivity differs by correction method: MethylCal-based calls are robust to relaxing the threshold, whereas Moskalev's CPR calls are threshold-sensitive
- No software version numbers for MethylCal, Moskalev's CPR, or R are stated, and no accession/availability information is provided for the BWS case data set
- The differential methylation test relies on a control-derived reference interval, so control sample size (15 healthy controls here) directly affects the mean/SD used for calling

## Evidence

- [results] Threshold robustness of MethylCal calls vs Moskalev CPR: "The observed methylation levels of 15 healthy controls and 18 potential BWS patients were corrected using the calibration curves obtained by MethylCal and Moskalev’s CPR"
- [readme] Differential methylation test capability: "If cases are provided, MethylCal performs a differential methylation test to detect hyper- and hypo-methylated samples."
- [abstract] Borderline-case clarity rationale: "The calibration of the methylation levels obtained by MethylCal allows a clearer identification of patients undergoing loss or gain of methylation in borderline cases"
- [readme] Concrete invocation for the KCNQ1OT1 assay: [evidence span withheld — anchor status: external_doc]
- [readme] Rounding output back to the data format: "output_data = TableFormatting(corr_data, n_Control = 15, n_Case = 18)"
