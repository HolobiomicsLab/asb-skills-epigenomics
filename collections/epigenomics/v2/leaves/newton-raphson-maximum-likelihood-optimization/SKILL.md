---
name: newton-raphson-maximum-likelihood-optimization
description: Use when fitting penalized GAM-type regression models (e.g. smooth functions
  of genomic position s(x) on ChIP-Seq counts) where the Hessian of the penalized
  negative-binomial likelihood is sparse and a direct sparse solve is preferable to
  forming the inverse.
when_to_use_negative:
- The analysis requires data-driven smoothing hyperparameter estimation by cross-validation
  — fixing lambda and theta (as in the vignette fit) skips that objective estimation
- The target is position-wise significance testing (computeSignificance()) or plotting
  (plotGenoGAM()); those are downstream steps operating on an already-fitted GenoGAM
  object
- The problem is dense (no exploitable sparsity in the Hessian), in which case the
  SuperLU sparse direct solver offers no advantage
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: GenoGAM 2.0
  role: R package whose genogam() function runs the per-tile penalized Newton-Raphson
    fit of the negative-binomial GAM
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Direct sparse solver used for parameter fitting; solves the Newton-Raphson
    linear system without forming the inverse Hessian
- name: mgcv
  role: General-purpose R library underlying the GAM framework that GenoGAM builds
    on
- name: HDF5
  role: Out-of-core storage backend keeping the tiled count data on disk, reducing
    memory footprint while keeping I/O low during fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:c2e6aff0f687609cfee72bebc66fbff79639c12d4ac3f3ede66f1fdd958c369a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - HDF5
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# newton-raphson-maximum-likelihood-optimization

## Summary

Fits the penalized negative-binomial generalized additive model per genomic tile in GenoGAM 2.0 via Newton-Raphson iteration, solving the sparse linear system with the SuperLU direct solver so the inverse Hessian is never formed. This exploitation of model sparsity is what makes whole-genome fits (ca. 300 million parameters) tractable.

## When to use

Use when fitting penalized GAM-type regression models (e.g. smooth functions of genomic position s(x) on ChIP-Seq counts) where the Hessian of the penalized negative-binomial likelihood is sparse and a direct sparse solve is preferable to forming the inverse.

## When NOT to use

- The analysis requires data-driven smoothing hyperparameter estimation by cross-validation — fixing lambda and theta (as in the vignette fit) skips that objective estimation
- The target is position-wise significance testing (computeSignificance()) or plotting (plotGenoGAM()); those are downstream steps operating on an already-fitted GenoGAM object
- The problem is dense (no exploitable sparsity in the Hessian), in which case the SuperLU sparse direct solver offers no advantage

## Inputs

- GenoGAMDataSet object (ChIP-Seq read counts tiled over overlapping genomic intervals, chunkSize = 50000, overhangSize = 1000) with computed size factors
- Fixed smoothing hyperparameters lambda and theta (e.g. lambda = 4601, theta = 4.51)
- Model design formula, e.g. ~ s(x) + s(x, by = genotype)

## Outputs

- GenoGAM object containing per-tile penalized Newton-Raphson fits of the negative-binomial GAM
- Fitted smooth functions per chromosome/position via fits() (e.g. s(x), s(x):genotype)
- Pointwise standard errors via se()
- Optionally serialized RDS file (e.g. genogam_fit.rds)

## How to apply

Load a GenoGAMDataSet (optionally with the HDF5 backend) with size factors already computed via computeSizeFactors(), then call genogam() on it. With fixed smoothing hyperparameters — the vignette uses lambda = 4601 and theta = 4.51 — genogam() triggers penalized Newton-Raphson fitting of the negative-binomial GAM per tile. Design and Hessian matrices are stored in sparse (Matrix) format and each iteration's linear system is solved with the SuperLU direct solver; the inverse Hessian is never formed because it would be dense and wasteful. Extract fitted smooths and standard errors with fits() and se(), and serialize the result (e.g. genogam_fit.rds). Rationale: exploiting sparsity plus out-of-core HDF5 processing gives a 2 to 3 orders of magnitude speedup over GenoGAM 1.x, with runtime and memory scaling linearly with the number of parameters per tile.

## Related tools

- **GenoGAM 2.0** (R package whose genogam() function runs the per-tile penalized Newton-Raphson fit of the negative-binomial GAM) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Direct sparse solver used for parameter fitting; solves the Newton-Raphson linear system without forming the inverse Hessian)
- **mgcv** (General-purpose R library underlying the GAM framework that GenoGAM builds on)
- **HDF5** (Out-of-core storage backend keeping the tiled count data on disk, reducing memory footprint while keeping I/O low during fitting)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- fits(result) and se(result) return finite fitted values and standard errors across the tile ranges (e.g. for chrI over the first 100kb of yeast chromosome I)
- No dense-matrix blow-up errors like the RcppArmadillo 'matrix multiplication: incompatible matrix dimensions' error reported in the README FAQ (fixed by enabling ARMA_64BIT_WORD)
- Runtime and memory footprint scale linearly with the number of parameters per tiles, consistent with the paper's discussion
- Tile fits join smoothly at overlap midpoints (vignette plots the center 10kb, positions 45000:55000, where both tiles meet)
- Re-running with identical fixed lambda = 4601 and theta = 4.51 reproduces the same fitted smooths

## Limitations

- Requires R 3.4.1 or higher and correctly installed Armadillo dependencies; large matrices need ARMA_64BIT_WORD enabled to avoid 32-bit matrix overflow errors
- Performance claims reference hardware ('a typical lab server', 'a standard 60-core server') without exact specifications
- The fastGenoGAM repository version is deprecated in favor of the Bioconductor GenoGAM release
- Software licensed under GPL-2, which may impose redistribution constraints

## Evidence

- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [discussion] We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [methods] result <- genogam(ggd, lambda = 4601, theta = 4.51): "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
