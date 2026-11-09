---
name: time-course-expression-smoothing
description: Use when time-course expression data (e.g. GSE44415, GSE6865, GDS38)
  must be converted into smooth continuous curves before fitting an ODE-based gene
  regulatory model or running regulon recovery comparisons.
when_to_use_negative:
- Data are single-condition or static expression measurements with no time dimension
  — the B-spline time-series smoothing is designed for expression time series
- Downstream analysis requires the raw, unsmoothed measurement uncertainty, since
  smoothing masks uncertainty in the data via hard thresholds
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Genexpi
  role: R/OpenCL package that encourages smoothing the data prior to computation and
    consumes the smoothed curves for ODE parameter fitting via simulated annealing
  repo: https://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: Environment in which the B-spline linear regression smoothing and the evaluation
    notebooks (evaluation.Rmd) are executed
  repo: https://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:28860016ef6e1b6ed328f14934d7998a5c03f5678918db070041640545f586c4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARACNE
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# time-course-expression-smoothing

## Summary

Smoothes noisy time-course gene expression profiles by linear regression on a B-spline basis prior to Genexpi regulon inference, producing continuous curves that can be subsampled and integrated cheaply with the Euler method during ODE parameter fitting.

## When to use

Use when time-course expression data (e.g. GSE44415, GSE6865, GDS38) must be converted into smooth continuous curves before fitting an ODE-based gene regulatory model or running regulon recovery comparisons.

## When NOT to use

- Data are single-condition or static expression measurements with no time dimension — the B-spline time-series smoothing is designed for expression time series
- Downstream analysis requires the raw, unsmoothed measurement uncertainty, since smoothing masks uncertainty in the data via hard thresholds

## Inputs

- Time-course gene expression matrix (e.g. GEO series GSE44415 for Streptomyces coelicolor, GSE6865 for B. subtilis SigA, GDS38 for S. cerevisiae cell cycle)
- Number of measurement time points per profile (to set B-spline degrees of freedom)

## Outputs

- Smoothed expression profiles as continuous curves (subsampleable for Euler integration of the ODE model)
- Smoothed time-course data ready for Genexpi fitting and for comparative methods such as TD-Aracne

## How to apply

Fit each expression profile by linear regression on a B-spline basis, setting the degrees of freedom to approximately half the number of measurement points. Use the resulting smoothed curves as input to downstream Genexpi steps: subsample the fitted curves so the ODE model of eq. 1 can be integrated accurately with the computationally cheap Euler method in OpenCL. Apply the same smoothing uniformly across all methods and regulon variants being compared (e.g. TD-Aracne and Genexpi on the SigR regulon variants from GSE44415) so that differences in recovery are attributable to the inference method, not preprocessing. In the Genexpi workflow this smoothing follows candidate-network construction and precedes filtering of no-change and constant-synthesis genes.

## Related tools

- **Genexpi** (R/OpenCL package that encourages smoothing the data prior to computation and consumes the smoothed curves for ODE parameter fitting via simulated annealing) — https://github.com/cas-bioinf/genexpi/wiki/
- **R** (Environment in which the B-spline linear regression smoothing and the evaluation notebooks (evaluation.Rmd) are executed) — https://github.com/cas-bioinf/genexpi/wiki/

## Examples

```
Rscript evaluation.Rmd  # smooth each expression profile by linear regression on a B-spline basis with df ≈ n_timepoints / 2 before Genexpi ODE fitting
```

## Evaluation signals

- Degrees of freedom of the B-spline basis equals approximately half the number of measurement time points
- Smoothed curves support accurate Euler integration after subsampling, making evaluation of the error function fast in OpenCL
- The same smoothed data reproduce downstream findings, e.g. Genexpi recovering substantially more SigR targets from literature-derived than random regulons on GSE44415 while TD-Aracne's recovery is barely distinguishable
- Subsequent no-change and constant-synthesis model filters can be evaluated on the smoothed profiles as in the Genexpi workflow

## Limitations

- Smoothing masks uncertainty in the data and Genexpi relies on multiple hard thresholds, so fitted-profile quality depends on spline choices
- The ODE subsampling/Euler integration requires sufficiently dense, smooth curves; the recommended rule of thumb (df ≈ half the number of time points) was tuned on bacterial regulon data

## Evidence

- [methods] B-spline degrees of freedom rule: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Euler integration rationale: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [results] Workflow position of smoothing: "Import the time-course expression data and smooth them to provide a continuous curve."
- [results] Limitation: masked uncertainty: "Most notably, Genexpi masks uncertainty in the data and uses multiple hard thresholds."
- [results] R notebook reproducibility: "Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1."
