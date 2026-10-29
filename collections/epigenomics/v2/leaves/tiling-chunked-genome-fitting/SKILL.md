---
name: tiling-chunked-genome-fitting
description: Use when fitting smooth base-resolution models (e.g. GAMs with design
  ~ s(x) + s(x, by = genotype)) across whole genomes from ChIP-seq counts, where genome-wide
  computation must be made tractable via tiling and parallelization.
when_to_use_negative:
- The dataset fits comfortably on a single small interval (e.g. the package's 100
  kb yeast vignette subset) where a single-tile fit with fixed hyperparameters is
  faster and sufficient
- The analysis does not require smooth base-level functions along chromosomes, e.g.
  simple peak calling or region-level counting pipelines without GAM-based significance
  testing
- Memory or hardware constraints rule out both in-memory and HDF5 out-of-core processing
  for very large genomes and only coarse-grained results are needed
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: GenoGAM 2.0
  role: R package providing the tiled genome-wide GAM fit via GenoGAMDataSet(), computeSizeFactors(),
    and genogam()
  repo: https://github.com/gstricker/fastGenoGAM
- name: mgcv
  role: General-purpose R library underlying the GAM fitting framework used by GenoGAM
- name: SuperLU
  role: Sparse direct solver exploited for parameter fitting, giving the 2-3 orders
    of magnitude speed-up
- name: BiocParallel
  role: Registers the parallel backend (SnowParam or MulticoreParam) used to fit tiles
    concurrently
- name: HDF5
  role: Optional on-disk backend that stores data efficiently, reducing memory footprint
    while keeping I/O low
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:3dc9c58b4a4c5eccfee16dcb4ffaaca96f54d95eb7dfcd267fcb18e10b47e14b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - Bioconductor
  - GenoGAM 2.0
  - HDF5
  - R
  - SuperLU
  - mgcv
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tiling-chunked-genome-fitting

## Summary

Fit genome-wide generalized additive models to ChIP-seq read counts by fitting on overlapping genomic tiles (chunkSize = 50000, overhangSize = 1000) and joining tile fits at overlap midpoints, using GenoGAM 2.0's sparse SuperLU solver to make gigabase-scale fits tractable.

## When to use

Use when fitting smooth base-resolution models (e.g. GAMs with design ~ s(x) + s(x, by = genotype)) across whole genomes from ChIP-seq counts, where genome-wide computation must be made tractable via tiling and parallelization.

## When NOT to use

- The dataset fits comfortably on a single small interval (e.g. the package's 100 kb yeast vignette subset) where a single-tile fit with fixed hyperparameters is faster and sufficient
- The analysis does not require smooth base-level functions along chromosomes, e.g. simple peak calling or region-level counting pipelines without GAM-based significance testing
- Memory or hardware constraints rule out both in-memory and HDF5 out-of-core processing for very large genomes and only coarse-grained results are needed

## Inputs

- ChIP-seq BAM files with an experiment design file specifying factorial conditions (e.g. two genotypes × two replicates)
- GAM design formula, e.g. ~ s(x) + s(x, by = genotype)
- chunkSize (50000) and overhangSize (1000) tiling parameters
- HDF5 setting (hdf5 = TRUE for on-disk backend, FALSE for in-memory processing)

## Outputs

- A GenoGAM result object with per-tile smooth fits and standard errors (fits(result), se(result))
- Position-wise log p-values via computeSignificance()
- Estimated smoothing hyperparameters (lambda, theta) when cross-validation is used

## How to apply

Read BAM-derived counts into a GenoGAMDataSet via GenoGAMDataSet() with design ~ s(x) + s(x, by = genotype), chunkSize = 50000 and overhangSize = 1000, choosing hdf5 = FALSE to keep all data in memory or hdf5 = TRUE to store data on hard drive and reduce memory footprint. Register a parallel backend with BiocParallel::register (SnowParam is advised for larger data; MulticoreParam is fine for yeast-size genomes). Compute size factors with computeSizeFactors(), then fit with genogam(), either fixing hyperparameters (e.g. lambda = 4601, theta = 4.51) for speed or leaving them to be estimated by cross-validation (~9 min CV plus ~3.5 min fitting for a genome-wide yeast fit on 8 cores, ~13 min total). Tile fits are joined at overlap midpoints, giving linear computation time and easy parallelization; optionally follow with computeSignificance() for position-wise log p-values.

## Related tools

- **GenoGAM 2.0** (R package providing the tiled genome-wide GAM fit via GenoGAMDataSet(), computeSizeFactors(), and genogam()) — https://github.com/gstricker/fastGenoGAM
- **mgcv** (General-purpose R library underlying the GAM fitting framework used by GenoGAM)
- **SuperLU** (Sparse direct solver exploited for parameter fitting, giving the 2-3 orders of magnitude speed-up)
- **BiocParallel** (Registers the parallel backend (SnowParam or MulticoreParam) used to fit tiles concurrently)
- **HDF5** (Optional on-disk backend that stores data efficiently, reducing memory footprint while keeping I/O low)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), settings = settings, hdf5 = FALSE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd)
```

## Evaluation signals

- Genome-wide yeast fit on an 8-core machine with in-memory data (hdf5 = FALSE) completes in roughly 8-20 minutes (~13 min reported: ~40 s preprocessing, ~9 min cross-validation, ~3.5 min fitting)
- Tile fits are joined at overlap midpoints, so fit curves and standard errors should be continuous across tile boundaries (e.g. inspect the center 10 kb where two tiles join)
- Runtime and memory footprint scale linearly with the number of parameters per tile
- With fixed hyperparameters genogam(ggd, lambda = 4601, theta = 4.51) returns a GenoGAM object whose fits(result) and se(result) are populated for each chromosome
- computeSignificance(result) yields position-wise log p-values accessible via pval(result)

## Limitations

- The original fastGenoGAM repository version is deprecated; users should switch to the Bioconductor/gstricker GenoGAM release
- Out-of-bound range warnings during tile shifting are emitted in the parallel Snow environment and standard suppressWarnings does not silence them
- Large-matrix multiplication errors indicate Armadillo is using 32-bit words; fix requires enabling ARMA_64BIT_WORD in RcppArmadillo
- Requires R 3.4.1 or higher and correct Armadillo dependencies
- Performance claims reference hardware ('a typical lab server', 'a standard PC') without exact specifications, so absolute runtimes vary

## Evidence

- [intro] Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints.: "Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints."
- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [readme] ## specify chunk and overhang size. It is possible to skip this,
## but the automatic specification would prevent us from using
## multiple tiles in such a small example.: "## specify chunk and overhang size. It is possible to skip this, but the automatic specification would prevent us from using multiple tiles in such a small example."
- [readme] Here we also use the SnowParam backend which is advised for larger data
## For yeast MulticoreParam should also do fine: "Here we also use the SnowParam backend which is advised for larger data For yeast MulticoreParam should also do fine"
