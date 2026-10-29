---
name: sparse-cholesky-factorization
description: Use when standard errors are needed for a fitted GenoGAM tile (e.g. genogam()
  output with lambda = 4601, theta = 4.51) but the sparse penalized Hessian is too
  large for a dense inverse.
when_to_use_negative:
- The model matrix is small enough that a dense matrix inverse is feasible — the sparse
  inverse subset machinery adds complexity without benefit
- The task requires off-diagonal covariances of H^-1 outside NZ(H), which the sparse
  inverse subset algorithm does not evaluate
- The input is raw count data not yet fitted — use GenoGAMDataSet(), computeSizeFactors(),
  and genogam() first
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: Provides the fitted tile whose penalized Hessian sparsity structure enables
    variance computation via the sparse inverse subset
  repo: https://github.com/gstricker/fastGenoGAM
- name: sparseinv
  role: R package wrapper of SuiteSparse code that evaluates only the elements of
    the inverse Hessian within NZ(H), yielding the variances
- name: SuperLU
  role: Sparse direct solver used by GenoGAM 2.0 to exploit model sparsity for parameter
    fitting, the same sparsity principle underlying this variance computation
- name: R
  role: Environment in which the sparse Hessian is assembled, factorized, and the
    standard errors extracted and validated against se(result)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:e431527fa6968989830d2166db2d3b2caf9cc93da837dc19ca3b751181a1622b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - R
  - SuperLU
  - sparseinv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sparse-cholesky-factorization

## Summary

Computes pointwise standard errors of GenoGAM smooth estimates by sparse Cholesky factorization of the penalized Hessian and evaluating only the inverse elements within the Hessian's sparsity pattern, avoiding explicit full-matrix inversion.

## When to use

Use when standard errors are needed for a fitted GenoGAM tile (e.g. genogam() output with lambda = 4601, theta = 4.51) but the sparse penalized Hessian is too large for a dense inverse.

## When NOT to use

- The model matrix is small enough that a dense matrix inverse is feasible — the sparse inverse subset machinery adds complexity without benefit
- The task requires off-diagonal covariances of H^-1 outside NZ(H), which the sparse inverse subset algorithm does not evaluate
- The input is raw count data not yet fitted — use GenoGAMDataSet(), computeSizeFactors(), and genogam() first

## Inputs

- Fitted GenoGAM tile object produced by genogam() (e.g. with fixed hyperparameters lambda = 4601, theta = 4.51)
- Sparse penalized Hessian matrix H of the tile fit and its nonzero sparsity pattern NZ(H)

## Outputs

- Pointwise variance vector sigma^2 (diagonal of the sparse inverse subset of H^-1)
- Per-tile standard-error vector, cross-checked against se(result), storable as tile_standard_errors.rds

## How to apply

Assemble the penalized Hessian H of the GenoGAM tile fit and identify its nonzero sparsity pattern NZ(H). Compute the sparse Cholesky factorization of H, then use the sparse inverse subset algorithm (R package sparseinv, a wrapper of SuiteSparse code) to evaluate only the elements of H^-1 that lie within NZ(H). Extract the diagonal entries of this partial inverse to form the pointwise variance vector sigma^2 and take square roots to obtain standard errors. This mirrors how GenoGAM 2.0 exploits model sparsity — the same principle behind using the SuperLU direct solver for parameter fitting — so the computation scales linearly with the number of parameters per tile. Cross-check the resulting per-tile standard-error vector against the package's se(result) accessor on the same fit before storing it.

## Related tools

- **GenoGAM 2.0** (Provides the fitted tile whose penalized Hessian sparsity structure enables variance computation via the sparse inverse subset) — https://github.com/gstricker/fastGenoGAM
- **sparseinv** (R package wrapper of SuiteSparse code that evaluates only the elements of the inverse Hessian within NZ(H), yielding the variances)
- **SuperLU** (Sparse direct solver used by GenoGAM 2.0 to exploit model sparsity for parameter fitting, the same sparsity principle underlying this variance computation)
- **R** (Environment in which the sparse Hessian is assembled, factorized, and the standard errors extracted and validated against se(result))

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result)
```

## Evaluation signals

- All computed standard errors are finite and non-negative
- The computed per-tile standard-error vector matches the values returned by se(result) on the same tile fit
- Only elements of H^-1 within NZ(H) are evaluated; the Cholesky factorization of H succeeds without fill-in blow-up
- Runtime and memory scale linearly with the number of parameters per tile, consistent with the paper's scaling claim

## Limitations

- fastGenoGAM (the reference implementation) is deprecated in favor of the Bioconductor GenoGAM release, so repository documentation may lag
- GenoGAM 2.0 requires R 3.4.1 or higher and is licensed under GPL-2, constraining compatible environments and redistribution
- Large-matrix errors in RcppArmadillo (e.g. 'incompatible matrix dimensions') arise when ARMA_64BIT_WORD is not enabled and must be fixed at compilation time
- The sparse inverse subset yields only inverse elements within NZ(H); other covariance entries are not available

## Evidence

- [abstract] SuperLU sparsity: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [discussion] scaling: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [readme] fixed hyperparameters fit: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] standard error accessor: "## Check the fit and standard error
fits(result)
se(result)"
