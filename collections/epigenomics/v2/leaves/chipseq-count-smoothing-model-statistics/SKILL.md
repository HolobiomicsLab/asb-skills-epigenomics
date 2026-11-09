---
name: chipseq-count-smoothing-model-statistics
description: Use when standard errors or position-wise significance are needed for
  smooth (spline) terms fitted to ChIP-Seq count data with a sparse penalized Hessian,
  and forming the full matrix inverse is infeasible.
when_to_use_negative:
- The model is small enough that the full Hessian can be inverted densely with no
  memory concern — direct inversion is simpler
- The analysis target is differential occupancy testing between conditions rather
  than pointwise uncertainty of the smooth fit itself
- The fit was produced by a non-sparse or non-penalized smoother (e.g. plain mgcv
  without sparsity exploitation), where the sparse inverse subset on NZ(H) does not
  apply as described
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: Provides the tiled GAM fit, the penalized sparse Hessian, and the se(result)/computeSignificance()
    accessors whose values the sparse inverse subset must reproduce
  repo: https://github.com/gstricker/fastGenoGAM
- name: sparseinv
  role: R wrapper of SuiteSparse that evaluates only the elements of the inverse Hessian
    within NZ(H), supplying the variances for standard errors
- name: R
  role: Execution environment; GenoGAM 2.0 is implemented as an open source R package
    building on Bioconductor infrastructure
- name: mgcv
  role: General-purpose R library underlying the GAM fitting framework on which GenoGAM
    is built
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:db1bf9dd154057d209d0ae37b949a1057ace048f64bbe9733692cea53b3d1d14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - R
  - mgcv
  - sparseinv
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chipseq-count-smoothing-model-statistics

## Summary

Computes pointwise standard errors (and downstream log p-values) for smooth fits in GenoGAM-style genome-wide GAMs by extracting only the needed elements of the inverse penalized Hessian via the sparse inverse subset algorithm (sparseinv/SuiteSparse), avoiding explicit inversion of the full sparse Hessian.

## When to use

Use when standard errors or position-wise significance are needed for smooth (spline) terms fitted to ChIP-Seq count data with a sparse penalized Hessian, and forming the full matrix inverse is infeasible.

## When NOT to use

- The model is small enough that the full Hessian can be inverted densely with no memory concern — direct inversion is simpler
- The analysis target is differential occupancy testing between conditions rather than pointwise uncertainty of the smooth fit itself
- The fit was produced by a non-sparse or non-penalized smoother (e.g. plain mgcv without sparsity exploitation), where the sparse inverse subset on NZ(H) does not apply as described

## Inputs

- Fitted GenoGAM tile object produced by genogam() (e.g. genogam(ggd, lambda = 4601, theta = 4.51))
- Penalized Hessian H of the tile fit with its nonzero sparsity pattern NZ(H)
- R session with fastGenoGAM/GenoGAM 2.0 and the sparseinv package (SuiteSparse)

## Outputs

- Pointwise variance vector sigma^2 (diagonal of the selected elements of H^-1)
- Per-tile standard-error vector (square roots of variances), storable as tile_standard_errors.rds
- Position-wise log p-values via computeSignificance(result)

## How to apply

Fit the tiled model with genogam() (e.g. with fixed hyperparameters lambda = 4601 and theta = 4.51) and assemble the penalized Hessian H of a tile fit, identifying its nonzero sparsity pattern NZ(H). Compute the sparse Cholesky factorization of H, then call the sparseinv R package — a wrapper of SuiteSparse code — to evaluate only those elements of H^-1 that lie within NZ(H). Extract the diagonal entries to form the pointwise variance vector sigma^2 and take square roots to obtain standard errors, since GenoGAM 2.0's speed comes from exploiting the sparsity of the model rather than dense matrix operations. Cross-check the computed standard-error vector against the package's se(result) accessor, which returns per-tile standard errors. These standard errors feed position-wise log p-values via computeSignificance() for base-level significance testing.

## Related tools

- **GenoGAM 2.0** (Provides the tiled GAM fit, the penalized sparse Hessian, and the se(result)/computeSignificance() accessors whose values the sparse inverse subset must reproduce) — https://github.com/gstricker/fastGenoGAM
- **sparseinv** (R wrapper of SuiteSparse that evaluates only the elements of the inverse Hessian within NZ(H), supplying the variances for standard errors)
- **R** (Execution environment; GenoGAM 2.0 is implemented as an open source R package building on Bioconductor infrastructure)
- **mgcv** (General-purpose R library underlying the GAM fitting framework on which GenoGAM is built)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result); computeSignificance(result)
```

## Evaluation signals

- All computed standard errors are finite and non-negative
- The computed standard-error vector matches se(result) on the same tile fit
- Standard errors are only computed for tiles' nonzero Hessian elements; diagonal extraction yields exactly one variance per fitted smooth coefficient
- Resulting log p-values from computeSignificance(result) are available position-wise via pval(result)

## Limitations

- fastGenoGAM is deprecated; the maintained version is the Bioconductor GenoGAM release / gstricker/GenoGAM
- Requires R 3.4.1 or higher and correctly configured Armadillo; large-matrix errors require enabling ARMA_64BIT_WORD in RcppArmadillo
- Fixed hyperparameters (lambda = 4601, theta = 4.51) in the vignette skip parameter estimation to save time; data-driven cross-validated smoothing is the default approach
- Performance claims reference generic hardware ('a typical lab server') without exact specifications; no Zenodo archival record, GPL-2 license

## Evidence

- [other] GenoGAM 2.0 computes standard errors with the sparse inverse subset algorithm, which yields the elements of the inverse Hessian needed for the variance; the implementation is based on the R package sparseinv, a wrapper of SuiteSparse code.: [evidence span withheld — anchor status: not_found]
- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] compute position-wise log p-values with `computeSignificance()`: "compute position-wise log p-values with `computeSignificance()`"
- [readme] ## Check the fit and standard error
fits(result)
se(result): "## Check the fit and standard error
fits(result)
se(result)"
