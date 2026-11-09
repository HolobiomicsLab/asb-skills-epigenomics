---
name: genomic-tile-standard-error-computation
description: Use when you need pointwise standard errors or variance of GAM smooth
  estimates for base-level significance testing in GenoGAM tile fits, where the penalized
  Hessian is large and sparse so a full inverse is infeasible.
when_to_use_negative:
- When the smoothing hyperparameters lambda and theta are unknown and must be estimated
  — the vignette workflow fixes lambda = 4601 and theta = 4.51 to skip parameter estimation,
  and this skill presumes a fitted tile
- When the Hessian is small or dense enough that a full inverse is affordable — the
  sparse inverse subset machinery is unnecessary overhead there
- When the goal is fitting the model itself with SuperLU rather than post-fit uncertainty
  quantification
edam_operation: http://edamontology.org/operation_2239
tools:
- name: GenoGAM 2.0
  role: Package whose tile fits and se(result)/fits(result) accessors supply and validate
    the standard errors
  repo: https://github.com/gstricker/fastGenoGAM
- name: R
  role: Language in which GenoGAM 2.0 is implemented and in which the sparse inverse
    subset computation is run
- name: sparseinv
  role: R package wrapping SuiteSparse code to compute only the required subset of
    inverse-Hessian elements
- name: SuperLU
  role: Direct sparse solver used for parameter fitting whose sparse Hessian structure
    feeds the variance computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:3a3377613c9e5969294f2a981ba5a7666a8658b860d8ff8e18254724d6c995d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - R
  - SuperLU
  - sparseinv
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-tile-standard-error-computation

## Summary

Computes pointwise standard errors of GenoGAM tile smooth estimates by extracting only the needed elements of the inverse penalized Hessian via the sparse inverse subset algorithm (sparseinv R package wrapping SuiteSparse), avoiding explicit formation of the full dense inverse.

## When to use

Use when you need pointwise standard errors or variance of GAM smooth estimates for base-level significance testing in GenoGAM tile fits, where the penalized Hessian is large and sparse so a full inverse is infeasible.

## When NOT to use

- When the smoothing hyperparameters lambda and theta are unknown and must be estimated — the vignette workflow fixes lambda = 4601 and theta = 4.51 to skip parameter estimation, and this skill presumes a fitted tile
- When the Hessian is small or dense enough that a full inverse is affordable — the sparse inverse subset machinery is unnecessary overhead there
- When the goal is fitting the model itself with SuperLU rather than post-fit uncertainty quantification

## Inputs

- Fitted GenoGAM tile result object from genogam() with fixed hyperparameters (e.g. lambda = 4601, theta = 4.51)
- Penalized Hessian H of the tile fit with its nonzero sparsity pattern NZ(H)
- HDF5-backed GenoGAMDataSet used for the fit (chunkSize = 50000, overhangSize = 1000 in the vignette)

## Outputs

- Pointwise variance vector sigma^2 (diagonal of the inverse Hessian restricted to NZ(H))
- Per-tile standard-error vector (e.g. tile_standard_errors.rds), cross-checked against se(result)
- Fit and standard-error data frames accessible via fits(result) and se(result), indexed per chromosome (e.g. se(result)$chrI[,'s(x)'])

## How to apply

Load the fitted GenoGAM tile produced by genogam() (with fixed hyperparameters, e.g. lambda = 4601 and theta = 4.51) in R with the fastGenoGAM package. Assemble the penalized Hessian H of the tile fit and identify its nonzero sparsity pattern NZ(H). Compute the sparse Cholesky factorization of H, then use the sparseinv R package (a wrapper of SuiteSparse sparse inverse subset code) to evaluate only the elements of H^-1 that lie within NZ(H). Extract the diagonal entries to form the pointwise variance vector sigma^2 and take square roots to obtain the standard errors. The rationale is that the variance of smooth estimates requires only diagonal elements of the inverse Hessian, and sparse inverse subset methods recover exactly these without O(n^2) or O(n^3) dense computations, which is what makes GenoGAM 2.0 2 to 3 orders of magnitude faster. Store the per-tile standard-error vector (e.g. as tile_standard_errors.rds) and cross-check against the package's se(result) accessor on the same tile fit.

## Related tools

- **GenoGAM 2.0** (Package whose tile fits and se(result)/fits(result) accessors supply and validate the standard errors) — https://github.com/gstricker/fastGenoGAM
- **R** (Language in which GenoGAM 2.0 is implemented and in which the sparse inverse subset computation is run)
- **sparseinv** (R package wrapping SuiteSparse code to compute only the required subset of inverse-Hessian elements)
- **SuperLU** (Direct sparse solver used for parameter fitting whose sparse Hessian structure feeds the variance computation)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result)
```

## Evaluation signals

- All computed standard errors are finite and non-negative
- The computed per-tile standard-error vector matches the values returned by se(result) on the same tile fit
- Only elements of H^-1 within the nonzero sparsity pattern NZ(H) are evaluated — no dense inverse is formed
- Standard errors vary smoothly along the chromosome and are usable in fits(result)$chrI[,'s(x)'] +/- 2*se(result)$chrI[,'s(x)'] envelopes

## Limitations

- The fastGenoGAM version is deprecated in favor of the Bioconductor GenoGAM release; new work should target the current version
- Large matrix operations rely on Armadillo with ARMA_64BIT_WORD enabled; if not enabled at compile time, dimension-incompatibility errors can occur during fitting on large data
- Requires R 3.4.1 or higher; performance is tied to model sparsity and per-tile parameter count, scaling linearly with the number of parameters per tile
- Vignette example is restricted to the first 100kb of the first yeast chromosome and uses fixed hyperparameters to skip estimation

## Evidence

- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] result <- genogam(ggd, lambda = 4601, theta = 4.51): "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] ## Check the fit and standard error
fits(result)
se(result): "## Check the fit and standard error
fits(result)
se(result)"
- [discussion] Runtime and memory footprint of GenoGAM scale linearly with the number of parameters per tiles: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
