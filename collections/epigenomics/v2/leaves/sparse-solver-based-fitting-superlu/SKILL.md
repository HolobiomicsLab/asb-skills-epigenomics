---
name: sparse-solver-based-fitting-superlu
description: Use when fitting tiled GAMs to genome-scale ChIP-Seq count data where
  dense Newton-Raphson matrix operations are prohibitively slow or memory-heavy, and
  linear runtime/memory scaling in parameters-per-tile is required.
when_to_use_negative:
- When smoothing parameters must be estimated de novo by cross-validation on very
  small datasets where fitting cost is negligible and the added HDF5/sparse-solver
  setup is unnecessary
- When Armadillo/RcppArmadillo 64-bit word support is unavailable, causing 32-bit
  matrix dimension overflow errors during large sparse matrix operations
- For non-count, non-genomic regression problems that do not exhibit the banded sparsity
  structure GenoGAM's SuperLU solver exploits
edam_operation: http://edamontology.org/operation_2945
tools:
- name: GenoGAM 2.0
  role: R package providing genogam() fitting with the sparse Newton-Raphson method
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Direct sparse solver used for parameter fitting
- name: HDF5
  role: Out-of-core on-disk storage backend reducing memory footprint while keeping
    I/O low
- name: mgcv
  role: General-purpose R GAM library underlying the model framework and design formulas
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:bbbc8da6ff4d76ad89955903f3981b9e7cf03126ae4662c35144a176dc292c9c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sparse-solver-based-fitting-superlu

## Summary

Fit genome-wide generalized additive models by exploiting the sparsity of the banded penalty design matrix and solving the parameter-estimation problem with the SuperLU direct sparse solver, as implemented in GenoGAM 2.0 with out-of-core HDF5 data handling. This yields 2-3 orders of magnitude faster and far lighter fits than the dense-matrix approach of GenoGAM 1.0 (e.g. 1 s and 52 MB vs 1 h 37 min and 8 GB at 1104 parameters on a single core).

## When to use

Use when fitting tiled GAMs to genome-scale ChIP-Seq count data where dense Newton-Raphson matrix operations are prohibitively slow or memory-heavy, and linear runtime/memory scaling in parameters-per-tile is required.

## When NOT to use

- When smoothing parameters must be estimated de novo by cross-validation on very small datasets where fitting cost is negligible and the added HDF5/sparse-solver setup is unnecessary
- When Armadillo/RcppArmadillo 64-bit word support is unavailable, causing 32-bit matrix dimension overflow errors during large sparse matrix operations
- For non-count, non-genomic regression problems that do not exhibit the banded sparsity structure GenoGAM's SuperLU solver exploits

## Inputs

- BAM files with an experimentDesign.txt specifying the factorial ChIP-Seq design
- GenoGAMSettings object with hdf5Control pointing to an HDF5 storage directory
- Design formula (e.g. ~ s(x) + s(x, by = genotype)) and tile parameters chunkSize = 50000, overhangSize = 1000
- Fixed hyperparameters lambda and theta (e.g. 4601 and 4.51)

## Outputs

- A GenoGAM result object with per-position smooth fits (fits(result)) and standard errors (se(result))
- Position-wise log p-values via computeSignificance(result)
- Runtime and peak-memory measurements per number of parameters per tile for scaling benchmarks

## How to apply

Load the tiled count data with GenoGAMDataSet() using chunkSize = 50000, overhangSize = 1000, a design such as ~ s(x) + s(x, by = genotype), and the HDF5 backend so data are stored on disk out-of-core (hdf5 = TRUE) with low I/O overhead. Compute normalization with computeSizeFactors(), register a single-worker BiocParallel backend if single-core timing is desired, then fit with genogam() holding hyperparameters fixed (lambda = 4601, theta = 4.51) so the Newton-Raphson step uses the SuperLU direct solver on the sparse model matrix instead of dense linear algebra. The rationale is that GenoGAM tiles produce sparse, banded penalty structures, so direct sparse solving avoids the dense matrix multiplications that made GenoGAM 1.0 scale badly; 64-bit Armadillo words (ARMA_64BIT_WORD) may be needed for large matrices. Inspect fits(result), se(result), and optionally computeSignificance(result) for position-wise log p-values; assemble elapsed time and peak memory per parameter count to benchmark scaling.

## Related tools

- **GenoGAM 2.0** (R package providing genogam() fitting with the sparse Newton-Raphson method) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Direct sparse solver used for parameter fitting)
- **HDF5** (Out-of-core on-disk storage backend reducing memory footprint while keeping I/O low)
- **mgcv** (General-purpose R GAM library underlying the model framework and design formulas)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory=folder, chunkSize=50000, overhangSize=1000, design=~ s(x) + s(x, by=genotype), settings=settings, hdf5=TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda=4601, theta=4.51)
```

## Evaluation signals

- At ~1104 parameters, single-core coefficient estimation should complete in on the order of 1 second using ~52 MB, versus ~1.6 h and 8 GB for the dense GenoGAM 1.0 method
- Runtime and memory footprint should scale approximately linearly with the number of parameters per tile
- Speedup over GenoGAM 1.0 should fall in the reported 2-3 orders of magnitude range
- Whole-genome human ChIP-seq fits (~300 million parameters) should be obtainable in under 9 hours on a ~60-core server
- Fits and standard errors (fits(result), se(result)) should agree with dense-solver reference fits at matched lambda and theta

## Limitations

- Requires R 3.4.1 or higher; RcppArmadillo must have ARMA_64BIT_WORD enabled or large-matrix multiplication fails with dimension-incompatibility errors
- HDF5 storage in temporary directories (e.g. tempdir()) risks data loss since contents are deleted later
- Performance claims reference hardware like 'a typical lab server' or 'a standard PC' without exact specifications; single-worker backend is needed for true single-core timing
- The packaged example covers only the first 100 kb of yeast chrI; out-of-bound range warnings during tiling cannot be suppressed in the Snow parallel backend

## Evidence

- [abstract] Sparse solver accelerates fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [abstract] HDF5 reduces memory: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [discussion] Linear scaling finding: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [readme] Fitting invocation: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [readme] 64-bit Armadillo requirement: "enable `ARMA_64BIT_WORD`, which is not enabled in RcppArmadillo by default"
