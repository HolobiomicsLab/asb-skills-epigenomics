---
name: cubic-spline-interpolation-of-random-effects
description: Use when fitted MethylCal/INLA calibration models and observed apparent
  methylation levels from bisulfite amplicon sequencing are available, and the PCR-corrected
  methylation degree must be recovered numerically rather than analytically.
when_to_use_negative:
- When the calibration method is Warnecke's hyperbolic approach, whose inversion is
  analytic and does not require spline interpolation plus numerical optimization
- When only exploratory visualization of calibration curves is needed (use ExploratoryPlot
  or MethylCalCalibrationPlot instead of correction)
- When the data have not been through the Formatting() step and no fitted INLA calibration
  model exists for the assay
tools:
- name: MethylCal
  role: Provides MethycalCorrection/MethylCalCorrection, which wraps the spline-interpolated
    random-effects inversion of equation (10); its fitted INLA models supply the posterior
    means of the AMP random effects
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend that fits the calibration models and yields the
    posterior means of the AMP random effects that the cubic spline interpolates
- name: R
  role: Implementation language; optimize() performs the one-dimensional numerical
    minimization of equation (10) per CpG-AMP combination
- name: Moskalev's CPR
  role: Baseline corrector whose equation (9) cubic-polynomial inversion is computed
    on the same fitted controls for comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:2902fb374182c68e33fc977c913e7c8fe4e306e6664f8bdf9b740fc869d94e0b
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cubic-spline-interpolation-of-random-effects

## Summary

Numerically inverts MethylCal's PCR-bias calibration by fitting a cubic spline through the posterior means of the AMP random effects (η_i) and minimizing objective function (10) with R's optimize(), yielding the corrected methylation degree per CpG-AMP combination. Unlike Warnecke's hyperbolic method, MethylCal's correction has no closed form, so this spline-plus-optimization procedure is required.

## When to use

Use when fitted MethylCal/INLA calibration models and observed apparent methylation levels from bisulfite amplicon sequencing are available, and the PCR-corrected methylation degree must be recovered numerically rather than analytically.

## When NOT to use

- When the calibration method is Warnecke's hyperbolic approach, whose inversion is analytic and does not require spline interpolation plus numerical optimization
- When only exploratory visualization of calibration curves is needed (use ExploratoryPlot or MethylCalCalibrationPlot instead of correction)
- When the data have not been through the Formatting() step and no fitted INLA calibration model exists for the assay

## Inputs

- Fitted MethylCal calibration models (INLA model objects) from the standard-control calibration step
- Formatted methylation data from Formatting(), e.g. BWS_data with AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18
- Observed apparent methylation levels for case/control samples per CpG-AMP combination
- Calibration-curve control points: five standard controls with known methylation percentages (0%, 25%, 50%, 75%, 100%)

## Outputs

- Corrected methylation degree (numeric value in [0%, 100%]) for each CpG-AMP combination and each case/control sample
- Baseline corrected values from equation (9) (Moskalev's CPR inversion) for comparison

## How to apply

Load the fitted MethylCal calibration models (INLA model objects, e.g. from MethylCalCalibrationPlot) together with data formatted via Formatting(), where AMP = c(0, 25, 50, 75, 100) for the five standard controls. Construct a cubic-spline interpolant of the posterior means of the AMP random effects (η_i) across the actual methylation percentages, so that η(x_j) can be predicted at any methylation value. For each CpG-AMP combination, call R's optimize() to minimize equation (10), the discrepancy between the observed apparent methylation level and the spline-predicted calibration curve; the minimizing argument is the corrected methylation degree. Implement equation (9) — inversion of Moskalev's cubic polynomial regression — as the analytic baseline corrector on the same fitted controls. Validate by comparing against the package's MethycalCorrection/MoskalevCorrection outputs and checking every corrected value falls within [0%, 100%].

## Related tools

- **MethylCal** (Provides MethycalCorrection/MethylCalCorrection, which wraps the spline-interpolated random-effects inversion of equation (10); its fitted INLA models supply the posterior means of the AMP random effects) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend that fits the calibration models and yields the posterior means of the AMP random effects that the cubic spline interpolates)
- **R** (Implementation language; optimize() performs the one-dimensional numerical minimization of equation (10) per CpG-AMP combination)
- **Moskalev's CPR** (Baseline corrector whose equation (9) cubic-polynomial inversion is computed on the same fitted controls for comparison)

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Recovered corrected methylation degrees match the package's MethycalCorrection()/MethylCalCorrection() output on the same fitted models within numerical tolerance
- Every corrected methylation degree falls within [0%, 100%]
- Baseline values from the equation (9) Moskalev inversion computed on the same fitted controls are consistent with MoskalevCorrection() output
- Running the example pipeline reproduces Figure 3 C&D of Ochoa et al. (2019) on BWS_data with Target = "KCNQ1OT1"
- The optimizer converges for every CpG-AMP combination (no failed optimize() calls), given controls at AMPs 0, 25, 50, 75 and 100

## Limitations

- No analytic solution exists: the correction depends on numerical minimization via optimize(), so results are tolerance-dependent and require a well-behaved spline interpolant
- Outliers must be handled before correction — models may need refitting without detected outliers (e.g. CpGs 21–23 were removed from the PLAGL1 assay by visual inspection)
- No software version numbers are stated for MethylCal or Moskalev's CPR, and R-INLA must be installed manually, complicating exact reproduction

## Evidence

- [results] For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR.: "For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR."
- [methods] The main correction functions are named in the methods: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [results] Calibration curves derive from five standard controls: "we derived the calibration curves using five standard controls with known methylation percentages"
- [readme] README identifies MethylCal as an INLA-based calibration tool that corrects methylation levels: "This package allows the calibration of methylation levels using **MethylCal**, a Bayesian calibration tool based on INLA"
