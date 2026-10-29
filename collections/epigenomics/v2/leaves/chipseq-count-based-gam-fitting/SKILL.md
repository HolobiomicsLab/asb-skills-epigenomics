---
name: chipseq-count-based-gam-fitting
description: Use when modeling ChIP-Seq read-count frequencies along chromosomes as
  smooth functions (e.g. factorial designs with genotype or treatment), or when genome-wide
  base-level and region-level significance testing with data-driven smoothing is required.
when_to_use_negative:
- When the analysis target is RNA-seq or non-genomic count data where the position-based
  GAM design (s(x) along chromosomes) does not apply
- When only ad-hoc window/bin statistics are acceptable and data-driven cross-validated
  smoothing is unnecessary (GenoGAM's advantage is eliminating ad-hoc binning)
- When interactive fitting on very large genomes without HDF5 out-of-core storage
  is attempted — full in-memory fitting will not scale to gigabase-scale genomes
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: GenoGAM 2.0
  role: Scalable R package implementing the sparse, out-of-core GAM fitting used for
    this skill
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Direct sparse solver exploited for parameter fitting to achieve the speedup
- name: HDF5
  role: Out-of-core data storage backend reducing memory footprint while keeping I/O
    low
- name: mgcv
  role: Underlying general-purpose R library for fitting generalized additive models
- name: BiocParallel
  role: Parallel backend registration (SnowParam/MulticoreParam) used to tile fits
    across workers
- name: GenoGAM
  role: Previous Bioconductor version used as baseline for the 2-3 orders of magnitude
    speedup comparison
  repo: gstricker/GenoGAM
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:220afa6134eb752511e95ec6073653180c2b995efdcc6e1620ac28b427beef6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - GenoGAM
  - GenoGAM 2.0
  - HDF5
  - R
  - Rcpp
  - SuperLU
  - mgcv
  - sparseinv
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chipseq-count-based-gam-fitting

## Summary

Fit a genome-wide generalized additive model (GenoGAM 2.0) to ChIP-Seq count data using a tiled overlapping-interval strategy with the fastGenoGAM R package, exploiting model sparsity (SuperLU solver) and HDF5 out-of-core storage to scale to gigabase-scale genomes.

## When to use

Use when modeling ChIP-Seq read-count frequencies along chromosomes as smooth functions (e.g. factorial designs with genotype or treatment), or when genome-wide base-level and region-level significance testing with data-driven smoothing is required.

## When NOT to use

- When the analysis target is RNA-seq or non-genomic count data where the position-based GAM design (s(x) along chromosomes) does not apply
- When only ad-hoc window/bin statistics are acceptable and data-driven cross-validated smoothing is unnecessary (GenoGAM's advantage is eliminating ad-hoc binning)
- When interactive fitting on very large genomes without HDF5 out-of-core storage is attempted — full in-memory fitting will not scale to gigabase-scale genomes

## Inputs

- BAM files plus an experimentDesign.txt file describing the ChIP-Seq factorial design
- GAM formula (e.g. ~ s(x) + s(x, by = genotype))
- Hyperparameters lambda (smoothing penalty, e.g. 4601) and theta (dispersion, e.g. 4.51) if fixed
- Chunk and overhang sizes (e.g. chunkSize = 50000, overhangSize = 1000)
- BiocParallel backend registration (SnowParam or MulticoreParam)

## Outputs

- A GenoGAM object holding tile fits joined at overlap midpoints
- Position-wise fitted smooths via fits(result)
- Position-wise standard errors via se(result)
- Position-wise log p-values via computeSignificance()/pval(result)

## How to apply

Build a GenoGAMDataSet from a BAM-backed experiment design using GenoGAMDataSet() with the HDF5 backend, chunkSize = 50000, overhangSize = 1000, and a design such as ~ s(x) + s(x, by = genotype); the tiling joins overlapping interval fits at overlap midpoints so runtime scales linearly with the number of parameters per tile. Compute normalization with computeSizeFactors(), then fit with genogam() — either estimating hyperparameters or fixing them to save time (e.g. lambda = 4601, theta = 4.51 as in the vignette-scale yeast example on the first 100 kb of chromosome I). Register a BiocParallel backend (SnowParam advised for larger data; MulticoreParam suffices for small data like yeast) before fitting. The fit exploits sparsity via the SuperLU direct solver and stores data out-of-core via HDF5, making it 2 to 3 orders of magnitude faster than GenoGAM 1.0's Newton-Raphson replacement (e.g. 1 s and 52 MB vs 1 h 37 min and 8 GB at 1104 parameters). Validate fits with fits(result) and standard errors with se(result), then compute position-wise log p-values with computeSignificance().

## Related tools

- **GenoGAM 2.0** (Scalable R package implementing the sparse, out-of-core GAM fitting used for this skill) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Direct sparse solver exploited for parameter fitting to achieve the speedup)
- **HDF5** (Out-of-core data storage backend reducing memory footprint while keeping I/O low)
- **mgcv** (Underlying general-purpose R library for fitting generalized additive models)
- **BiocParallel** (Parallel backend registration (SnowParam/MulticoreParam) used to tile fits across workers)
- **GenoGAM** (Previous Bioconductor version used as baseline for the 2-3 orders of magnitude speedup comparison) — gstricker/GenoGAM

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- Runtime and memory of coefficient estimation scale approximately linearly with the number of parameters per tile
- Speedup over GenoGAM 1.0 is in the 2-3 orders of magnitude range (e.g. 1104 parameters: 1 s and 52 MB vs 1 h 37 min and 8 GB)
- Standard errors se(result) are computable position-wise and consistent between the sparse inverse subset and indirect Wood variance methods in shape (though not speed)
- computeSignificance(result) returns valid position-wise log p-values via pval(result) without errors from matrix dimension mismatches
- Whole-genome human-scale fits (~300 million parameters) should complete in under 9 hours on a standard 60-core server

## Limitations

- Requires R 3.4.1 or higher, constraining compatible environments
- Licensed under GPL-2, which may impose redistribution constraints
- Armadillo must use 64-bit words (ARMA_64BIT_WORD) or large matrix multiplication fails with dimension-incompatibility errors on big matrices
- Out-of-bound range warnings during genome tiling cannot be suppressed in the parallel Snow environment
- Performance claims reference generic hardware ('a typical lab server', 'a standard PC') without exact specifications

## Evidence

- [abstract] HDF5 storage and linear scaling: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [abstract] SuperLU sparsity exploit: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [readme] Package workflow: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object. This is done with the HDF5 backend."
- [readme] Fixed hyperparameters invocation: "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [intro] Tiling rationale: "Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints."
