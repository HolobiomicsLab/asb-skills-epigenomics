---
name: regularized-maximum-a-posteriori-estimation
description: Use when you have smoothed time-course expression data for a target gene
  and candidate regulators and need to estimate ODE synthesis/decay/weight parameters
  while forcing them into biologically interpretable ranges.
when_to_use_negative:
- The gene's expression does not change significantly over the whole time-course (no
  change model) — Genexpi excludes these as both regulators and targets
- The gene is well fit by the constant synthesis model (could be fitted by any regulator
  with w = 0 and large b), so it is excluded as a target
- Data are not time-course measurements; steady-state expression cannot support the
  ODE-based fitting approach
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: Core package that runs the simulated-annealing parameter fitting with regularization,
    available as command-line tool and R package
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: Interface and scripting environment for Genexpi's core functionality; R notebooks
    (e.g. evaluation_sacharomyces.Rmd) reproduce the fitting and evaluation results
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi's fitting functionality for regulon discovery
    workflows
  repo: http://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:bd24ff09d3e5d4f57040d2d3a691e5b9e2a79c6b7019406ce0c131b87b1fbb39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CyGenexpi
  - Genexpi
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# regularized-maximum-a-posteriori-estimation

## Summary

Fits Genexpi's ODE gene-regulation model (dz/dt = k1*f(rho) - k2*z with sigmoidal activation) to smoothed time-course expression by minimizing a penalized objective via simulated annealing, yielding biologically interpretable MAP-like parameter estimates (k1, k2, b, regulator weights).

## When to use

Use when you have smoothed time-course expression data for a target gene and candidate regulators and need to estimate ODE synthesis/decay/weight parameters while forcing them into biologically interpretable ranges.

## When NOT to use

- The gene's expression does not change significantly over the whole time-course (no change model) — Genexpi excludes these as both regulators and targets
- The gene is well fit by the constant synthesis model (could be fitted by any regulator with w = 0 and large b), so it is excluded as a target
- Data are not time-course measurements; steady-state expression cannot support the ODE-based fitting approach

## Inputs

- Smoothed time-course gene expression profiles (e.g. B-spline-smoothed GSE6865 Bacillus subtilis SigA regulon series)
- Candidate regulator set with regulator expression profiles (e.g. ChIP- or literature-derived SigA/SigB/SigR regulons)
- User-specified error-margin parameters (e_minimal, e_absolute, e_relative) and regularization constant c

## Outputs

- Converged fitted parameters (k1, k2, b, regulator weights) per gene/regulator set
- Fit-quality score (proportion of time points within the error margin)
- Fitted expression profile for comparison against the measured profile

## How to apply

Smooth the measured expression profiles by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points, then subsample and integrate dz/dt = k1*f(rho) - k2*z (logistic f(x) = 1/(1+e^-x), rho(t) = sum_j w_j*y_j(t) + b) with the Euler method. Minimize the objective by simulated annealing, running 128 annealing runs with different initial parameter values per gene and candidate regulator set. Add the regularization term r(beta) that penalizes k1 exceeding the maximal target expression level, regulator weights with max_t |w_j*y_j(t)| >= 10, and regulatory input never approaching the sigmoid midpoint (min_t |rho(t)| >= 0.5); set the regularization constant c to approximately one tenth of the number of time points. Judge convergence by replicability across the 128 annealing runs, and score each fit as the proportion of time points within error(t) = max{e_minimal, e_absolute + z(t)*e_relative}, treating fits above the default 0.8 quality threshold as good.

## Related tools

- **Genexpi** (Core package that runs the simulated-annealing parameter fitting with regularization, available as command-line tool and R package) — http://github.com/cas-bioinf/genexpi/wiki/
- **R** (Interface and scripting environment for Genexpi's core functionality; R notebooks (e.g. evaluation_sacharomyces.Rmd) reproduce the fitting and evaluation results) — http://github.com/cas-bioinf/genexpi/wiki/
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi's fitting functionality for regulon discovery workflows) — http://github.com/cas-bioinf/genexpi/wiki/

## Evaluation signals

- The 128 annealing runs converge to stable parameter values with high replicability across runs with different initial parameter values
- Fitted profile lies within error(t) = max{e_minimal, e_absolute + z(t)*e_relative} for at least 80% of time points (default quality threshold 0.8)
- Regularization constraints hold: k1 does not exceed the maximal target expression level; for every regulator max_t |w_j*y_j(t)| < 10; min_t |rho(t)| >= 0.5 so the sigmoid operates near its midpoint region
- Genes flagged by the no-change and constant-synthesis pre-filters are excluded from fitting as targets and regulators

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Time-series expression data cannot in principle provide complete information about regulatory interactions; results are best used as one of multiple sources of insight
- Genexpi was designed for bacterial regulons; applying it to eukaryotic data requires caution and is not fully validated

## Evidence

- [methods] Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values.: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β).: "To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β)."
- [methods] Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile.: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile."
- [abstract] Genexpi is also available as a standalone command line tool and an R package.: "Genexpi is also available as a standalone command line tool and an R package."
- [readme] Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
