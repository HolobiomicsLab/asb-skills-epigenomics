---
name: hdf5-backed-genomic-data-import
description: Use when raw ChIP-Seq BAM alignments and an experimentDesign.txt must
  be converted into tiled count data for GenoGAM model fitting and the dataset is
  too large to hold in memory, requiring the HDF5 backend.
when_to_use_negative:
- Input is not raw alignment data — if a GenoGAMDataSet or fitted GenoGAM object already
  exists, do not re-run ingestion.
- Small in-memory datasets where the HDF5 out-of-core overhead is unnecessary and
  default in-memory storage suffices.
- Analyses that are not GenoGAM-based (e.g. peak calling with MACS2 or differential
  tools outside the GenoGAM framework).
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: GenoGAM 2.0 (fastGenoGAM)
  role: R package providing GenoGAMDataSet() for HDF5-backed read-in of BAM alignments
    and computeSizeFactors() for DESeq2-style normalization
  repo: https://github.com/gstricker/fastGenoGAM
- name: HDF5
  role: On-disk storage backend used by GenoGAMDataSet(hdf5 = TRUE) to keep the memory
    footprint low while keeping I/O low
- name: R
  role: Runtime environment for the GenoGAM package workflow (R 3.4.1 or higher required)
- name: Bioconductor
  role: Infrastructure framework on which GenoGAM builds; BiocParallel SnowParam backend
    registers workers for parallel read-in
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:aeec274fc20b80f7b7862f94e7fa2b52d46982a5f367da615d25c5db61c4415f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - Bioconductor
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM R package)
  - GenoGAM 2.0 (fastGenoGAM)
  - HDF5
  - R
  - mgcv
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# hdf5-backed-genomic-data-import

## Summary

Import raw ChIP-Seq BAM alignments into a GenoGAMDataSet object with the HDF5 out-of-core backend, producing tiled per-position count data stored on disk to keep the memory footprint low. This is the first ingestion stage of the GenoGAM 2.0 workflow for genome-wide GAM fitting.

## When to use

Use when raw ChIP-Seq BAM alignments and an experimentDesign.txt must be converted into tiled count data for GenoGAM model fitting and the dataset is too large to hold in memory, requiring the HDF5 backend.

## When NOT to use

- Input is not raw alignment data — if a GenoGAMDataSet or fitted GenoGAM object already exists, do not re-run ingestion.
- Small in-memory datasets where the HDF5 out-of-core overhead is unnecessary and default in-memory storage suffices.
- Analyses that are not GenoGAM-based (e.g. peak calling with MACS2 or differential tools outside the GenoGAM framework).

## Inputs

- ChIP-Seq BAM files (e.g. yeast Set1 bam directory or ENCODE CEBPB/CTCF/USF1/MAX/PAX5/YY1 chromosome 22 experiments)
- experimentDesign.txt file mapping samples to conditions
- GenoGAMSettings with hdf5Control directory and parallel backend configuration
- chunkSize and overhangSize parameters (e.g. 50000 and 1000)

## Outputs

- HDF5-backed GenoGAMDataSet object containing tiled per-position count data
- GenoGAMDataSet with DESeq2-style normalization size factors attached after computeSizeFactors()

## How to apply

Set up a GenoGAMSettings pointing the HDF5 store at a working directory (GenoGAMSettings(hdf5Control = list(dir = ...))), register a BiocParallel SnowParam backend (e.g. 2 workers), and set chunkSize = 50000, overhangSize = 1000 with the design ~ s(x) + s(x, by = genotype). Call GenoGAMDataSet() on the experiment design file and BAM directory (optionally restricting input, e.g. to the first 100kb of yeast chromosome I) with hdf5 = TRUE; the function reads alignments, computes per-position counts, and writes tiled counts to the HDF5 backend. Fragment centering, coverage computation, tiling, and DESeq2-based normalization happen at this ingestion stage. Immediately follow with computeSizeFactors() to attach size factors before model fitting. Out-of-bound range warnings emitted during genome tiling/shifting can be safely ignored since the ranges are trimmed accordingly.

## Related tools

- **GenoGAM 2.0 (fastGenoGAM)** (R package providing GenoGAMDataSet() for HDF5-backed read-in of BAM alignments and computeSizeFactors() for DESeq2-style normalization) — https://github.com/gstricker/fastGenoGAM
- **HDF5** (On-disk storage backend used by GenoGAMDataSet(hdf5 = TRUE) to keep the memory footprint low while keeping I/O low)
- **R** (Runtime environment for the GenoGAM package workflow (R 3.4.1 or higher required))
- **Bioconductor** (Infrastructure framework on which GenoGAM builds; BiocParallel SnowParam backend registers workers for parallel read-in)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd)
```

## Evaluation signals

- The returned object is HDF5-backed: is.HDF5 evaluates to TRUE.
- assay(ggd) returns the stored count matrix and size factors are populated after computeSizeFactors().
- Out-of-bound range warnings during read-in occur during the shifting/tiling process and are trimmed, so they can be ignored.
- The GAM design attribute on the object is ~ s(x) + s(x, by = genotype) and the object carries chunkSize 50000 / overhangSize 1000 tiling parameters.
- HDF5 files are written to the directory specified in GenoGAMSettings(hdf5Control = list(dir = ...)).

## Limitations

- Warnings about out-of-bound ranges are emitted during the shifting process when the genome gets tiled and cannot be silenced by suppressWarnings in the parallel Snow environment; they must be ignored manually.
- Requires R 3.4.1 or higher.
- The automatic chunk/overhang specification may prevent multiple tiles on small regions, so chunkSize = 50000 and overhangSize = 1000 should be set explicitly for small example data.
- Avoid using /tmp for the HDF5 store because stored data and results get deleted later.

## Evidence

- [abstract] the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low.: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [methods] Reading in data through GenoGAMDataSet() to get a GenoGAMDataSet object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [methods] Computing size factors with computeSizeFactors(): "Computing size factors with `computeSizeFactors()`"
- [other] During preprocessing, the fragments are centered, the coverage y is computed, the data is split by genomic tiles, and normalization factors ... are computed using DESeq2.: [evidence span withheld — anchor status: not_found]
- [readme] Warnings about out-of-bound ranges can be ignored, as they occur during the shifting process when the genome gets tiled.: "Warnings about out-of-bound ranges can be ignored, as they occur during the 
shifting process when the genome gets tiled."
