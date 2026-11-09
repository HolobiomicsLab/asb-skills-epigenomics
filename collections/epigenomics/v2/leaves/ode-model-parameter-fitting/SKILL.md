---
name: ode-model-parameter-fitting
description: Use when inferring regulatory interactions from time-course gene expression
  data and per-gene ODE parameters (synthesis rate, decay rate, regulator weights)
  must be fitted to a sigmoidal activation model, e.g. Bacillus subtilis SigA regulon
  with GSE6865 series.
when_to_use_negative:
- Genes whose expression does not change significantly over the whole time range —
  the no-change model excludes them as the Genexpi model contains no information for
  them
- Genes fitted adequately by the constant synthesis model (w = 0, large b) — they
  must be excluded as targets since any regulator could fit them
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: Genexpi
  role: Core package providing the ODE model, simulated-annealing fitting (OpenCL),
    regularization, and R/command-line interfaces
  repo: https://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: R package/interface used to smooth expression profiles on a B-spline basis
    and run the fitting workflow
  repo: https://github.com/cas-bioinf/genexpi/wiki/
- name: CyGenexpi
  role: Cytoscape plugin wrapping Genexpi's fitting functionality for regulon discovery
  repo: https://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:db9f9a48707f33aed0de0015025fc606db3df30c78add7b3238b7f7b627a15e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CyGenexpi
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ode-model-parameter-fitting

## Summary

Fit Genexpi's ODE regulation model — mRNA synthesis dz/dt = k1*f(rho) - k2*z with a logistic (sigmoidal) activation of regulator inputs and a per-gene decay rate — to smoothed time-course expression data, yielding per-gene/regulator parameters (k1, k2, b, weights) and a fit-quality score.

## When to use

Use when inferring regulatory interactions from time-course gene expression data and per-gene ODE parameters (synthesis rate, decay rate, regulator weights) must be fitted to a sigmoidal activation model, e.g. Bacillus subtilis SigA regulon with GSE6865 series.

## When NOT to use

- Genes whose expression does not change significantly over the whole time range — the no-change model excludes them as the Genexpi model contains no information for them
- Genes fitted adequately by the constant synthesis model (w = 0, large b) — they must be excluded as targets since any regulator could fit them

## Inputs

- Time-course gene expression series (e.g. GSE6865 for the Bacillus subtilis SigA regulon)
- Candidate regulator profiles (e.g. SigA regulon membership from Subtiwiki or ChIP experiments)
- User-specified error-margin parameters (e_minimal, e_absolute, e_relative) and fit-quality threshold (default 0.8)

## Outputs

- Converged fitted ODE parameters per gene/regulator set (k1, k2, b, regulator weights)
- Fit-quality score per gene (proportion of time points within the error margin)
- Parameter table (e.g. ode_fit_results.csv)

## How to apply

Smooth measured expression profiles by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points, then subsample the smoothed curves so the ODE dz/dt = k1*f(rho) - k2*z, with logistic activation f(x) = 1/(1+e^-x) and regulatory input rho(t) = sum_j w_j*y_j(t) + b, can be integrated accurately with the cheap Euler method. Minimize the error by simulated annealing (OpenCL in Genexpi), running 128 annealing runs with different initial parameter values per gene and candidate regulator set to ensure replicable convergence. Add the regularization term r(beta) with constant c of approximately one tenth of the number of time points, penalizing k1 exceeding the maximal target expression level, regulator weights with max_t |w_j*y_j(t)| >= 10, and regulatory input never approaching the sigmoid midpoint (min_t |rho(t)| >= 0.5). Judge fit quality as the proportion of time points where the fitted profile lies within the error margin error(t) = max{e_minimal, e_absolute + z(t)*e_relative}, treating the fit as good above the default threshold of 0.8, and write converged parameters plus the quality score to output (e.g. ode_fit_results.csv).

## Related tools

- **Genexpi** (Core package providing the ODE model, simulated-annealing fitting (OpenCL), regularization, and R/command-line interfaces) — https://github.com/cas-bioinf/genexpi/wiki/
- **R** (R package/interface used to smooth expression profiles on a B-spline basis and run the fitting workflow) — https://github.com/cas-bioinf/genexpi/wiki/
- **CyGenexpi** (Cytoscape plugin wrapping Genexpi's fitting functionality for regulon discovery) — https://github.com/cas-bioinf/genexpi/wiki/

## Examples

```
Rscript genexpi_fit.R --expression GSE6865.csv --regulon sigA --runs 128 --threshold 0.8 --output ode_fit_results.csv
```

## Evaluation signals

- All 128 annealing runs per gene/regulator set converge to stable, replicable parameter values
- Fit quality exceeds the default threshold of 0.8 (fitted profile within the error margin for at least 80% of time points)
- Regularization satisfied: k1 does not exceed the maximal target expression level; no regulator weight has max_t |w_j*y_j(t)| >= 10; regulatory input approaches the sigmoid midpoint (min_t |rho(t)| < 0.5 avoided)
- B-spline smoothing used degrees of freedom equal to approximately half the number of measurement points
- Genes failing the no-change and constant-synthesis pre-models were excluded before fitting

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Euler integration on subsampled smoothed curves is computationally cheap but relies on sufficient subsampling for accuracy
- Eukaryotic application requires caution; the method was designed and validated on bacterial regulons

## Evidence

- [methods] ODE model definition and Euler integration: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [methods] Smoothing via B-splines: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [methods] Simulated annealing with 128 runs: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Fit-quality threshold: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile. A fit is considered good if fit quality is above a given threshold (the"
- [readme] Repository availability: "Gene Network Inferece from Expression Time Series."
