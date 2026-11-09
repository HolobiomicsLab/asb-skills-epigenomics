---
name: bayesian-gam-posterior-variance-estimation
description: Use when pointwise variance or standard errors of smooth estimates are
  needed for statistical testing in a sparse penalized GAM fit where forming the full
  inverse Hessian is infeasible.
when_to_use_negative:
- When the full inverse Hessian is small enough to form explicitly and a dense covariance
  matrix is actually required beyond NZ(H)
- When the analysis goal is parameter fitting rather than posterior variance estimation
  — parameter fitting uses the SuperLU direct solver, not sparseinv
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: sparseinv
  role: R wrapper of SuiteSparse sparse inverse subset code; evaluates only the elements
    of the inverse Hessian needed for the variance
- name: GenoGAM 2.0 (fastGenoGAM/GenoGAM)
  role: Provides the fitted tile model, Hessian, and the se(result) accessor used
    as reference output
  repo: https://github.com/gstricker/fastGenoGAM
- name: R
  role: Implementation environment for the variance computation workflow
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Direct sparse solver used for parameter fitting that produces the sparse model
    structure the Hessian inherits
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:8fc7fbee3b4682807f1824b1e240d08a3b19a241cac378a95e94cd998986d918
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM/GenoGAM)
  - R
  - SuperLU
  - sparseinv
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# bayesian-gam-posterior-variance-estimation

## Summary

Computes pointwise standard errors of GAM smooth estimates in GenoGAM 2.0 by evaluating only the needed elements of the inverse penalized Hessian with the sparse inverse subset algorithm (sparseinv/SuiteSparse), avoiding explicit full matrix inversion.

## When to use

Use when pointwise variance or standard errors of smooth estimates are needed for statistical testing in a sparse penalized GAM fit where forming the full inverse Hessian is infeasible.

## When NOT to use

- When the full inverse Hessian is small enough to form explicitly and a dense covariance matrix is actually required beyond NZ(H)
- When the analysis goal is parameter fitting rather than posterior variance estimation — parameter fitting uses the SuperLU direct solver, not sparseinv

## Inputs

- Fitted GenoGAM tile object from genogam() (fastGenoGAM/GenoGAM R package)
- Penalized Hessian H of the tile fit and its nonzero sparsity pattern NZ(H)
- Fixed hyperparameters (e.g. lambda = 4601, theta = 4.51)

## Outputs

- Sparse subset of H^-1 elements within NZ(H)
- Pointwise variance vector sigma^2 (diagonal of the inverse Hessian subset)
- Per-tile standard-error vector, e.g. tile_standard_errors.rds

## How to apply

Load the fitted GenoGAM tile produced by genogam() (e.g. with fixed hyperparameters lambda = 4601, theta = 4.51) in R with the fastGenoGAM package. Assemble the penalized Hessian H of the tile fit and identify its nonzero sparsity pattern NZ(H). Compute the sparse Cholesky factorization of H, then use the sparseinv R package (a wrapper of SuiteSparse sparse inverse subset code) to evaluate only those elements of H^-1 lying within NZ(H). Extract the diagonal entries to form the pointwise variance vector sigma^2 and take square roots to obtain standard errors; store them per tile (e.g. tile_standard_errors.rds) and cross-check against the package's se(result) accessor.

## Related tools

- **sparseinv** (R wrapper of SuiteSparse sparse inverse subset code; evaluates only the elements of the inverse Hessian needed for the variance)
- **GenoGAM 2.0 (fastGenoGAM/GenoGAM)** (Provides the fitted tile model, Hessian, and the se(result) accessor used as reference output) — https://github.com/gstricker/fastGenoGAM
- **R** (Implementation environment for the variance computation workflow) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Direct sparse solver used for parameter fitting that produces the sparse model structure the Hessian inherits)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); se(result)
```

## Evaluation signals

- All computed standard errors are finite and non-negative
- The computed standard-error vector matches the values returned by se(result) on the same tile fit
- Only elements of H^-1 within NZ(H) are evaluated (no full inversion performed)
- Square roots of the diagonal variance entries reproduce the pointwise standard-error vector

## Limitations

- fastGenoGAM is deprecated in favor of the newer GenoGAM Bioconductor version
- Requires R 3.4.1 or higher; GPL-2 license may impose redistribution constraints
- Large-matrix errors can occur if RcppArmadillo does not enable ARMA_64BIT_WORD

## Evidence

- [other] GenoGAM 2.0 computes standard errors with the sparse inverse subset algorithm: [evidence span withheld — anchor status: not_found]
- [other] the implementation is based on the R package sparseinv: [evidence span withheld — anchor status: not_found]
- [abstract] SuperLU direct solver exploits model sparsity: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [readme] README fixed-hyperparameter fit invocation: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] README standard error accessor: "## Check the fit and standard error"
