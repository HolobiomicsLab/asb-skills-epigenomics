---
name: parallel-backend-configuration-genomics
description: Use when benchmarking or running a tiled genome-wide GAM fit (GenoGAM
  2.0) that benefits from per-tile parallelization, and you need a registered BiocParallel
  backend plus a per-stage wall-clock breakdown.
when_to_use_negative:
- Fits restricted to a single small tile (e.g. the package's first 100 kb of yeast
  chrI vignette) where parallel overhead over a few tiles provides no speedup
- Gigabase-scale genomes (e.g. human, ~300 million parameters) where the article recommends
  out-of-core HDF5 processing on a many-core server rather than an in-memory multicore
  desktop run
- When lambda and theta hyperparameters are fixed a priori, since the ~9 min cross-validation
  stage being benchmarked is then skipped
tools:
- name: BiocParallel
  role: Registers the SnowParam parallel backend used to distribute tile fits across
    8 workers during the timing benchmark
- name: GenoGAM 2.0
  role: Genome-wide GAM fitting framework whose preprocessing, cross-validation, and
    fitting stages are parallelized and timed
  repo: https://github.com/gstricker/GenoGAM
- name: SuperLU
  role: Sparse direct solver used in the parameter-fitting stage that is timed as
    part of the runtime breakdown
- name: mgcv
  role: General-purpose R GAM library underlying the model fitting whose performance
    is benchmarked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:16b15c3eb48a1f9be312419a457e5a6796cdeab5f1c1de02ea013fdb9178997d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/GenoGAM
  tools:
  - BiocParallel
  - Bioconductor
  - GenoGAM 2.0
  - R
  - SuperLU
  - mgcv
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# parallel-backend-configuration-genomics

## Summary

Configure a BiocParallel worker backend (e.g. SnowParam with 8 workers) so genome-wide GenoGAM 2.0 tile fits can be timed and benchmarked per pipeline stage (preprocessing, cross-validation, fitting) on a multi-core desktop, with all data kept in memory.

## When to use

Use when benchmarking or running a tiled genome-wide GAM fit (GenoGAM 2.0) that benefits from per-tile parallelization, and you need a registered BiocParallel backend plus a per-stage wall-clock breakdown.

## When NOT to use

- Fits restricted to a single small tile (e.g. the package's first 100 kb of yeast chrI vignette) where parallel overhead over a few tiles provides no speedup
- Gigabase-scale genomes (e.g. human, ~300 million parameters) where the article recommends out-of-core HDF5 processing on a many-core server rather than an in-memory multicore desktop run
- When lambda and theta hyperparameters are fixed a priori, since the ~9 min cross-validation stage being benchmarked is then skipped

## Inputs

- ChIP-seq BAM files (two conditions × two replicates, yeast)
- experiment design file with design ~ s(x) + s(x, by = genotype)
- GenoGAMSettings with chunkSize = 50000, overhangSize = 1000, hdf5 = FALSE
- BiocParallel worker specification (SnowParam(workers = 8))

## Outputs

- GenoGAMDataSet object with computed size factors
- GenoGAM result object from genogam()
- Runtime report table with rows for preprocessing (~40 s), cross-validation (~9 min), fitting (~3.5 min), and total (~13 min)

## How to apply

Register an 8-worker parallel backend with BiocParallel::register(SnowParam(workers = 8)) before building the GenoGAMDataSet, using chunkSize = 50000 and overhangSize = 1000 so the genome is split into overlapping tiles that can be fit independently and joined at overlap midpoints. Set hdf5 = FALSE to process all data in memory (multicore-style backend, no hard drive I/O), which is appropriate for a small genome like yeast; the README notes that 'For yeast MulticoreParam should also do fine' and that SnowParam is advised for larger data. Load the yeast ChIP-seq counts (two conditions × two replicates) via GenoGAMDataSet() with design ~ s(x) + s(x, by = genotype), time computeSizeFactors() (~40 s preprocessing), then time genogam() which internally spends ~9 min on cross-validation estimation of lambda/theta and ~3.5 min on parameter fitting via the SuperLU sparse direct solver. Record the three per-stage wall-clock times plus the total (~13 min) in a runtime report table for benchmarking.

## Related tools

- **BiocParallel** (Registers the SnowParam parallel backend used to distribute tile fits across 8 workers during the timing benchmark)
- **GenoGAM 2.0** (Genome-wide GAM fitting framework whose preprocessing, cross-validation, and fitting stages are parallelized and timed) — https://github.com/gstricker/GenoGAM
- **SuperLU** (Sparse direct solver used in the parameter-fitting stage that is timed as part of the runtime breakdown)
- **mgcv** (General-purpose R GAM library underlying the model fitting whose performance is benchmarked)

## Examples

```
BiocParallel::register(BiocParallel::SnowParam(workers = 8)); ggd <- GenoGAMDataSet(expDesign, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), hdf5 = FALSE)
```

## Evaluation signals

- Total end-to-end runtime on 8 cores falls within the accepted tolerance of ~8–20 min, close to the reported ~13 min
- Runtime report contains rows for all three stages: preprocessing (~40 s), cross-validation (~9 min), and fitting (~3.5 min)
- Backend is registered before GenoGAMDataSet construction and reports 8 workers (SnowParam)
- hdf5 = FALSE is set so no hard-drive I/O occurs during the in-memory benchmark, isolating compute time from disk time
- Genome is tiled with chunkSize = 50000 and overhangSize = 1000, confirming tiles were actually distributed rather than fit as one block

## Limitations

- Performance claims reference hardware ('a typical lab server', 'a standard PC') without fully specifying exact machine specifications
- Suppressing out-of-bound range warnings during tiling does not work reliably in the parallel Snow environment (per README)
- The fastGenoGAM repository version is deprecated in favour of the Bioconductor GenoGAM package; benchmarks must use a consistent version
- RcppArmadillo must have ARMA_64BIT_WORD enabled, otherwise large-matrix multiplication errors occur during fitting

## Evidence

- [readme] SnowParam advised for larger data: "Here we also use the SnowParam backend which is advised for larger data"
- [abstract] sparsity exploited for speed: "This is achieved by exploiting the sparsity of the model using the SuperLU direct solver for parameter fitting"
- [methods] data reading step: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [methods] size factor computation step: "Computing size factors with `computeSizeFactors()`"
- [discussion] linear scaling justification: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
