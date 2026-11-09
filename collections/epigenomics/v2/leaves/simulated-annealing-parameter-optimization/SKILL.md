---
name: simulated-annealing-parameter-optimization
description: Use when fitting non-linear ODE regulation models to smoothed time-course
  expression data and parameter values (synthesis rate, decay rate, regulator weights)
  must be estimated by global optimization over a regularized objective.
when_to_use_negative:
- The data are static (single-condition) expression measurements rather than a time
  course, since the ODE integration and annealing objective require temporal profiles.
- The analysis targets genes whose expression does not change significantly over the
  whole time range — these are excluded as both regulators and targets because the
  Genexpi model contains no information for them.
- A differentiable, convex objective is available where gradient-based optimization
  would be simpler and more reliable than stochastic global search.
edam_operation: http://edamontology.org/operation_3893
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: Genexpi
  role: Provides the simulated-annealing parameter fitting core (128 runs per gene/regulator
    set) executed via OpenCL
  repo: https://github.com/cas-bioinf/genexpi
- name: R
  role: R package interface for running the Genexpi fitting and evaluation notebooks
    (e.g. evaluation.Rmd)
  repo: https://github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:2168fa39e855a4f22e95d523f1af0000d8f42c6fe9301da23cf6f4609de5af00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cas-bioinf/genexpi
  tools:
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_3893
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# simulated-annealing-parameter-optimization

## Summary

Optimizes parameters of Genexpi's ODE gene regulation model (k1, k2, b, regulator weights) by minimizing a regularized error function via simulated annealing, with 128 annealing runs per gene–regulator set executed in OpenCL.

## When to use

Use when fitting non-linear ODE regulation models to smoothed time-course expression data and parameter values (synthesis rate, decay rate, regulator weights) must be estimated by global optimization over a regularized objective.

## When NOT to use

- The data are static (single-condition) expression measurements rather than a time course, since the ODE integration and annealing objective require temporal profiles.
- The analysis targets genes whose expression does not change significantly over the whole time range — these are excluded as both regulators and targets because the Genexpi model contains no information for them.
- A differentiable, convex objective is available where gradient-based optimization would be simpler and more reliable than stochastic global search.

## Inputs

- Smoothed time-course gene expression profiles (B-spline regression with ~half the measurement points as degrees of freedom)
- Candidate regulator expression profiles (e.g. SigA/SigB from the GSE6865 B. subtilis dataset)
- Regularization constant c (~1/10 of number of smoothed time points) and user-set error-margin parameters (e_minimal, e_absolute, e_relative)

## Outputs

- Converged ODE parameter sets (k1, k2, b, regulator weights w_j) per gene–regulator set
- Fit-quality scores per gene (proportion of time points within the error margin)
- Gene/regulator exclusion flags from the no-change and constant-synthesis model fits

## How to apply

Smooth the measured expression profiles first by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points, and subsample the curves so the ODE dz/dt = k1*f(rho) - k2*z can be integrated with the cheap Euler method. For each gene and candidate regulator set, minimize the regularized objective r(beta) by simulated annealing, executing 128 annealing runs with different initial parameter values via the OpenCL core. The regularization term forces biologically interpretable parameters: penalize k1 exceeding the maximal target expression level, regulator weights with max_t |w_j*y_j(t)| >= 10, and regulatory input never approaching the sigmoid midpoint (min_t |rho(t)| >= 0.5); set the regularization constant c to roughly one tenth of the number of smoothed time points. The same annealing machinery is also used to fit the no-change and constant-synthesis models, whose genes are then excluded from further analysis. Score each converged fit by fit quality — the proportion of time points where the fitted profile lies within the error margin error(t) = max{e_minimal, e_absolute + z(t)*e_relative} — and accept the fit only if quality exceeds the default threshold of 0.8.

## Related tools

- **Genexpi** (Provides the simulated-annealing parameter fitting core (128 runs per gene/regulator set) executed via OpenCL) — https://github.com/cas-bioinf/genexpi
- **R** (R package interface for running the Genexpi fitting and evaluation notebooks (e.g. evaluation.Rmd)) — https://github.com/cas-bioinf/genexpi

## Examples

```
Rscript -e "genexpi::fit_parameters(expression_data, regulators, n_annealing_runs=128, regularization_c=1)"
```

## Evaluation signals

- All 128 annealing runs per gene–regulator set converge to stable, replicable parameter values (k1, k2, b, w_j).
- Fitted profile lies within the error margin for at least 80% of time points (default good-fit threshold 0.8).
- Regularization bounds hold: max_t |w_j*y_j(t)| < 10, min_t |rho(t)| >= 0.5, and k1 below the maximal target expression level.
- Genes best explained by the no-change or constant-synthesis models are correctly excluded from the target set.
- On the SigB regulon benchmark (GSE6865, 4-DF spline), the recovered literature regulon fraction is ~90% with roughly one third of random regulator profiles falsely accepted.

## Limitations

- Stochastic annealing with hard regularization thresholds may mask uncertainty in the data; results should be used alongside other sources of insight.
- Time-series expression data cannot in principle provide complete information about regulatory interactions.
- No software version numbers are stated for Genexpi or R in the article, complicating exact reproduction.

## Evidence

- [methods] Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values.: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β).: "To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β)."
- [methods] Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile.: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile."
- [methods] Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [results] Genexpi masks uncertainty in the data and uses multiple hard thresholds.: "Most notably, Genexpi masks uncertainty in the data and uses multiple hard thresholds."
