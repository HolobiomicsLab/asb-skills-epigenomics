---
name: genomic-count-matrix-construction
description: Use when raw ChIP-Seq BAM alignments plus an experiment design file must
  be converted into a tiled, normalized count matrix too large for memory, as preprocessing
  before GAM model fitting.
when_to_use_negative:
- Data are already a normalized count matrix or feature table and only model fitting
  (genogam()) or significance testing is needed
- The dataset is small enough to fit in memory and HDF5 out-of-core overhead is unnecessary
  (e.g. MulticoreParam suffices for yeast-scale data)
- The analysis requires peak calling rather than smooth-function modeling of count
  frequencies along chromosomes
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0 (fastGenoGAM R package)
  role: Provides GenoGAMDataSet() and computeSizeFactors() to build the tiled HDF5-backed
    count matrix from BAM alignments
  repo: https://github.com/gstricker/fastGenoGAM
- name: HDF5
  role: On-disk backend storing tiled counts efficiently, reducing memory footprint
    while keeping I/O low
- name: R
  role: Implementation language for the GenoGAM package (requires R 3.4.1 or higher)
- name: Bioconductor
  role: Infrastructure (BiocParallel) on which GenoGAM builds and through which the
    parallel SnowParam backend is registered
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:bfdea4488774e708680638f7fcaa8d073b54a2da55a197cc838abbac4b7823f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - Bioconductor
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM R package)
  - HDF5
  - R
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47619047619047616
n_claims: 21
---

# genomic-count-matrix-construction

## Summary

Builds an out-of-core, HDF5-backed genomic count matrix from raw ChIP-Seq BAM alignments using GenoGAM 2.0: reads are converted to per-position counts, split into genomic tiles (chunkSize/overhangSize), and DESeq2-style size factors are attached for sequencing-depth normalization.

## When to use

Use when raw ChIP-Seq BAM alignments plus an experiment design file must be converted into a tiled, normalized count matrix too large for memory, as preprocessing before GAM model fitting.

## When NOT to use

- Data are already a normalized count matrix or feature table and only model fitting (genogam()) or significance testing is needed
- The dataset is small enough to fit in memory and HDF5 out-of-core overhead is unnecessary (e.g. MulticoreParam suffices for yeast-scale data)
- The analysis requires peak calling rather than smooth-function modeling of count frequencies along chromosomes

## Inputs

- ChIP-Seq BAM alignment files organized in a bam directory
- experimentDesign.txt file describing the factorial design and sample-to-BAM mapping
- GenoGAMSettings object with hdf5Control directory for the HDF5 backend
- Design formula (e.g. ~ s(x) + s(x, by = genotype)), chunkSize (50000), and overhangSize (1000)

## Outputs

- HDF5-backed GenoGAMDataSet object (is.HDF5 true) holding tiled per-position count data
- GenoGAMDataSet with DESeq2-style size factors populated after computeSizeFactors()
- Count matrix retrievable via assay(ggd)

## How to apply

Configure GenoGAMSettings to point the HDF5 store at a working directory, register a BiocParallel backend (SnowParam with 2 workers as in the README), and set chunkSize = 50000 and overhangSize = 1000 along with the GAM design formula ~ s(x) + s(x, by = genotype). Call GenoGAMDataSet() on the experimentDesign.txt and the BAM directory (hdf5 = TRUE) so alignments are read, fragments centered, coverage computed, and per-position counts written to disk in overlapping genomic tiles — this out-of-core HDF5 design keeps the memory footprint low while I/O stays manageable. Then run computeSizeFactors() on the resulting GenoGAMDataSet to attach DESeq2-style normalization factors accounting for sequencing-depth variation. Warnings about out-of-bound ranges during the tiling/shifting process can be safely ignored because ranges are trimmed accordingly. The result is a size-factor-annotated tiled count matrix ready for genogam() model fitting.

## Related tools

- **GenoGAM 2.0 (fastGenoGAM R package)** (Provides GenoGAMDataSet() and computeSizeFactors() to build the tiled HDF5-backed count matrix from BAM alignments) — https://github.com/gstricker/fastGenoGAM
- **HDF5** (On-disk backend storing tiled counts efficiently, reducing memory footprint while keeping I/O low)
- **R** (Implementation language for the GenoGAM package (requires R 3.4.1 or higher))
- **Bioconductor** (Infrastructure (BiocParallel) on which GenoGAM builds and through which the parallel SnowParam backend is registered)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd)
```

## Evaluation signals

- is.HDF5 on the returned GenoGAMDataSet is TRUE, confirming the object is disk-backed rather than in-memory
- assay(ggd) returns a count matrix with dimensions matching the tiled genome regions (e.g. first 100kb of yeast chromosome I) and all samples
- Size factor slots are populated (non-empty, finite values) after computeSizeFactors() runs
- Out-of-bound range warnings occurring during the tiling/shifting process do not correspond to missing or corrupted tiles after trimming
- Tiling respects the configured chunkSize = 50000 and overhangSize = 1000 so overlapping tiles exist and can be joined at overlap midpoints

## Limitations

- Requires R 3.4.1 or higher, constraining compatible environments
- Armadillo matrix errors on large data if ARMA_64BIT_WORD is not enabled in RcppArmadillo
- Out-of-bound range warnings during the tiling/shifting process cannot easily be suppressed under the parallel Snow backend
- Do not use /tmp for the HDF5 folder in persistent workflows because data and results get deleted later

## Evidence

- [methods] Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [methods] Computing size factors with `computeSizeFactors()`: "Computing size factors with `computeSizeFactors()`"
- [abstract] the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low.: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [readme] Warnings about out-of-bound ranges can be ignored, as they occur during the shifting process when the genome gets tiled.: "Warnings about out-of-bound ranges can be ignored, as they occur during the shifting process when the genome gets tiled."
- [other] During preprocessing, the fragments are centered, the coverage y is computed, the data is split by genomic tiles, and normalization factors ... computed using DESeq2.: [evidence span withheld — anchor status: not_found]
