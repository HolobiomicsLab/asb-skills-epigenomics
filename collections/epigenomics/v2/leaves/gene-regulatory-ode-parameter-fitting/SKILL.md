---
name: gene-regulatory-ode-parameter-fitting
description: Use when fitting gene-regulator ODE parameters to smoothed time-course
  expression profiles (e.g. SigB regulon targets) and you need to compare regularized
  vs. unregularized fits or enforce biologically interpretable parameter bounds.
when_to_use_negative:
- Genes whose expression does not change significantly over the whole time range (no
  change model) — Genexpi contains no information in that case and such genes are
  excluded as regulators and targets
- Genes that could be fitted by the constant synthesis model (any regulator fits with
  w = 0 and large b) — excluded as targets
- Eukaryotic time-course data where full validation is lacking — the article states
  Genexpi was designed for bacterial regulons and can be employed for eukaryotic data
  only with caution
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: Genexpi
  role: R/OpenCL package that performs simulated-annealing ODE parameter fitting and
    applies the regularization term r(β)
  repo: github.com/cas-bioinf/genexpi
- name: R
  role: Interface for running the evaluation.Rmd notebook that reproduces the regularized
    vs. unregularized fitting comparison
  repo: github.com/cas-bioinf/genexpi
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:19b05f35bd39d78de42ba6a7e86e9129cbe0e1dd8da75bcc7599de549ea3df41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genexpi
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-regulatory-ode-parameter-fitting

## Summary

Fits ODE model parameters (b, k1, w, ρ) for gene-regulator pairs to smoothed time-course expression data via simulated annealing in Genexpi, with a regularization term r(β) that discriminates candidate models and forces parameters into biologically interpretable ranges.

## When to use

Use when fitting gene-regulator ODE parameters to smoothed time-course expression profiles (e.g. SigB regulon targets) and you need to compare regularized vs. unregularized fits or enforce biologically interpretable parameter bounds.

## When NOT to use

- Genes whose expression does not change significantly over the whole time range (no change model) — Genexpi contains no information in that case and such genes are excluded as regulators and targets
- Genes that could be fitted by the constant synthesis model (any regulator fits with w = 0 and large b) — excluded as targets
- Eukaryotic time-course data where full validation is lacking — the article states Genexpi was designed for bacterial regulons and can be employed for eukaryotic data only with caution

## Inputs

- Time-course gene expression series (e.g. GSE6865 for SigA, GSE44415 for SigR, GDS38 for yeast)
- Candidate regulon definition (e.g. SigB regulon of B. subtilis from Subtiwiki, January 2017)
- Additional file 1 R notebook (evaluation.Rmd) with Genexpi R/OpenCL code
- B-spline smoothing settings (degrees of freedom ≈ half the number of measurement points)

## Outputs

- Per-gene fitted ODE parameter sets (b, k1, w, ρ) for regularized and unregularized settings
- Mean residual error across genes for both fitting settings
- Per-gene regularization penalty values r(β)
- regularization_comparison.csv (table of per-gene residual errors and penalties for both settings)

## How to apply

Smooth expression profiles by linear regression on a B-spline basis with degrees of freedom approximately half the number of measurement points, then subsample the smoothed curves so the ODE (eq. 1) can be integrated accurately with the cheap Euler method in the Genexpi OpenCL implementation. For each gene and candidate regulator set, minimize the error function (eq. 2) by simulated annealing using 128 annealing runs with different initial parameter values — run this both without regularization (c = 0) and with the regularization term r(β), setting c to roughly one tenth of the number of time points after smoothing. Apply the penalty bounds k1 < max expression level of the target, max_t |w_j*y_j(t)| < 10 for all regulators, and min_t |ρ(t)| < 0.5. Judge success by fit quality (proportion of time points where the fitted profile falls within the error margin of the measured profile, threshold 0.8 by default) and by confirming that regularization does not materially worsen residuals while driving r(β) to zero for most models.

## Related tools

- **Genexpi** (R/OpenCL package that performs simulated-annealing ODE parameter fitting and applies the regularization term r(β)) — github.com/cas-bioinf/genexpi
- **R** (Interface for running the evaluation.Rmd notebook that reproduces the regularized vs. unregularized fitting comparison) — github.com/cas-bioinf/genexpi

## Examples

```
Rscript evaluation.Rmd
```

## Evaluation signals

- Mean residual error with regularization differs by less than one part in a hundred from the unregularized fit
- r(β) = 0 for most regularized models, indicating penalty bounds are satisfied without residual cost
- Fitted parameters fall within the penalty bounds: k1 < max target expression level, max_t |w_j*y_j(t)| < 10 for all regulators, min_t |ρ(t)| < 0.5
- Exactly 128 annealing runs executed per gene/regulator set with different initial parameter values
- Good fits show fit quality (proportion of time points within the error margin) above the default threshold of 0.8

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds (e.g. the 0.8 fit-quality cutoff)
- Time-series expression data cannot in principle provide complete information about regulatory interactions; Genexpi is best used as one of multiple sources of insight
- Fitting is computationally heavy for large regulons; the yeast evaluation reduced burden by randomly selecting 30 targets per transcription factor after excluding constant and constant synthesis targets

## Evidence

- [methods] To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β).: "To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β)."
- [methods] Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values.: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL.: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [methods] Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom"
- [readme] Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
