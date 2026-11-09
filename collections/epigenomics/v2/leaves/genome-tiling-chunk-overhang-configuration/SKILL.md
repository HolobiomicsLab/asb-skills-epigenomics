---
name: genome-tiling-chunk-overhang-configuration
description: Use when preparing a ChIP-Seq count dataset for genome-wide GAM fitting
  with GenoGAM, i.e. when BAM alignments must be tiled into overlapping intervals
  of fixed chunkSize/overhangSize and written to an HDF5 backend.
when_to_use_negative:
- When the task is model fitting itself rather than data ingestion — use genogam()
  on an already constructed GenoGAMDataSet
- When the input is not raw ChIP-Seq alignments requiring tiling (e.g. a fitted GenoGAM
  object or a count matrix already loaded in memory)
- When a genome is too small to require out-of-core HDF5 storage and in-memory processing
  suffices
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0 (fastGenoGAM R package)
  role: Provides GenoGAMSettings, GenoGAMDataSet() and the chunk/overhang tiling mechanism
    used to build the HDF5-backed tiled count dataset
  repo: https://github.com/gstricker/fastGenoGAM
- name: HDF5
  role: Out-of-core backend that stores the tiled count data on hard drive, reducing
    memory footprint while keeping I/O low
- name: BiocParallel
  role: Registers the SnowParam parallel backend (2 workers) over which tile-wise
    processing runs
- name: R
  role: Language in which the package and the configuration workflow are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:62a8c6efee3d4cc3b0f23f970dc33df6c236bfceff630dec90f6b95aca95183c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM R package)
  - HDF5
  - R
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# genome-tiling-chunk-overhang-configuration

## Summary

Configure GenoGAM 2.0's out-of-core genome tiling by setting GenoGAMSettings with an HDF5 directory, registering a BiocParallel backend, and supplying chunkSize/overhangSize plus a GAM design formula so that genome-wide fits are computed on overlapping genomic tiles.

## When to use

Use when preparing a ChIP-Seq count dataset for genome-wide GAM fitting with GenoGAM, i.e. when BAM alignments must be tiled into overlapping intervals of fixed chunkSize/overhangSize and written to an HDF5 backend.

## When NOT to use

- When the task is model fitting itself rather than data ingestion — use genogam() on an already constructed GenoGAMDataSet
- When the input is not raw ChIP-Seq alignments requiring tiling (e.g. a fitted GenoGAM object or a count matrix already loaded in memory)
- When a genome is too small to require out-of-core HDF5 storage and in-memory processing suffices

## Inputs

- experimentDesign.txt experiment design file
- Directory of BAM alignment files (e.g. Set1/bam restricted to the first 100kb of yeast chromosome I)
- GAM design formula (~ s(x) + s(x, by = genotype))
- chunkSize and overhangSize integer parameters

## Outputs

- HDF5-backed GenoGAMDataSet object containing tiled per-position counts
- GenoGAMSettings object pointing at the HDF5 store

## How to apply

Point GenoGAMSettings at a persistent HDF5 working directory (avoid /tmp) and register a BiocParallel backend, preferring SnowParam for large data (e.g. SnowParam(worker = 2)). Set chunkSize and overhangSize explicitly — the README example uses chunkSize = 50000 and overhangSize = 1000 — because automatic specification may prevent multiple tiles in small examples. Provide the GAM design formula (e.g. ~ s(x) + s(x, by = genotype)) reflecting the factorial experiment. Then call GenoGAMDataSet() with hdf5 = TRUE on the experimentDesign.txt and BAM directory; out-of-bound range warnings during the tile-shifting process can be ignored as ranges are trimmed accordingly. The rationale is that genome-wide fits are obtained by fitting models on overlapping genomic intervals and joining tile fits at overlap midpoints, enabling linear computation time and parallelization.

## Related tools

- **GenoGAM 2.0 (fastGenoGAM R package)** (Provides GenoGAMSettings, GenoGAMDataSet() and the chunk/overhang tiling mechanism used to build the HDF5-backed tiled count dataset) — https://github.com/gstricker/fastGenoGAM
- **HDF5** (Out-of-core backend that stores the tiled count data on hard drive, reducing memory footprint while keeping I/O low)
- **BiocParallel** (Registers the SnowParam parallel backend (2 workers) over which tile-wise processing runs)
- **R** (Language in which the package and the configuration workflow are implemented)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE)
```

## Evaluation signals

- is.HDF5(ggd) returns TRUE, confirming the dataset is backed by the HDF5 store
- assay(ggd) returns the stored count matrix with per-position counts across the tiled ranges
- Tiling succeeds with multiple tiles — e.g. in the 100kb yeast example chunkSize = 50000 yields two tiles joined around the 50kb midpoint
- Out-of-bound range warnings occur during tile shifting and are trimmed (expected behavior, not an error)
- computeSizeFactors(ggd) subsequently attaches DESeq2-style size factors to the object without error

## Limitations

- Out-of-bound range warnings during the shifting/tiling process cannot be silenced via suppressWarnings in the parallel Snow environment
- Automatic chunk/overhang specification may prevent multiple tiles for small inputs, so parameters must be set manually
- Avoid /tmp for the HDF5 folder because all data and results are deleted later
- Requires R 3.4.1 or higher; large matrix operations need ARMA_64BIT_WORD enabled in RcppArmadillo to avoid dimension errors

## Evidence

- [intro] Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints.: "Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints."
- [readme] chunkSize <- 50000; overhangSize <- 1000: "chunkSize <- 50000
overhangSize <- 1000"
- [readme] Warnings about out-of-bound ranges can be ignored, as they occur during the shifting process when the genome gets tiled. It is trimmed accordingly.: "Warnings about out-of-bound ranges can be ignored, as they occur during the shifting process when the genome gets tiled. It is trimmed accordingly."
- [abstract] the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low.: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [methods] Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
