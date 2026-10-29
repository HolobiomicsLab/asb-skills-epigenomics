---
name: regularization-impact-comparison-analysis
description: Use when evaluating whether a penalty-based model-selection term (Genexpi's
  r(β)) improves parameter interpretability without degrading fit quality, by refitting
  the same genes/regulators with the penalty switched off and on.
when_to_use_negative:
- The goal is regulon discovery or fit-quality thresholding (e.g. fit quality > 0.8)
  rather than assessing the effect of the r(β) penalty itself
- Data are not time-course expression profiles smoothed onto a B-spline basis; regularization
  bounds like max_t |w_j*y_j(t)| < 10 are defined on smoothed profiles
- Working with the eukaryotic (Sacharomyces/GDS38) evaluation, whose scope and validation
  differ from the bacterial SigB setting
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: Genexpi
  role: R/OpenCL package used to smooth data, run simulated-annealing ODE fitting
    with and without the r(β) term, and integrate eq. 1 with Euler on subsampled curves
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: R
  role: Runtime for the evaluation.Rmd notebook (Additional file 1) that orchestrates
    smoothing, fitting, and export of regularization_comparison.csv
  repo: http://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:867bc3b3419b9f3cc9705ad85bc4199860aea2231aeb4aaaaab1f2c6c521f0bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# regularization-impact-comparison-analysis

## Summary

Compares Genexpi ODE model fits run with and without the regularization term r(β) on the SigB regulon of B. subtilis (Subtiwiki, January 2017) with GSE time-course expression, to verify that regularization constrains parameters into biologically interpretable ranges without materially increasing residual error.

## When to use

Use when evaluating whether a penalty-based model-selection term (Genexpi's r(β)) improves parameter interpretability without degrading fit quality, by refitting the same genes/regulators with the penalty switched off and on.

## When NOT to use

- The goal is regulon discovery or fit-quality thresholding (e.g. fit quality > 0.8) rather than assessing the effect of the r(β) penalty itself
- Data are not time-course expression profiles smoothed onto a B-spline basis; regularization bounds like max_t |w_j*y_j(t)| < 10 are defined on smoothed profiles
- Working with the eukaryotic (Sacharomyces/GDS38) evaluation, whose scope and validation differ from the bacterial SigB setting

## Inputs

- SigB regulon of B. subtilis from Subtiwiki (as of January 2017)
- Time-course gene expression data for B. subtilis (GSE series used in the SigB evaluation)
- Additional file 1 R notebook (evaluation.Rmd) with Genexpi fitting code
- Genexpi R/OpenCL implementation (github.com/cas-bioinf/genexpi)

## Outputs

- regularization_comparison.csv: per-gene residual errors and regularization penalties r(β) for both the unregularized (c = 0) and regularized fits
- Mean residual error across genes for the regularized and unregularized settings
- Per-gene r(β) values confirming which models fall within penalty bounds

## How to apply

Load the Additional file 1 R notebook (evaluation.Rmd) with the SigB regulon (Subtiwiki, January 2017) and smooth expression profiles by linear regression on a B-spline basis with degrees of freedom equal to approximately half the number of measurement points. Fit the ODE model for each SigB target gene without regularization (c = 0) using simulated annealing with 128 annealing runs per gene/regulator set, integrating eq. 1 with the Euler method on subsampled smoothed curves via the Genexpi R/OpenCL implementation. Repeat the identical procedure with regularization, setting c to about one tenth of the number of time points after smoothing and applying penalty bounds: k1 below the maximum expression level of the target, max_t |w_j*y_j(t)| < 10 for all regulators, and min_t |ρ(t)| < 0.5. Compute mean residual error across genes for both settings plus per-gene r(β), then assemble per-gene residual errors and penalties into regularization_comparison.csv.

## Related tools

- **Genexpi** (R/OpenCL package used to smooth data, run simulated-annealing ODE fitting with and without the r(β) term, and integrate eq. 1 with Euler on subsampled curves) — http://github.com/cas-bioinf/genexpi/wiki/
- **R** (Runtime for the evaluation.Rmd notebook (Additional file 1) that orchestrates smoothing, fitting, and export of regularization_comparison.csv) — http://github.com/cas-bioinf/genexpi/wiki/

## Examples

```
Rscript -e "rmarkdown::render('evaluation.Rmd', params=list(regularized=TRUE))" && Rscript -e "rmarkdown::render('evaluation.Rmd', params=list(regularized=FALSE))"
```

## Evaluation signals

- Mean residual error with regularization differs from the unregularized fit by less than one part in one hundred
- r(β) = 0 for most regularized models, indicating fitted parameters fall inside the penalty bounds (k1 < max target expression, max_t |w_j*y_j(t)| < 10, min_t |ρ(t)| < 0.5)
- Both settings use identical fitting configuration: 128 annealing runs per gene/regulator set, Euler integration on subsampled smoothed curves
- regularization_comparison.csv contains per-gene residual errors and r(β) for both settings in a single table
- Smoothing used B-spline regression with degrees of freedom ≈ half the number of measurement points, and c ≈ one tenth of the number of time points after smoothing

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds, so results are incomplete on their own and best used as one of multiple sources of insight
- No software version numbers are stated for Genexpi or R, so exact reproducibility of the comparison may depend on the code snapshot in Additional file 1
- The comparison is grounded in the bacterial SigB regulon; the article notes Genexpi was designed for bacterial regulons and should be applied to eukaryotic data only with caution

## Evidence

- [methods] To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β).: "To discriminate between those models and to force the parameters into biologically interpretable ranges, we introduce the regularization term r(β)."
- [methods] Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values.: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL.: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [results] the SigB regulon of B. subtilis from Subtiwiki [14] as of January 2017: "the SigB regulon of B. subtilis from Subtiwiki [14] as of January 2017"
- [intro] Binaries, source code and documentation are available at http://github.com/cas-bioinf/genexpi/wiki/.: "Binaries, source code and documentation are available at http://github.com/cas-bioinf/genexpi/wiki/."
