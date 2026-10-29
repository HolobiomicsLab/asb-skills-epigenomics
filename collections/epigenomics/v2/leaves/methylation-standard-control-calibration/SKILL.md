---
name: methylation-standard-control-calibration
description: Use when bisulfite amplicon sequencing standard controls with known methylation
  percentages (e.g. 0/25/50/75/100% BWS or 0/12.5/25/37.5/50/62.5/87.5/100% Celiac)
  are available and you must pick the MethylCal model specification per assay.
when_to_use_negative:
- When the goal is to correct apparent methylation levels in case/control samples
  using an already-chosen calibration curve — use MethycalCorrection or MoskalevCorrection
  instead of re-running model selection.
- When standard controls with known methylation percentages are not available for
  the assay; the calibration curves are derived from these controls and cannot be
  estimated without them.
- When a one-at-a-time CpG calibration (e.g. Moskalev's CPR) is required; MethylCal
  deliberately analyses all CpGs within a CGI/DMR jointly rather than calibrating
  individual CpGs.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2269
tools:
- name: MethylCal
  role: R package providing Formatting() to restructure standard-control data and
    MethylCalCalibrationPlot to fit the four random-effects model specifications and
    return mlik, DIC and RSS
  repo: https://github.com/lb664/MethylCal
- name: INLA
  role: Bayesian inference engine used by MethylCalCalibrationPlot to fit the latent
    Gaussian random-effects models and compute DIC/mlik
  repo: http://www.r-inla.org/
- name: R
  role: Language in which the MethylCal package is written and run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz325
    title: 'MethylCal: Bayesian calibration of methylation levels'
  source_content_hash: sha256:1916f01c3a3ae527855bc6d4acc7a3487003ea956c0820ce335183a53b3ac8b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lb664/MethylCal
  tools:
  - INLA
  - MethylCal
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# methylation-standard-control-calibration

## Summary

A Bayesian calibration skill that fits MethylCal's four random-effects regression model specifications (M1-M4) jointly across all CpGs of a CGI/DMR assay using standard controls at known methylation percentages, and selects the best-fitting model per assay via the Deviance Information Criterion (DIC).

## When to use

Use when bisulfite amplicon sequencing standard controls with known methylation percentages (e.g. 0/25/50/75/100% BWS or 0/12.5/25/37.5/50/62.5/87.5/100% Celiac) are available and you must pick the best MethylCal model specification per assay.

## When NOT to use

- When the goal is to correct apparent methylation levels in case/control samples using an already-chosen calibration curve — use MethycalCorrection or MoskalevCorrection instead of re-running model selection.
- When standard controls with known methylation percentages are not available for the assay; the calibration curves are derived from these controls and cannot be estimated without them.
- When a one-at-a-time CpG calibration (e.g. Moskalev's CPR) is required; MethylCal deliberately analyses all CpGs within a CGI/DMR jointly rather than calibrating individual CpGs.

## Inputs

- Packaged BWS_data standard-control methylation dataset (Beckwith-Wiedemann syndrome, AMPs 0/25/50/75/100%, 15 controls and 18 cases)
- Packaged Celiac_data standard-control methylation dataset (eight NFkB-related genes, AMPs 0/12.5/25/37.5/50/62.5/87.5/100%)
- Assay definition: targeted CpG island (CGI) or differentially methylated region (DMR) and its CpG positions
- R environment with the MethylCal package and its dependencies lattice, latticeExtra and R-INLA (installed manually from inla.r-inla-download.org)

## Outputs

- Per-assay fitted MethylCal calibration models (M1: AMP effects, M2: crossed CpG effects, M3: RW1 latent Gaussian field over AMP, M4: random CpG intercepts/slopes)
- Goodness-of-fit measures per estimated model: mlik (marginal likelihood), DIC (Deviance Information Criterion) and RSS (Residual Sum of Squares)
- Selected calibration model per assay (lowest-DIC rule), e.g. M4 for KCNQ1OT1 and M3 for H19/IGF2
- Eight-row assay-to-selected-model mapping written to dic_model_selection.csv

## How to apply

First restructure the packaged standard-control data with the MethylCal Formatting() function at the actual AMP percentages (AMP = c(0, 25, 50, 75, 100) for BWS_data; eight AMPs for Celiac_data), so each CpG-AMP combination is in the format MethylCal expects. For each of the eight independent assays (two CGIs on the SDHC promoter and six imprinted DMRs), fit the four candidate random-effects specifications of equations 2-5 — AMP effects (M1), crossed CpG effects (M2), an RW1 latent Gaussian field over AMP (M3), and random CpG intercepts/slopes (M4) — jointly across all CpGs in the targeted CGI/DMR using MethylCalCalibrationPlot, which performs inference with INLA. Record the goodness-of-fit measures returned by each fitted model: mlik (marginal likelihood), DIC and RSS. Select, per assay, the model with the lowest DIC as the preferred calibration model (e.g. M4 for KCNQ1OT1 and M3 for H19/IGF2 in Ochoa et al. 2019). Before fitting, screen the standard-control data with ExploratoryPlot and remove visually identified problem CpGs or outliers (detected via the posterior predictive interval l = Q1 − 1.5IQR, u = Q3 + 1.5IQR), refitting models without them.

## Related tools

- **MethylCal** (R package providing Formatting() to restructure standard-control data and MethylCalCalibrationPlot to fit the four random-effects model specifications and return mlik, DIC and RSS) — https://github.com/lb664/MethylCal
- **INLA** (Bayesian inference engine used by MethylCalCalibrationPlot to fit the latent Gaussian random-effects models and compute DIC/mlik) — http://www.r-inla.org/
- **R** (Language in which the MethylCal package is written and run)

## Examples

```
data(BWS_data); AMP = c(0, 25, 50, 75, 100); data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)
```

## Evaluation signals

- For each assay, the lowest-DIC candidate reproduces the preferred model reported in Ochoa et al. (2019), e.g. M4 for KCNQ1OT1 and M3 for H19/IGF2.
- dic_model_selection.csv contains exactly eight assay rows, one per independent assay, each with a non-empty model assignment.
- Every candidate model fit returns all three goodness-of-fit measures (mlik, DIC, RSS) before the lowest-DIC rule is applied.
- Data restructuring used Formatting() with the correct actual AMP vector per dataset (5 AMPs for BWS_data; 8 AMPs for Celiac_data).
- Outlier screening used the posterior predictive interval l = Q1 − 1.5IQR and u = Q3 + 1.5IQR, with refits excluding detected outliers.

## Limitations

- No software version numbers are stated for MethylCal, INLA or R, so reproducibility depends on current package builds; R-INLA must be installed manually from its own repository.
- Manual visual inspection of the data may be needed before fitting — three CpGs (21-23) were removed from the PLAGL1 assay in the original study — and detected outliers require models to be refitted without them.
- Selection is by lowest DIC only; alternative criteria returned by the same fits (mlik, RSS) may in principle rank models differently and should be inspected.

## Evidence

- [methods] MethylCalCalibrationPlot implements the published models: "MethylCal as presented in Ochoa et. (2019) is implemented in the function `MethylCalCalibrationPlot`."
- [results] Calibration derives from standard controls with known methylation percentages: "we derived the calibration curves using five standard controls with known methylation percentages"
- [abstract] Eight assays and outlier detection motivate per-assay model choice: "We tested the proposed method on eight independent assays (two CpG islands and six imprinting DMRs) and demonstrated its benefits, including the ability to detect outliers"
- [results] Outlier threshold definition used before/while calibrating: "the posterior predictive interval [l, u] for outliers detection with l = Q1 − 1.5IQR and u = Q3 + 1.5IQR"
- [readme] Package supports visualising calibration curves from standard controls: "The package permits the visualisation of the calibration curve derived from standard controls with known methylation percentages for a selected assay."
- [readme] Formatting of standard-control data at actual AMP percentages: "AMP = c(0, 25, 50, 75, 100)
    data = Formatting(BWS_data, AMP = AMP, n_Control = 15, n_Case = 18)"
