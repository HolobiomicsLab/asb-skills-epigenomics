---
name: euler-ode-integration
description: Use when fitting ODE-based gene regulation models with sigmoidal activation
  and per-gene mRNA decay to smoothed time-course expression data, and a fast, OpenCL-friendly
  numerical integrator is needed.
when_to_use_negative:
- The target gene's expression does not change significantly over the whole time-course
  — the no-change model applies and the gene carries no regulatory information
- The gene can be fitted by the constant synthesis model (any regulator fits with
  w = 0 and large b), so it should be excluded as a target
- You need a complete description of regulatory interactions — time-series expression
  data cannot in principle provide this and Genexpi should be one of multiple sources
  of insight
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_2259
tools:
- name: Genexpi
  role: Provides the ODE model, simulated-annealing parameter fitting, and Euler integration
    on subsampled smoothed curves; available as command-line tool and R package
  repo: https://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: Language interface for loading expression series, B-spline smoothing, and
    driving the Genexpi ODE fitting core
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi's core ODE-based inference for interactive
    regulon discovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:74e5a1b9f5c332fc8df5ebeb38d461af21f57f546c0ba2b6e04e6b96c93f9051
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CyGenexpi
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# euler-ode-integration

## Summary

Integrates the Genexpi gene-regulation ODE dz/dt = k1*f(rho) - k2*z, with sigmoidal activation f(x) = 1/(1+e^-x) and regulatory input rho(t) = sum_j w_j*y_j(t) + b, using the computationally cheap Euler method on subsampled B-spline-smoothed expression curves to infer regulatory interactions from time-course data.

## When to use

Use when fitting ODE-based gene regulation models with sigmoidal activation and per-gene mRNA decay to smoothed time-course expression data, and a fast, OpenCL-friendly numerical integrator is needed.

## When NOT to use

- The target gene's expression does not change significantly over the whole time-course — the no-change model applies and the gene carries no regulatory information
- The gene can be fitted by the constant synthesis model (any regulator fits with w = 0 and large b), so it should be excluded as a target
- You need a complete description of regulatory interactions — time-series expression data cannot in principle provide this and Genexpi should be one of multiple sources of insight

## Inputs

- Time-course gene expression series (e.g. GSE6865 Bacillus subtilis SigA regulon data)
- Candidate regulator (e.g. SigA regulon) expression profiles for each target gene
- B-spline-smoothed and subsampled expression curves
- Error-margin parameters (e_minimal, e_absolute, e_relative) and fit-quality threshold (default 0.8)

## Outputs

- Converged fitted ODE parameters (k1, k2, b, regulator weights) per gene/regulator set (e.g. ode_fit_results.csv)
- Per-gene fit-quality score (proportion of time points within the error margin)
- Good-fit/bad-fit classification per gene-regulator pair

## How to apply

First smooth the measured expression profiles by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points, then subsample the smoothed curves so the ODE can be integrated accurately with the Euler method — a deliberately cheap scheme chosen because it is fast and easy to implement in OpenCL. The model synthesizes mRNA via a non-linear sigmoidal transformation of regulator expression, f(x) = 1/(1+e^-x), applied to the weighted regulatory input rho(t) = sum_j w_j*y_j(t) + b, and includes a per-gene mRNA decay rate k2 (dz/dt = k1*f(rho) - k2*z). Fit the parameter vector beta = (k1, k2, b, weights) by simulated annealing, executing 128 annealing runs with different initial parameter values per gene/regulator set. Apply the regularization term r(beta) penalizing k1 exceeding the maximal target expression level, regulator weights with max_t |w_j*y_j(t)| >= 10, and regulatory input never approaching the sigmoid midpoint (min_t |rho(t)| >= 0.5), with regularization constant c of approximately one tenth of the number of time points. Judge fit quality as the proportion of time points where the fitted profile lies within the error margin error(t) = max{e_minimal, e_absolute + z(t)*e_relative}; a fit is good if quality exceeds the default threshold of 0.8.

## Related tools

- **Genexpi** (Provides the ODE model, simulated-annealing parameter fitting, and Euler integration on subsampled smoothed curves; available as command-line tool and R package) — https://github.com/cas-bioinf/genexpi/wiki/
- **R** (Language interface for loading expression series, B-spline smoothing, and driving the Genexpi ODE fitting core)
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi's core ODE-based inference for interactive regulon discovery)

## Examples

```
Rscript -e 'library(genexpi); fit <- fitGeneModel(expression_series, regulon_profiles, method="euler")'
```

## Evaluation signals

- The 128 annealing runs converge to stable, highly replicable parameter values across different initial values
- Fitted profile lies within the error margin error(t) = max{e_minimal, e_absolute + z(t)*e_relative} for at least 80% of time points (default good-fit threshold 0.8)
- Regularization penalties are not triggered: k1 does not exceed the maximal target expression level, no regulator weight reaches max_t |w_j*y_j(t)| >= 10, and min_t |rho(t)| >= 0.5 so the sigmoid midpoint is approached
- Euler integration is performed on a subsampled version of the B-spline-smoothed curves (smoothing done with degrees of freedom ≈ half the number of measurement points)
- Constant-synthesis and no-change genes are excluded before fitting

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Genexpi was designed for bacterial regulons; eukaryotic use (e.g. the GDS38 Sacharomyces cell-cycle series) requires caution and is not fully validated
- Time-series expression data cannot in principle provide complete information about regulatory interactions; results are best used alongside other sources of insight
- No software version numbers are stated for Genexpi, R, RStudio, or Cytoscape, which may complicate reproduction

## Evidence

- [methods] The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL.: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [methods] Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values.: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile. A fit is considered good if fit quality is above a given threshold (the: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile. A fit is considered good if fit quality is above a given threshold (the"
- [abstract] Genexpi is also available as a standalone command line tool and an R package.: "Genexpi is also available as a standalone command line tool and an R package."
