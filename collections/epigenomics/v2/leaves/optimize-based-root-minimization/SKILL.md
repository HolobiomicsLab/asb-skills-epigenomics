---
name: optimize-based-root-minimization
description: Use when fitted INLA-based MethylCal calibration models must be inverted
  numerically to recover corrected methylation degrees from observed apparent methylation
  levels, because the correction (eq. 10) has no closed-form solution unlike Warnecke's
  or Moskalev's CPR (eq. 9).
when_to_use_negative:
- When the calibration method has an analytical inverse — e.g. Warnecke's hyperbolic
  method or Moskalev's cubic polynomial regression (equation 9) — which can be solved
  directly without numerical optimization
- When only exploratory calibration-curve visualisation is needed (use MethylCalCalibrationPlot
  / ExploratoryPlot rather than invoking the inversion)
- When the fitted calibration model or the formatted standard-control data are absent
  — the spline of posterior AMP random-effect means cannot be constructed
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3674
tools:
- name: MethylCal
  role: R package providing the fitted INLA calibration models, the MethycalCorrection
    routine, and the cubic-spline-based objective (equation 10) that is numerically
    inverted
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference backend producing the posterior means of the AMP random
    effects that feed the spline interpolant
- name: R
  role: Language in which optimize() performs the one-dimensional minimization of
    equation (10) per CpG-AMP combination
- name: Moskalev's CPR
  role: Baseline cubic polynomial regression corrector (equation 9) whose analytical
    inversion is computed for comparison on the same fitted controls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:130bbe65cd4ab8a09a3bc80d6291618f15cf4a3e089fdfb65d352dfc96462bb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - Moskalev's CPR
  - R
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# optimize-based-root-minimization

## Summary

Numerically inverts MethylCal's calibration curves by minimizing the discrepancy between the observed apparent methylation level and a cubic-spline-predicted calibration curve with R's optimize(), yielding the PCR-bias corrected methylation degree per CpG-AMP combination (equation (10)), where no analytical inverse exists as in Warnecke's hyperbolic method.

## When to use

Use when fitted INLA-based MethylCal calibration models must be inverted numerically to recover corrected methylation degrees from observed apparent methylation levels, because the correction (eq. 10) has no closed-form solution unlike Warnecke's or Moskalev's CPR (eq. 9).

## When NOT to use

- When the calibration method has an analytical inverse — e.g. Warnecke's hyperbolic method or Moskalev's cubic polynomial regression (equation 9) — which can be solved directly without numerical optimization
- When only exploratory calibration-curve visualisation is needed (use MethylCalCalibrationPlot / ExploratoryPlot rather than invoking the inversion)
- When the fitted calibration model or the formatted standard-control data are absent — the spline of posterior AMP random-effect means cannot be constructed

## Inputs

- Fitted MethylCal calibration INLA model objects (per assay, from MethylCalCalibrationPlot on standard controls)
- Formatting()-processed methylation data (e.g. BWS_data with AMP = c(0, 25, 50, 75, 100), n_Control = 15, n_Case = 18)
- Posterior means of the AMP random effects (ηi) at the five standard control methylation percentages

## Outputs

- Corrected methylation degree per CpG-AMP combination for case/control samples, constrained to [0%, 100%]
- Moskalev CPR (equation 9) corrected levels for comparison
- Table-formatted output via TableFormatting() matching the original data format

## How to apply

Load the fitted MethylCal INLA model objects for the assay together with the Formatting()-processed methylation data (e.g. BWS_data with AMP = c(0, 25, 50, 75, 100)). Build a cubic-spline interpolant of the posterior means of the AMP random effects (ηi) across the actual methylation percentages of the five standard controls; this spline predicts η(xj) for any actual methylation percentage. For each CpG-AMP combination, call R's optimize() to minimize the objective function (10), the squared discrepancy between the observed apparent methylation level and the spline-predicted calibration curve, over the [0, 100]% actual-methylation grid; the minimizer is the corrected methylation degree. Run the baseline Moskalev correction (equation 9, inversion of the cubic polynomial regression) on the same fitted controls for comparison, and verify every corrected value lies within [0%, 100%].

## Related tools

- **MethylCal** (R package providing the fitted INLA calibration models, the MethycalCorrection routine, and the cubic-spline-based objective (equation 10) that is numerically inverted) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference backend producing the posterior means of the AMP random effects that feed the spline interpolant)
- **R** (Language in which optimize() performs the one-dimensional minimization of equation (10) per CpG-AMP combination)
- **Moskalev's CPR** (Baseline cubic polynomial regression corrector (equation 9) whose analytical inversion is computed for comparison on the same fitted controls)

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18); corr_data = MethylCalCorrection(data, Target = "KCNQ1OT1", n_Control = 15, n_Case = 18, printing = FALSE)
```

## Evaluation signals

- Every corrected methylation degree returned by optimize() falls within [0%, 100%]
- Corrected values match the package's MethycalCorrection()/MethylCalCorrection() outputs within numerical tolerance on the same fitted models
- Moskalev equation (9) inversion reproduces the package's MoskalevCorrection() values within tolerance
- The optimize() minimizer is unique and interior to the [0, 100] search interval for each CpG-AMP combination (no boundary artifacts)
- Re-running Formatting + MethylCalCorrection on BWS_data with Target = "KCNQ1OT1" reproduces Figure 3 C&D of Ochoa et al. (2019)

## Limitations

- The spline is built only on the five standard control percentages (0%, 25%, 50%, 75%, 100%) for the BWS assays; prediction outside this range relies entirely on interpolation behaviour
- Optimization is iterative — results depend on numerical tolerance and the optimize() convergence, unlike the closed-form Moskalev/Warnecke inversions
- Outlier CpGs or CpGs removed from the fit (e.g. CpGs 21–23 of the PLAGL1 assay removed by visual inspection) degrade the spline and the corrected values; refit without outliers first (MethylCalCalibrationOutlierPlot)
- No software version numbers are stated for MethylCal or Moskalev's CPR in the article, hindering exact reproduction

## Evidence

- [results] For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR.: "For each CpG-AMP combination, (9) and (10) are used to correct the apparent level of methylation after PCR."
- [full_text] optimize() minimizes equation (10): "we derived the calibration curves using five standard controls with known methylation percentages"
- [methods] correction functions named: "The main functions for the correction of the methylation levels in case and control samples are `MoskalevCorrection` and `MethycalCorrection`."
- [readme] README example invocation: [evidence span withheld — anchor status: external_doc]
- [readme] README data formatting step: "data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)"
