---
name: simulated-annealing-model-regularization
description: Use when fitting dynamical ODE parameters from smoothed time-course expression
  data and you need unregularized vs regularized (r(β), c ≈ 0.1 × number of smoothed
  time points) fits compared for model discrimination.
when_to_use_negative:
- Genes whose expression does not change significantly over the whole time range —
  Genexpi excludes these as both regulators and targets since the model contains no
  information in that case
- Genes explainable by the constant synthesis model — they can be fitted by any regulator
  via w = 0 and large b, and are excluded as targets
- Data with single-point (non time-course) measurements — smoothing on a B-spline
  basis and ODE integration require a time series
edam_operation: http://edamontology.org/operation_2429
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Genexpi
  role: R/OpenCL implementation used to run simulated annealing fits (128 runs per
    gene/regulator set) and Euler integration of the ODE model
  repo: https://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: Interface for loading the Additional file 1 evaluation.Rmd notebook, smoothing
    expression profiles on a B-spline basis, and assembling regularization_comparison.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:565922569a93d2ec543f5db67df1e1c939c32ee01cde2de6a427715a4d7818ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_2429
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Simulated annealing model regularization

## Summary

Fits Genexpi ODE model parameters for gene-regulator pairs (e.g. the SigB regulon) via simulated annealing with 128 annealing runs per gene, and applies the regularization term r(β) to discriminate between candidate models and force parameters into biologically interpretable ranges.

## When to use

Use when fitting dynamical ODE parameters from smoothed time-course expression data and you need unregularized vs regularized (r(β), c ≈ 0.1 × number of smoothed time points) fits compared for model discrimination.

## When NOT to use

- Genes whose expression does not change significantly over the whole time range — Genexpi excludes these as both regulators and targets since the model contains no information in that case
- Genes explainable by the constant synthesis model — they can be fitted by any regulator via w = 0 and large b, and are excluded as targets
- Data with single-point (non time-course) measurements — smoothing on a B-spline basis and ODE integration require a time series

## Inputs

- Time-course gene expression dataset (e.g. the SigB regulon time series from the evaluation.Rmd Additional file 1 notebook)
- Regulon candidate gene-regulator pairs (e.g. SigB regulon of B. subtilis from Subtiwiki, January 2017)
- B-spline-smoothed expression curves (linear regression, df ≈ half the number of measurement points)

## Outputs

- Per-gene residual errors for unregularized (c = 0) and regularized fits
- Per-gene r(β) regularization penalty values
- regularization_comparison.csv combining residual errors and regularization penalties for both settings

## How to apply

Smooth expression profiles by linear regression on a B-spline basis with degrees of freedom ≈ half the number of measurement points, then minimize the error function (eq. 2) by simulated annealing, executing 128 annealing runs with different initial parameter values per gene and candidate regulator set. Integrate the ODE model (eq. 1) with the cheap Euler method on subsampled smoothed curves via the Genexpi R/OpenCL implementation. First fit with no regularization (c = 0), then repeat identically with the regularization term r(β), setting c ≈ one tenth of the number of time points after smoothing and enforcing penalty bounds: k1 < max expression level of the target, max_t |w_j*y_j(t)| < 10 for all regulators, and min_t |ρ(t)| < 0.5. Compare mean residual error across genes between the two settings; regularization should leave residual error nearly unchanged (within one part in a hundred) while driving r(β) = 0 for most regularized models, demonstrating that the penalty constrains parameters without degrading fit quality.

## Related tools

- **Genexpi** (R/OpenCL implementation used to run simulated annealing fits (128 runs per gene/regulator set) and Euler integration of the ODE model) — https://github.com/cas-bioinf/genexpi/wiki/
- **R** (Interface for loading the Additional file 1 evaluation.Rmd notebook, smoothing expression profiles on a B-spline basis, and assembling regularization_comparison.csv)

## Examples

```
Rscript -e "rmarkdown::render('evaluation.Rmd')"
```

## Evaluation signals

- Mean residual error across genes with regularization differs by less than one part in a hundred from the unregularized fit
- r(β) = 0 for most models fitted with the regularization term active
- Each gene/regulator set was fitted with exactly 128 annealing runs with different initial parameter values
- Regularized parameters satisfy the penalty bounds: k1 < max target expression level, max_t |w_j*y_j(t)| < 10, min_t |ρ(t)| < 0.5
- regularization_comparison.csv contains per-gene residual errors and penalties for both the c = 0 and regularized settings

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Time-series expression data cannot in principle provide complete information about regulatory interactions; results are best used as one of multiple sources of insight
- Genexpi was designed for bacterial regulons; applying it to eukaryotic data requires caution and is not fully validated

## Evidence

- [methods] simulated annealing runs: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] regularization term: "To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β)."
- [methods] Euler integration: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [results] SigB dataset: "the SigB regulon of B. subtilis from Subtiwiki [14] as of January 2017"
- [results] reproducibility notebook: "Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1."
- [readme] project documentation: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
