---
name: genomic-scaling-benchmark-profiling
description: Use when evaluating whether a tiled/smoothed genome-wide model implementation's
  runtime and memory footprint scale linearly with parameters per tile, or when reproducing
  version-to-version speedup benchmarks (e.g. 2-3 orders of magnitude).
when_to_use_negative:
- The goal is statistical validation of fits (p-values, differential occupancy) rather
  than runtime/memory scaling — use computeSignificance()/plotGenoGAM workflows instead
- The analysis requires parallel, genome-wide production fitting (e.g. ~300 million
  human parameters on 60 cores) — this skill measures single-core scaling only
- Smoothing parameter estimation by cross-validation must be performed — fix lambda
  and theta instead, as parameter estimation would confound the timing
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: GenoGAM 2.0
  role: R package whose Newton-Raphson coefficient estimation is benchmarked for runtime
    and memory scaling
  repo: https://github.com/gstricker/fastGenoGAM
- name: SuperLU
  role: Sparse direct solver exploited by GenoGAM 2.0 for parameter fitting, enabling
    the observed speedup
- name: HDF5
  role: Out-of-core on-disk data storage backend (hdf5 = TRUE) that reduces memory
    footprint while keeping I/O low during fits
- name: R
  role: Implementation language; the benchmark is run as R code using BiocParallel
    for single-worker registration
- name: mgcv
  role: Underlying mgcv GAM framework on which GenoGAM's model fitting builds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:0b791e1386649c47c976ef58a4b5fd9e8aa7c579fd5403b8c449e2b8fab7e527
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# genomic-scaling-benchmark-profiling

## Summary

A benchmarking skill for measuring how single-core coefficient-estimation runtime and peak memory scale with the number of model parameters, as done for GenoGAM 2.0 (Newton-Raphson + SuperLU, HDF5 out-of-core) versus GenoGAM 1.0 (Fig. 2).

## When to use

Use when evaluating whether a tiled/smoothed genome-wide model implementation's runtime and memory footprint scale linearly with parameters per tile, or when reproducing version-to-version speedup benchmarks (e.g. 2-3 orders of magnitude).

## When NOT to use

- The goal is statistical validation of fits (p-values, differential occupancy) rather than runtime/memory scaling — use computeSignificance()/plotGenoGAM workflows instead
- The analysis requires parallel, genome-wide production fitting (e.g. ~300 million human parameters on 60 cores) — this skill measures single-core scaling only
- Smoothing parameter estimation by cross-validation must be performed — fix lambda and theta instead, as parameter estimation would confound the timing

## Inputs

- Packaged yeast ChIP-seq example (first 100 kb of chrI) with experimentDesign.txt and BAM files
- GenoGAMSettings with hdf5Control directory for HDF5 out-of-core storage
- Fixed hyperparameters lambda = 4601 and theta = 4.51
- Sequence of increasing model parameter counts (design ~ s(x) + s(x, by = genotype))
- v1.0 reference measurements for comparison (1104 parameters: 1 h 37 min, 8 GB)

## Outputs

- Table of elapsed single-core fitting time versus number of parameters per tile (Fig. 2a)
- Table of peak memory usage versus number of parameters per tile (Fig. 2b)
- Speedup ratio of GenoGAM 2.0 vs GenoGAM 1.0 at the last comparable benchmark point

## How to apply

Load a packaged example dataset (first 100 kb of yeast chrI) via GenoGAMDataSet() with chunkSize = 50000, overhangSize = 1000, design ~ s(x) + s(x, by = genotype), and hdf5 = TRUE so data is stored out-of-core via the HDF5 backend. Compute size factors with computeSizeFactors(), then repeatedly call genogam() with fixed hyperparameters (lambda = 4601, theta = 4.51) at increasing model parameter counts, registering a single-worker BiocParallel backend (e.g. SnowParam(worker = 1)) so timings reflect single-core coefficient estimation. Record elapsed fitting time and peak memory per fit, and where feasible observe the SuperLU sparse direct solver's behavior. Assemble a runtime/memory-versus-parameter-count table and compare against v1.0 reference measurements (1104 parameters: 1 h 37 min and 8 GB vs 1 s and 52 MB for v2.0).

## Related tools

- **GenoGAM 2.0** (R package whose Newton-Raphson coefficient estimation is benchmarked for runtime and memory scaling) — https://github.com/gstricker/fastGenoGAM
- **SuperLU** (Sparse direct solver exploited by GenoGAM 2.0 for parameter fitting, enabling the observed speedup)
- **HDF5** (Out-of-core on-disk data storage backend (hdf5 = TRUE) that reduces memory footprint while keeping I/O low during fits)
- **R** (Implementation language; the benchmark is run as R code using BiocParallel for single-worker registration)
- **mgcv** (Underlying mgcv GAM framework on which GenoGAM's model fitting builds)

## Examples

```
library(fastGenoGAM); ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd); result <- genogam(ggd, lambda = 4601, theta = 4.51)
```

## Evaluation signals

- At the last comparable point of 1104 parameters, GenoGAM 2.0 should take about 1 second and ~52 MB versus GenoGAM 1.0's 1 h 37 min and 8 GB (a 2-3 orders of magnitude speedup)
- Runtime and peak memory should scale approximately linearly with the number of parameters per tile
- Timing must be single-core: verify the BiocParallel backend is registered with a single worker (e.g. SnowParam(worker = 1))
- Fits must use fixed hyperparameters lambda = 4601, theta = 4.51 so that timing reflects coefficient estimation, not smoothing-parameter cross-validation
- HDF5 backend must be active (hdf5 = TRUE) so memory measurements reflect out-of-core processing

## Limitations

- Performance claims reference hardware ('a typical lab server', 'a standard PC') without specifying exact specifications, so absolute numbers are machine-dependent
- Comparison points between versions are limited; 1104 parameters is only the last comparable benchmark point
- Large-matrix fitting can fail with Armadillo 32-bit matrix errors (e.g. 'incompatible matrix dimensions'); ARMA_64BIT_WORD must be enabled in RcppArmadillo
- fastGenoGAM (the benchmarked repo) is deprecated; the maintained version is gstricker/GenoGAM on Bioconductor
- Requires R 3.4.1 or higher; licensed under GPL-2; no DOI/Zenodo archival record is provided

## Evidence

- [abstract] Here we present GenoGAM 2.0, a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version.: "Here we present GenoGAM 2.0, a scalable and efficient implementation that is 2 to 3 orders of magnitude faster than the previous version."
- [abstract] This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] result <- genogam(ggd, lambda = 4601, theta = 4.51): "result <- genogam(ggd, lambda = 4601, theta = 4.51)"
- [methods] Computing size factors with `computeSizeFactors()`: "Computing size factors with `computeSizeFactors()`"
- [discussion] We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
- [readme] The example dataset that is part of the package is restricted to the first 100kb of the first yeast chromosome.: [evidence span withheld — anchor status: external_doc]
