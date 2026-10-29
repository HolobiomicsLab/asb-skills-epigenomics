---
name: negative-binomial-count-model-regression
description: Use when modeling base-resolution ChIP-Seq read counts as smooth functions
  of genomic position with a penalized NB GAM, especially genome-wide where per-tile
  sparsity must be exploited for tractable fitting.
when_to_use_negative:
- The data are not base-level count data suitable for a negative-binomial link (e.g.
  continuous intensities or already-normalized feature tables)
- A dense, small-scale regression is needed where the overhead of sparse Matrix storage,
  tiling, and HDF5 out-of-core processing is unnecessary
- Ad-hoc smoothing over binned/windowed data is acceptable and data-driven cross-validated
  smoothing or tile-joining is not required
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GenoGAM 2.0
  role: R package providing genogam() with penalized Newton-Raphson NB-GAM fitting
    per tile using sparse matrices
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Direct solver used to solve the sparse Hessian linear systems during parameter
    fitting without forming the inverse
- name: mgcv
  role: General-purpose R GAM library on which the GenoGAM model framework is built
- name: R
  role: Implementation language; requires R 3.4.1 or higher
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:f7075073356a99f0f22fae28f261a07bcfd14682641b0032461f6ea70c52b41a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - GenoGAM 2.0
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# negative-binomial-count-model-regression

## Summary

Fits penalized negative-binomial generalized additive model coefficients per genomic tile by Newton-Raphson iteration in GenoGAM 2.0, exploiting sparsity of the design and Hessian matrices: the Hessian inverse is never formed, and the sparse linear systems are solved with the SuperLU direct solver.

## When to use

Use when modeling base-resolution ChIP-Seq read counts as smooth functions of genomic position with a penalized NB GAM, especially genome-wide where per-tile sparsity must be exploited for tractable fitting.

## When NOT to use

- The data are not base-level count data suitable for a negative-binomial link (e.g. continuous intensities or already-normalized feature tables)
- A dense, small-scale regression is needed where the overhead of sparse Matrix storage, tiling, and HDF5 out-of-core processing is unnecessary
- Ad-hoc smoothing over binned/windowed data is acceptable and data-driven cross-validated smoothing or tile-joining is not required

## Inputs

- GenoGAMDataSet object (HDF5-backed) built from an experimentDesign.txt pointing to BAM files
- GAM design formula (e.g. ~ s(x) + s(x, by = genotype))
- Chunk and overhang sizes (e.g. chunkSize = 50000, overhangSize = 1000)
- Optional fixed smoothing hyperparameters (lambda = 4601, theta = 4.51)
- Computed size factors via computeSizeFactors()

## Outputs

- GenoGAM object containing per-tile fitted smooth functions
- Position-wise fitted values via fits(result)
- Position-wise standard errors via se(result)
- Optionally position-wise log p-values via computeSignificance(result)

## How to apply

Prepare a GenoGAMDataSet (e.g. restricted to the first 100kb of yeast chromosome I) with an experimental design such as ~ s(x) + s(x, by = genotype) and computed size factors, then call genogam() with either fixed hyperparameters (lambda = 4601, theta = 4.51) or data-driven smoothing. The per-tile penalized Newton-Raphson fitting keeps all design and Hessian matrices in sparse Matrix format and solves the sparse linear systems via the SuperLU direct solver instead of forming the Hessian inverse, which avoids redundant storage of zeros. Chunk sizes (e.g. chunkSize = 50000, overhangSize = 1000) define overlapping tiles whose fits are joined at overlap midpoints. Extract fitted smooths and standard errors with fits() and se(); this linear-scaling fitting is what enables whole-genome human fits (~300 million parameters) in under 9 hours on a 60-core server.

## Related tools

- **GenoGAM 2.0** (R package providing genogam() with penalized Newton-Raphson NB-GAM fitting per tile using sparse matrices) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Direct solver used to solve the sparse Hessian linear systems during parameter fitting without forming the inverse)
- **mgcv** (General-purpose R GAM library on which the GenoGAM model framework is built)
- **R** (Implementation language; requires R 3.4.1 or higher)

## Examples

```
result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- fits(result) and se(result) return finite fitted values and standard errors across all tile ranges of the GenoGAM object
- Runtime and memory footprint scale linearly with the number of parameters per tile, consistent with the paper's reported scaling
- Sparse storage is in effect: design and Hessian matrices are held in Matrix sparse format and no Hessian inverse is formed
- With fixed hyperparameters the fit reproduces without parameter estimation overhead (the README vignette shortcut)
- Tile fits join at overlap midpoints of adjacent tiles defined by chunkSize/overhangSize

## Limitations

- Requires R 3.4.1 or higher and correct Armadillo/RcppArmadillo 64-bit configuration (ARMA_64BIT_WORD) or large-matrix multiplication errors occur
- Performance claims reference hardware ('a typical lab server', 'a standard 60-core server') without exact specifications
- The fastGenoGAM repo is deprecated in favor of the Bioconductor GenoGAM release
- Suppressing out-of-bound range warnings during tiling via suppressWarnings does not work in the parallel Snow environment

## Evidence

- [abstract] SuperLU direct solver: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [discussion] linear scaling: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [methods] genogam fitting step: "Compute the model with `genogam()` to get the result `GenoGAM` object"
- [readme] fixed hyperparameter invocation: "## compute model without parameter estimation to save time in vignette
result <- genogam(ggd, lambda = 4601, theta = 4.51)"
