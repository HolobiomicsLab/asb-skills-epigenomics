---
name: sparse-hessian-linear-system-solving
description: Use when fitting penalized GAMs (e.g., negative-binomial ChIP-Seq count
  models) whose Hessian is sparse, and you need direct sparse solves of the Newton-Raphson
  system rather than dense matrix inversion.
when_to_use_negative:
- The Hessian or design matrix is dense — sparse storage and SuperLU would add overhead
  without benefit
- You need the inverse Hessian itself (e.g., for full covariance estimation) rather
  than only solving the Newton step
- Smooth hyperparameters must be estimated from data by cross-validation rather than
  fixed — this skill covers the fitting step with fixed lambda/theta
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: SuperLU
  role: Direct sparse linear solver used for parameter fitting, exploiting model sparsity
    instead of forming the Hessian inverse
- name: GenoGAM 2.0
  role: R package whose per-tile penalized Newton-Raphson fitting invokes the sparse
    storage and SuperLU solves; available as fastGenoGAM
  repo: https://github.com/gstricker/fastGenoGAM
- name: R
  role: Implementation language; GenoGAM is an open source R package built on mgcv
    and Bioconductor infrastructure
  repo: https://github.com/gstricker/GenoGAM
- name: mgcv
  role: General-purpose R library for fitting GAMs on which GenoGAM's model framework
    is based
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:2ed5e7355d9499f49a2a652ce0f2893aa1b6382b9357fdbf456e26968f01b225
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# sparse-hessian-linear-system-solving

## Summary

Solves the penalized Newton-Raphson fitting equations of GenoGAM 2.0 by exploiting the sparsity of the model Hessian: instead of forming the inverse Hessian, the sparse linear system is solved directly with SuperLU and all matrices are stored in sparse format to avoid redundant storage of zeros. This is the key trick that makes GenoGAM 2.0 2 to 3 orders of magnitude faster than the previous version.

## When to use

Use when fitting penalized GAMs (e.g., negative-binomial ChIP-Seq count models) whose Hessian is sparse, and you need direct sparse solves of the Newton-Raphson system rather than dense matrix inversion.

## When NOT to use

- The Hessian or design matrix is dense — sparse storage and SuperLU would add overhead without benefit
- You need the inverse Hessian itself (e.g., for full covariance estimation) rather than only solving the Newton step
- Smooth hyperparameters must be estimated from data by cross-validation rather than fixed — this skill covers the fitting step with fixed lambda/theta

## Inputs

- GenoGAMDataSet restricted to the first 100kb of chromosome I of yeast (example dataset), with size factors computed via computeSizeFactors()
- Fixed smoothing hyperparameters lambda = 4601 and theta = 4.51
- Design formula (e.g. ~ s(x) + s(x, by = genotype)) and chunk/overhang sizes (chunkSize 50000, overhangSize 1000)

## Outputs

- A GenoGAM object with per-tile fitted coefficients
- Fitted smooth functions retrievable via fits(result) and standard errors via se(result)
- Optionally the serialized GenoGAM object (genogam_fit.rds)

## How to apply

During each penalized Newton-Raphson iteration of the per-tile negative-binomial GAM fit, store the design and Hessian matrices in sparse (Matrix/RcppArmadillo) format rather than dense arrays. Never form the Hessian inverse; instead solve the linear system H * delta = gradient with the SuperLU direct solver to obtain the coefficient update step. Set up the fit per tile (overlapping genomic intervals, e.g. chunkSize 50000 with overhang 1000) so that both runtime and memory footprint scale linearly with the number of parameters per tile. If fitting fails with Armadillo matrix-dimension errors on large systems, enable ARMA_64BIT_WORD since RcppArmadillo defaults to 32bit matrices. Validate by extracting fits(result) and se(result) and confirming finite fitted values and standard errors across the tile ranges.

## Related tools

- **SuperLU** (Direct sparse linear solver used for parameter fitting, exploiting model sparsity instead of forming the Hessian inverse)
- **GenoGAM 2.0** (R package whose per-tile penalized Newton-Raphson fitting invokes the sparse storage and SuperLU solves; available as fastGenoGAM) — https://github.com/gstricker/fastGenoGAM
- **R** (Implementation language; GenoGAM is an open source R package built on mgcv and Bioconductor infrastructure) — https://github.com/gstricker/GenoGAM
- **mgcv** (General-purpose R library for fitting GAMs on which GenoGAM's model framework is based)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); fits(result); se(result)
```

## Evaluation signals

- fits(result) and se(result) return finite fitted values and standard errors across all tile ranges
- No Armadillo 'matrix multiplication: incompatible matrix dimensions' errors during fitting (would indicate 32bit matrix overflow; fix with ARMA_64BIT_WORD)
- Runtime and memory footprint scale linearly with the number of parameters per tile
- Resulting object is a valid GenoGAM object and can be serialized and reloaded (e.g. as genogam_fit.rds)
- Downstream computeSignificance(result) yields finite position-wise log p-values

## Limitations

- Requires R 3.4.1 or higher, constraining compatible environments
- Armadillo uses 32bit matrices by default in RcppArmadillo, causing dimension overflow errors on very large systems unless ARMA_64BIT_WORD is enabled
- The fastGenoGAM GitHub version is deprecated in favor of the Bioconductor GenoGAM release
- Software licensed under GPL-2, which may impose redistribution constraints

## Evidence

- [abstract] SuperLU direct solver for sparsity exploitation: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [discussion] Runtime scaling with parameters per tile: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [readme] Fixed hyperparameter fit invocation: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] Armadillo 64bit word fix: "The solution is to enable `ARMA_64BIT_WORD`, which is not enabled in RcppArmadillo by default."
