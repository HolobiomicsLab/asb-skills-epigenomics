---
name: genomic-tiled-model-fitting
description: Use when base-level ChIP-Seq count data along large chromosomes must
  be modeled with smooth functions genome-wide, and a full-genome GAM fit is intractable
  without tiling and sparse-matrix parameter fitting.
when_to_use_negative:
- The genome/region is small enough that a single monolithic GAM fit (e.g. via mgcv
  directly) is tractable without tiling overhead
- The analysis goal is region-level peak calling rather than base-level smooth-function
  modeling and significance testing with data-driven smoothing
- Input is raw read counts without an experiment design defining the GAM covariates
  (position, genotype, etc.)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: R package implementing the tiled penalized Newton-Raphson fitting of the negative-binomial
    GAM with sparse matrix storage
  repo: https://github.com/gstricker/GenoGAM
- name: SuperLU
  role: Direct sparse solver used to solve the linear systems during parameter fitting
    without forming the Hessian inverse
- name: mgcv
  role: General-purpose R library underpinning the GAM fitting framework used by GenoGAM
- name: HDF5
  role: Out-of-core backend that stores GenoGAMDataSet data on hard drive to reduce
    memory footprint while keeping I/O low
- name: fastGenoGAM
  role: Deprecated predecessor implementation of the same tiled workflow; README documents
    the GenoGAMDataSet → computeSizeFactors → genogam pipeline
  repo: https://github.com/gstricker/fastGenoGAM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:3cb05edd311c68f8f5e257084463be266e631d4acc9fa7e541d4a44904882330
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/GenoGAM
  tools:
  - GenoGAM 2.0
  - HDF5
  - R
  - SuperLU
  - fastGenoGAM
  - mgcv
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# genomic-tiled-model-fitting

## Summary

Fits penalized negative-binomial generalized additive models on overlapping genomic tiles using GenoGAM 2.0, exploiting model sparsity with the SuperLU direct solver (never forming the Hessian inverse) so that runtime and memory scale linearly with parameters per tile.

## When to use

Use when base-level ChIP-Seq count data along large chromosomes must be modeled with smooth functions genome-wide, and a full-genome GAM fit is intractable without tiling and sparse-matrix parameter fitting.

## When NOT to use

- The genome/region is small enough that a single monolithic GAM fit (e.g. via mgcv directly) is tractable without tiling overhead
- The analysis goal is region-level peak calling rather than base-level smooth-function modeling and significance testing with data-driven smoothing
- Input is raw read counts without an experiment design defining the GAM covariates (position, genotype, etc.)

## Inputs

- BAM files with an experiment design file (experimentDesign.txt) describing the ChIP-Seq factorial design
- GenoGAMDataSet object built with HDF5 backend, chunkSize = 50000, overhangSize = 1000, and a GAM design formula (e.g. ~ s(x) + s(x, by = genotype))
- Size factors computed via computeSizeFactors()
- Optional fixed smoothing hyperparameters (lambda, theta)

## Outputs

- GenoGAM object containing per-tile fitted smooth functions (accessible via fits())
- Standard errors of the fits (accessible via se())
- Position-wise log p-values from computeSignificance()
- Serialized fitted object (e.g. genogam_fit.rds)

## How to apply

Restrict the input to a tractable genomic region (the package example uses the first 100kb of yeast chromosome I) and build a GenoGAMDataSet with an HDF5 backend, chunkSize = 50000 and overhangSize = 1000 so multiple tiles are generated, plus a design such as ~ s(x) + s(x, by = genotype). Compute size factors with computeSizeFactors() before fitting. Call genogam() with either data-driven smoothing or fixed hyperparameters (e.g. lambda = 4601, theta = 4.51 in the vignette); fitting proceeds by penalized Newton-Raphson per tile, with design and Hessian matrices stored in sparse (Matrix) format and the sparse linear systems solved directly with SuperLU rather than inverting the Hessian. Extract fitted smooths and standard errors with fits(result) and se(result), and optionally compute position-wise log p-values with computeSignificance(). Tile fits are joined at overlap midpoints, giving linear computation time and parallelization across tiles.

## Related tools

- **GenoGAM 2.0** (R package implementing the tiled penalized Newton-Raphson fitting of the negative-binomial GAM with sparse matrix storage) — https://github.com/gstricker/GenoGAM
- **SuperLU** (Direct sparse solver used to solve the linear systems during parameter fitting without forming the Hessian inverse)
- **mgcv** (General-purpose R library underpinning the GAM fitting framework used by GenoGAM)
- **HDF5** (Out-of-core backend that stores GenoGAMDataSet data on hard drive to reduce memory footprint while keeping I/O low)
- **fastGenoGAM** (Deprecated predecessor implementation of the same tiled workflow; README documents the GenoGAMDataSet → computeSizeFactors → genogam pipeline) — https://github.com/gstricker/fastGenoGAM

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51); fits(result); se(result)
```

## Evaluation signals

- fits(result) and se(result) return finite fitted values and standard errors across all tile ranges
- Tile fits join consistently at overlap midpoints between adjacent tiles
- Runtime and memory footprint scale linearly with the number of parameters per tile
- Design and Hessian matrices are stored in sparse (Matrix) format — no dense inverse of the Hessian is formed
- computeSignificance(result) yields finite position-wise log p-values retrievable via pval(result)

## Limitations

- fastGenoGAM version is deprecated; use the Bioconductor GenoGAM release or gstricker/GenoGAM
- Requires R 3.4.1 or higher; licensed under GPL-2
- RcppArmadillo must have ARMA_64BIT_WORD enabled to avoid 32-bit matrix dimension overflow errors during large matrix multiplication
- Warnings about out-of-bound ranges during genome tiling cannot be suppressed in the parallel Snow environment
- Out-of-bound range warnings appear during the tile shifting process and are not reliably suppressible via suppressWarnings in Snow parallel backends

## Evidence

- [methods] Per-tile parameter fitting uses Newton-Raphson iteration; because the Hessian is sparse, its inverse is never formed: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [intro] Tiles are overlapping genomic intervals joined at midpoints for linear scaling.: "Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints."
- [readme] The vignette fixes hyperparameters for fitting without parameter estimation.: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [abstract] HDF5 backend reduces memory footprint while keeping I/O low.: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [discussion] Runtime and memory scale linearly with parameters per tile.: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
