---
name: control-interval-threshold-analysis
description: Use when patient gain/loss-of-methylation calls must be shown to be robust
  (or sensitive) to the choice of differential methylation decision threshold, e.g.
  comparing a ±3SD control interval against α = 0.01.
when_to_use_negative:
- The goal is to compare calibration methods themselves (MethylCal vs Moskalev's CPR
  vs Warnecke) rather than test threshold stability of already-corrected calls
- The data have not been corrected for PCR bias — applying ±3SD/1% classification
  to uncorrected observed methylation levels defeats the purpose of the calibration
  step
- The assay is not a case/control DMR design with a defined healthy control group
  from which a control interval can be estimated
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: MethylCal
  role: R package performing the Bayesian calibration, correction of methylation levels,
    and the differential methylation test whose calls are re-evaluated under both
    thresholds
  repo: https://github.com/lb664/MethylCal
- name: R
  role: Environment in which MethylCal is loaded, thresholds are applied, and the
    comparison CSV is assembled
  repo: https://github.com/lb664/MethylCal
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:63b28a8ae26307b68b3692cc0375f88c7d60f13943313b26757c1bf815b61e86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - MethylCal
  - R
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# control-interval-threshold-analysis

## Summary

A sensitivity-analysis skill for BWS methylation diagnostics that reclassifies patient hyper-/hypo-methylation calls under two decision thresholds (conservative ±3SD control interval vs a less restrictive 1% significance level) on MethylCal-corrected methylation levels.

## When to use

Use when patient gain/loss-of-methylation calls must be shown to be robust (or sensitive) to the choice of differential methylation decision threshold, e.g. comparing a ±3SD control interval against α = 0.01.

## When NOT to use

- The goal is to compare calibration methods themselves (MethylCal vs Moskalev's CPR vs Warnecke) rather than test threshold stability of already-corrected calls
- The data have not been corrected for PCR bias — applying ±3SD/1% classification to uncorrected observed methylation levels defeats the purpose of the calibration step
- The assay is not a case/control DMR design with a defined healthy control group from which a control interval can be estimated

## Inputs

- MethylCal-corrected methylation levels for the KCNQ1OT1 and H19/IGF2 DMR assays (from MethylCalCorrection)
- Raw pyrosequenced/amplicon methylation data for 15 healthy controls and 18 BWS patients (BWS_data)
- Decision thresholds: ±3SD control confidence interval (α = 0.0027) and 1% significance level

## Outputs

- CSV table listing each patient's classification (hypermethylated, hypomethylated, normal) under both the ±3SD and 1% thresholds for KCNQ1OT1 and H19/IGF2
- Per-DMR control mean and standard deviation of corrected methylation levels

## How to apply

Load the MethylCal-corrected methylation levels for the KCNQ1OT1 and H19/IGF2 assays (15 healthy controls and 18 BWS patients) produced by a MethylCalCorrection run, converting the output back to table format with TableFormatting if needed. Compute the mean and standard deviation of the corrected control methylation levels per targeted DMR, then classify each patient as hypermethylated, hypomethylated, or normal using MethylCal's differential methylation test with the conservative +/-3SD control interval (α = 0.0027). Repeat the identical classification with the less restrictive 1% decision threshold, and assemble a CSV table listing each patient's label under both thresholds for both DMRs. The rationale is that MethylCal's calibration sharpens separation of borderline cases, so calls should be stable across thresholds — unlike Moskalev's CPR, whose calls are threshold-sensitive.

## Related tools

- **MethylCal** (R package performing the Bayesian calibration, correction of methylation levels, and the differential methylation test whose calls are re-evaluated under both thresholds) — https://github.com/lb664/MethylCal
- **R** (Environment in which MethylCal is loaded, thresholds are applied, and the comparison CSV is assembled) — https://github.com/lb664/MethylCal

## Examples

```
corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- The set of hyper-/hypo-methylation calls must be identical between the ±3SD (α = 0.0027) and 1% thresholds for MethylCal-corrected data
- Classifications must match the differential methylation test output printed by MethylCalCorrection
- Every patient must receive exactly one of the three labels (hypermethylated, hypomethylated, normal) per DMR and per threshold
- The comparison table must cover all 18 patients for both KCNQ1OT1 and H19/IGF2, with control statistics computed from exactly 15 controls

## Limitations

- Stability of calls under relaxed thresholds is demonstrated for MethylCal-corrected levels; Moskalev's CPR calls are threshold-sensitive, so results do not transfer to that correction method
- The article provides no software version numbers for MethylCal or R, so exact reproducibility requires pinning the GitHub package state
- The BWS case dataset has no accession or availability information beyond the BWS_data object shipped with the R package

## Evidence

- [results] Corrected methylation levels of 15 healthy controls and 18 potential BWS patients were classified using a 3SD confidence interval on the KCNQ1OT1 and H19/IGF2 assays: [evidence span withheld — anchor status: not_found]
- [readme] differential methylation test: "If cases are provided, MethylCal performs a differential methylation test to detect hyper- and hypo-methylated samples."
- [abstract] MethylCal calibration allows clearer identification of patients undergoing loss or gain of methylation in borderline cases: "The calibration of the methylation levels obtained by MethylCal allows a clearer identification of patients undergoing loss or gain of methylation in borderline cases"
- [methods] correction functions: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [readme] TableFormatting usage: [evidence span withheld — anchor status: external_doc]
