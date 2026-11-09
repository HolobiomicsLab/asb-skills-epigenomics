---
name: hdf5-out-of-memory-data-handling
description: Use when fitting genome-wide generalized additive models on gigabase-scale
  ChIP-Seq data where the count matrix exceeds available RAM, and out-of-core processing
  with low I/O overhead is required.
when_to_use_negative:
- Datasets small enough to fit comfortably in memory (e.g. the packaged 100 kb yeast
  chrI example used only for vignettes) where HDF5 setup adds overhead without benefit
- Analyses requiring interactive reuse of intermediate results stored under a temporary
  HDF5 directory, since data in tempdir() is deleted later
- Fitting methods that require dense in-memory matrices and cannot exploit model sparsity,
  where the out-of-core HDF5 design gives no speedup
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: HDF5
  role: On-disk storage library used as the backend of GenoGAM 2.0 to store data efficiently
    on hard drive, reducing memory footprint while keeping I/O low
- name: GenoGAM 2.0
  role: R/Bioconductor package implementing out-of-core genome-wide GAM fitting via
    GenoGAMDataSet(hdf5 = TRUE), chunk/overhang tiling, and SuperLU-based sparse parameter
    fitting
  repo: https://github.com/gstricker/GenoGAM
- name: fastGenoGAM
  role: Pre-release GenoGAM 2.0 package whose quick guide demonstrates the HDF5-backed
    workflow (GenoGAMSettings, GenoGAMDataSet, computeSizeFactors, genogam)
  repo: https://github.com/gstricker/fastGenoGAM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:72039b969446fe56511b1135181078a81c9da521f091e7a5bbf5cc12b89fef22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/GenoGAM
  tools:
  - GenoGAM 2.0
  - HDF5
  - R
  - fastGenoGAM
  - mgcv
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# HDF5 out-of-memory data handling

## Summary

Stores large genomic count matrices on disk via the HDF5 backend of the GenoGAM 2.0 R package, so that genome-wide GAM fits operate out-of-core instead of holding all data in RAM, reducing the memory footprint while keeping I/O low.

## When to use

Use when fitting genome-wide generalized additive models on gigabase-scale ChIP-Seq data where the count matrix exceeds available RAM, and out-of-core processing with low I/O overhead is required.

## When NOT to use

- Datasets small enough to fit comfortably in memory (e.g. the packaged 100 kb yeast chrI example used only for vignettes) where HDF5 setup adds overhead without benefit
- Analyses requiring interactive reuse of intermediate results stored under a temporary HDF5 directory, since data in tempdir() is deleted later
- Fitting methods that require dense in-memory matrices and cannot exploit model sparsity, where the out-of-core HDF5 design gives no speedup

## Inputs

- ChIP-Seq BAM files referenced by an experimentDesign.txt file
- Experiment design path plus GAM design formula (e.g. ~ s(x) + s(x, by = genotype))
- chunkSize (50000) and overhangSize (1000) tiling parameters
- HDF5 folder specification via GenoGAMSettings

## Outputs

- GenoGAMDataSet object backed by on-disk HDF5 storage
- Fitted GenoGAM object with per-tile smooth fits and standard errors
- Position-wise log p-values via computeSignificance()

## How to apply

Configure an HDF5 storage directory via GenoGAMSettings(hdf5Control = list(dir = ...)) and avoid /tmp since results persist there; then call GenoGAMDataSet() with hdf5 = TRUE, chunkSize = 50000 and overhangSize = 1000 to tile the genome into overlapping intervals whose data are written to HDF5 on hard drive. Size factors are computed with computeSizeFactors() before fitting. The model is then fit with genogam() (optionally fixing lambda = 4601 and theta = 4.51) reading data out-of-core from HDF5, exploiting model sparsity with the SuperLU direct solver for parameter fitting. Rationale: the tiling approach on overlapping genomic intervals joined at overlap midpoints keeps runtime and memory scaling linear with the number of parameters per tile, and HDF5 keeps the memory footprint small while I/O stays low.

## Related tools

- **HDF5** (On-disk storage library used as the backend of GenoGAM 2.0 to store data efficiently on hard drive, reducing memory footprint while keeping I/O low)
- **GenoGAM 2.0** (R/Bioconductor package implementing out-of-core genome-wide GAM fitting via GenoGAMDataSet(hdf5 = TRUE), chunk/overhang tiling, and SuperLU-based sparse parameter fitting) — https://github.com/gstricker/GenoGAM
- **fastGenoGAM** (Pre-release GenoGAM 2.0 package whose quick guide demonstrates the HDF5-backed workflow (GenoGAMSettings, GenoGAMDataSet, computeSizeFactors, genogam)) — https://github.com/gstricker/fastGenoGAM

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE)
```

## Evaluation signals

- The GenoGAMDataSet object is successfully created with hdf5 = TRUE and no 'incompatible matrix dimensions' error from Armadillo 32-bit word limits (fix with #define ARMA_64BIT_WORD 1 if it appears)
- Peak memory during genogam() fitting stays far below the in-memory v1.0 reference (e.g. ~52 MB vs 8 GB at 1104 parameters) and scales linearly with parameters per tile
- Chunk/overhang tiling produces multiple tiles with fits joined at overlap midpoints (check the 10 kb center region around positions 45000:55000 for continuity)
- fits(result) and se(result) return per-tile smooth terms s(x) and s(x):genotype with finite standard errors
- HDF5 files are written to the configured hdf5Control directory rather than a transient location that gets deleted

## Limitations

- Data stored in a tempdir()-based HDF5 folder is deleted afterwards; use a persistent directory for results you want to keep
- Large-matrix errors can occur if RcppArmadillo is compiled with 32-bit words; ARMA_64BIT_WORD must be enabled during compilation
- Warnings about out-of-bound ranges during the genome tiling/shifting step cannot be silenced with suppressWarnings in the parallel Snow environment
- Performance claims reference hardware ('a typical lab server', 'a standard PC') without specifying exact specifications

## Evidence

- [abstract] HDF5 reduces memory footprint: "the HDF5 library is employed to store data efficiently on hard drive, reducing memory footprint while keeping I/O low."
- [intro] out-of-core data processing: "This is achieved by exploiting the sparsity of the model and by using out-of-core data processing."
- [readme] GenoGAMDataSet HDF5 backend: "Reading in data through `GenoGAMDataSet()` to get a `GenoGAMDataSet` object. This is done with the HDF5 backend."
- [readme] HDF5 folder caveat: [evidence span withheld — anchor status: external_doc]
- [discussion] linear scaling of runtime and memory: "We showed its runtime and memory footprint to scale linearly with the number of parameters per tiles"
