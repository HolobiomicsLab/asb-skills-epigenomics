---
name: chipseq-read-count-ingestion-genomic-intervals
description: Use when raw ChIP-seq BAM files from a factorial design experiment (e.g.
  two conditions × two replicates) must be converted into position-indexed read counts
  over overlapping genomic tiles as input to a GenoGAM model fit.
when_to_use_negative:
- The data are already a GenoGAMDataSet or pre-binned count matrix — re-ingestion
  would duplicate the tiling step
- The analysis requires ad-hoc fixed binning/windowing rather than data-driven smoothing
  over overlapping tiles
- Non-count genomic data (e.g. continuous tracks) that do not originate from BAM read
  alignments
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: GenoGAM 2.0
  role: R package whose GenoGAMDataSet() constructor reads BAM files, tiles the genome,
    and produces the count data object
  repo: https://github.com/gstricker/fastGenoGAM
- name: Bioconductor
  role: Infrastructure layer on which GenoGAM builds, including BiocParallel for the
    parallel backend used during data ingestion
- name: HDF5
  role: Optional backend for storing count data on hard drive to reduce memory footprint
    while keeping I/O low
- name: R
  role: Implementation language for the GenoGAM package and the ingestion workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:11ae717071415f699895886968a90f8daec17acc6d3f5bebdfd6bba9a74be75e
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
  - mgcv
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# chipseq-read-count-ingestion-genomic-intervals

## Summary

Ingest ChIP-seq BAM files into tiled genomic read-count intervals with GenoGAMDataSet() using a factorial design (~ s(x) + s(x, by = genotype)), chunkSize = 50000 and overhangSize = 1000, choosing between in-memory (hdf5 = FALSE) and HDF5-backed storage before fitting a genome-wide GAM.

## When to use

Use when raw ChIP-seq BAM files from a factorial design experiment (e.g. two conditions × two replicates) must be converted into position-indexed read counts over overlapping genomic tiles as input to a GenoGAM model fit.

## When NOT to use

- The data are already a GenoGAMDataSet or pre-binned count matrix — re-ingestion would duplicate the tiling step
- The analysis requires ad-hoc fixed binning/windowing rather than data-driven smoothing over overlapping tiles
- Non-count genomic data (e.g. continuous tracks) that do not originate from BAM read alignments

## Inputs

- ChIP-seq BAM alignment files (e.g. two conditions × two replicates for yeast)
- experimentDesign.txt file mapping samples to BAM paths and the model design (~ s(x) + s(x, by = genotype))
- GenoGAMSettings object specifying HDF5 directory when using the HDF5 backend

## Outputs

- A GenoGAMDataSet object holding tiled read counts over overlapping genomic intervals (chunkSize 50000, overhang 1000)
- Optionally an on-disk HDF5 store of the count data when hdf5 = TRUE

## How to apply

Build an experiment design file (BAM directory plus design ~ s(x) + s(x, by = genotype), where x is position) and call GenoGAMDataSet() with chunkSize = 50000 and overhangSize = 1000; the genome is automatically tiled into overlapping intervals that are later joined at overlap midpoints. Set hdf5 = FALSE to keep all data in memory (as done for the 8-core yeast runtime benchmark) or hdf5 = TRUE with GenoGAMSettings(hdf5Control = list(dir = ...)) to reduce memory footprint via the HDF5 backend. Register a BiocParallel backend (e.g. SnowParam(workers = 8) for an 8-core machine; for yeast MulticoreParam also suffices) before constructing the object so tiling/read-in parallelizes. Warnings about out-of-bound ranges during the shifting process are expected and can be ignored because the ranges are trimmed accordingly. After ingestion, compute normalization with computeSizeFactors() before passing the GenoGAMDataSet to genogam().

## Related tools

- **GenoGAM 2.0** (R package whose GenoGAMDataSet() constructor reads BAM files, tiles the genome, and produces the count data object) — https://github.com/gstricker/fastGenoGAM
- **Bioconductor** (Infrastructure layer on which GenoGAM builds, including BiocParallel for the parallel backend used during data ingestion)
- **HDF5** (Optional backend for storing count data on hard drive to reduce memory footprint while keeping I/O low)
- **R** (Implementation language for the GenoGAM package and the ingestion workflow)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = ~ s(x) + s(x, by = genotype), settings = settings, hdf5 = FALSE); ggd <- computeSizeFactors(ggd)
```

## Evaluation signals

- The returned object is a valid GenoGAMDataSet whose tiling reflects chunkSize = 50000 with overhangSize = 1000 (overlapping intervals joined at overlap midpoints).
- When hdf5 = FALSE all data live in memory; when hdf5 = TRUE the HDF5 directory contains the stored count data and memory footprint is reduced.
- For the yeast benchmark configuration (8 cores, in-memory), preprocessing via GenoGAMDataSet + computeSizeFactors completes in roughly the reported ~40 s.
- Out-of-bound range warnings may appear during tiling; they should correspond only to the shifting/trimming process, not to data corruption.
- Downstream genogam() fitting accepts the object without dimension errors (e.g. the RcppArmadillo ARMA_64BIT_WORD issue indicates malformed large-matrix handling).

## Limitations

- The example dataset shipped with the package covers only the first 100 kb of yeast chromosome I; genome-wide ingestion requires user-supplied BAM files.
- Out-of-bound range warnings during the tile-shifting step cannot easily be silenced (suppressWarnings does not work in the parallel Snow environment).
- Large matrix multiplication errors can occur if Armadillo uses 32-bit words; ARMA_64BIT_WORD must be enabled in RcppArmadillo.
- The fastGenoGAM repository version is deprecated in favor of the Bioconductor GenoGAM release.

## Evidence

- [readme] Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object.: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object."
- [abstract] HDF5 reduces memory footprint: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [intro] Tiling approach for genome-wide fits: "Genome-wide fits were obtained by fitting models on tiles, defined as overlapping genomic intervals of a tractable size, and joining together tile fits at overlap midpoints."
- [readme] Warnings during tiling can be ignored: "Warnings about out-of-bound ranges can be ignored, as they occur during the shifting process when the genome gets tiled. It is trimmed accordingly."
- [readme] chunkSize and overhangSize specification: "chunkSize <- 50000
overhangSize <- 1000"
