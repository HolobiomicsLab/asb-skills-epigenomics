---
name: sparse-matrix-inverse-subset-computation
description: Use when standard errors or variances of fitted smooths are needed from
  a sparse penalized GAM fit (e.g., GenoGAM tiles with lambda = 4601, theta = 4.51)
  and explicitly inverting the Hessian would be too slow or memory-intensive.
when_to_use_negative:
- When only the model fit or p-values are needed and no standard errors/variances
  of smooths are required (e.g., just calling computeSignificance(result))
- When the Hessian is small or dense enough that a direct inverse or mgcv's indirect
  method is affordable without the sparse inverse subset machinery
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: sparseinv
  role: R wrapper of SuiteSparse code that evaluates only the elements of the inverse
    sparse Hessian within NZ(H) needed for the variance
- name: GenoGAM 2.0 (fastGenoGAM)
  role: Provides the fitted GenoGAM tiles and imports the sparse inverse subset computation
    for standard errors
  repo: https://github.com/gstricker/fastGenoGAM
- name: mgcv
  role: General-purpose GAM fitting library underlying GenoGAM; its indirect Simon
    Wood method serves as the baseline alternative for variance computation
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:d23c654f331036d968fe18b76279e2000e7672f44f21dc2df8d6912286d198ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM)
  - Matrix
  - R
  - Rcpp
  - mgcv
  - sparseinv
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sparse-matrix-inverse-subset-computation

## Summary

Computes the diagonal of the inverse of a sparse penalized Hessian (i.e., pointwise variances/standard errors of GAM smooth estimates) using the sparse inverse subset algorithm implemented in the R package sparseinv, a wrapper of SuiteSparse code, instead of explicitly forming the full matrix inverse.

## When to use

Use when standard errors or variances of fitted smooths are needed from a sparse penalized GAM fit (e.g., GenoGAM tiles with lambda = 4601, theta = 4.51) and explicitly inverting the Hessian would be too slow or memory-intensive.

## When NOT to use

- When only the model fit or p-values are needed and no standard errors/variances of smooths are required (e.g., just calling computeSignificance(result))
- When the Hessian is small or dense enough that a direct inverse or mgcv's indirect method is affordable without the sparse inverse subset machinery

## Inputs

- Fitted GenoGAM tile object from genogam() (fixed hyperparameters lambda = 4601, theta = 4.51)
- Penalized Hessian matrix H of the tile fit with its nonzero sparsity pattern NZ(H)

## Outputs

- Pointwise variance vector sigma^2 (diagonal of the Hessian inverse restricted to NZ(H))
- Per-tile standard-error vector, cross-checkable against se(result) (tile_standard_errors.rds)

## How to apply

Assemble the penalized Hessian H of the fitted tile (e.g., from a GenoGAM fit produced with genogam(ggd, lambda = 4601, theta = 4.51)) and identify its nonzero sparsity pattern NZ(H). Compute the sparse Cholesky factorization of H, then call the sparseinv R package (SuiteSparse sparse inverse subset) to evaluate only those elements of H^-1 that lie within NZ(H). Extract the diagonal entries to form the pointwise variance vector sigma^2 and take square roots to obtain standard errors. This is the approach GenoGAM 2.0 uses for statistical testing; compared to Simon Wood's indirect method (mgcv) on a single core, it is roughly two orders of magnitude faster with linear rather than quadratic runtime scaling, and its memory slope is about four times lower.

## Related tools

- **sparseinv** (R wrapper of SuiteSparse code that evaluates only the elements of the inverse sparse Hessian within NZ(H) needed for the variance)
- **GenoGAM 2.0 (fastGenoGAM)** (Provides the fitted GenoGAM tiles and imports the sparse inverse subset computation for standard errors) — https://github.com/gstricker/fastGenoGAM
- **mgcv** (General-purpose GAM fitting library underlying GenoGAM; its indirect Simon Wood method serves as the baseline alternative for variance computation)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result)
```

## Evaluation signals

- All computed standard errors are finite and non-negative
- The sparse-inverse-subset standard-error vector matches the values returned by se(result) on the same tile fit
- On a single core, runtime is about two orders of magnitude lower than the indirect Simon Wood (mgcv) method at matched parameter counts
- Log-log memory slope of the sparse method is about four times shallower than the indirect method (both linear in memory)
- Runtime scales linearly (not quadratically) with the number of parameters per tile

## Limitations

- Requires the Hessian to be sparse and its sparsity pattern to contain the needed inverse elements; unsuitable for dense Hessians
- Performance claims are for single-core comparisons on benchmark hardware ('a typical lab server', 'a standard PC') with exact specifications unspecified
- Requires R 3.4.1 or higher; large-matrix computations may fail unless ARMA_64BIT_WORD is enabled in RcppArmadillo

## Evidence

- [other] GenoGAM 2.0 computes standard errors with the sparse inverse subset algorithm: [evidence span withheld — anchor status: not_found]
- [other] On a single core, the sparse inverse subset algorithm computes standard errors about 2 orders of magnitude faster: [evidence span withheld — anchor status: not_found]
- [readme] Compute position-wise log p-values with computeSignificance: "compute position-wise log p-values with `computeSignificance()`"
- [methods] result <- genogam(ggd, lambda = 4601, theta = 4.51): "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
