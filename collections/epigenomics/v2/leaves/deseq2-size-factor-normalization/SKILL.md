---
name: deseq2-size-factor-normalization
description: Use when a GenoGAMDataSet of per-position ChIP-Seq counts (BAM-derived,
  chunkSize 50000) has just been built from multiple samples/conditions and must be
  depth-normalized before calling genogam().
when_to_use_negative:
- Data are single-sample or already depth-normalized by another method, so no cross-sample
  size factors are needed
- Input is raw BAM alignments not yet converted to a tiled GenoGAMDataSet — build
  the dataset with GenoGAMDataSet() first
- The goal is full differential testing with DESeq2 itself rather than normalization
  within a GenoGAM GAM fit workflow
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: GenoGAM 2.0
  role: Provides computeSizeFactors() on GenoGAMDataSet objects as the normalization
    step of the ingestion-to-fit workflow
  repo: https://github.com/gstricker/fastGenoGAM
- name: DESeq2
  role: Origin of the size-factor normalization approach used for sequencing-depth
    correction
- name: BiocParallel
  role: Registers the parallel backend (e.g. SnowParam with 2 or 8 workers) under
    which preprocessing and size-factor computation run
- name: R
  role: Implementation language of the GenoGAM 2.0 package and its computeSizeFactors()
    function
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2238-7
    title: 'GenoGAM 2.0: scalable and efficient implementation of genome-wide generalized
      additive models for gigabase-scale genomes'
  source_content_hash: sha256:15c6b2a5d44048724fdf75e2e5f2432a8c3fe54c84dd6ea60962081798fa59df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gstricker/fastGenoGAM
  tools:
  - BiocParallel
  - Bioconductor
  - DESeq2
  - GenoGAM 2.0
  - GenoGAM 2.0 (fastGenoGAM R package)
  - R
  - mgcv
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# deseq2-size-factor-normalization

## Summary

Computes sequencing-depth normalization size factors for tiled ChIP-Seq count data within the GenoGAM 2.0 workflow, using computeSizeFactors() to attach DESeq2-style size factors to a GenoGAMDataSet so downstream GAM fits compare conditions fairly.

## When to use

Use when a GenoGAMDataSet of per-position ChIP-Seq counts (BAM-derived, chunkSize 50000) has just been built from multiple samples/conditions and must be depth-normalized before calling genogam().

## When NOT to use

- Data are single-sample or already depth-normalized by another method, so no cross-sample size factors are needed
- Input is raw BAM alignments not yet converted to a tiled GenoGAMDataSet — build the dataset with GenoGAMDataSet() first
- The goal is full differential testing with DESeq2 itself rather than normalization within a GenoGAM GAM fit workflow

## Inputs

- GenoGAMDataSet object of tiled per-position ChIP-Seq counts (optionally HDF5-backed) built from BAM files via GenoGAMDataSet() with chunkSize = 50000 and overhangSize = 1000
- Experiment design table (experimentDesign.txt) defining conditions/replicates (e.g. two conditions x two replicates yeast ChIP-seq)

## Outputs

- GenoGAMDataSet with attached DESeq2-style size factors, ready for genogam() model fitting
- Populated size-factor vector accessible via the object's assay/size-factor accessors

## How to apply

After reading BAM alignments with GenoGAMDataSet() (optionally with hdf5 = TRUE and an HDF5 folder set via GenoGAMSettings), call computeSizeFactors() on the GenoGAMDataSet to compute DESeq2-style size factors that correct for sequencing depth variation. This step must run before model fitting with genogam(), since the GAM compares count frequencies across conditions (e.g. genotype groups). In the in-memory workflow (hdf5 = FALSE with an 8-worker SnowParam backend on yeast genome-wide data), preprocessing including size-factor computation took ~40 s of the ~13 min total runtime. Reassign the result back to the object (`ggd <- computeSizeFactors(ggd)`) so the size factors are stored in the object and available to the fit.

## Related tools

- **GenoGAM 2.0** (Provides computeSizeFactors() on GenoGAMDataSet objects as the normalization step of the ingestion-to-fit workflow) — https://github.com/gstricker/fastGenoGAM
- **DESeq2** (Origin of the size-factor normalization approach used for sequencing-depth correction)
- **BiocParallel** (Registers the parallel backend (e.g. SnowParam with 2 or 8 workers) under which preprocessing and size-factor computation run)
- **R** (Implementation language of the GenoGAM 2.0 package and its computeSizeFactors() function)

## Examples

```
ggd <- GenoGAMDataSet(expDesign, directory = folder, chunkSize = 50000, overhangSize = 1000, design = design, settings = settings, hdf5 = TRUE); ggd <- computeSizeFactors(ggd)
```

## Evaluation signals

- computeSizeFactors() returns a GenoGAMDataSet (reassigned via ggd <- computeSizeFactors(ggd)) with size factors populated rather than NULL/empty
- Size factors vary across replicates in a way consistent with sequencing depth (multiplicative correction near 1 for similar-depth libraries)
- Downstream genogam() fit accepts the object without warnings about missing size factors
- In the yeast benchmark, the preprocessing stage including size-factor computation completes in ~40 s on an 8-core in-memory configuration
- Assay values are unchanged by the call — only the normalization metadata is added

## Limitations

- Requires R 3.4.1 or higher, constraining compatible environments
- Out-of-bound range warnings can appear during the parallel tiling step preceding normalization and are documented as ignorable, but suppressWarnings does not work reliably in the parallel Snow environment
- HDF5-backed workflows need a writable, persistent HDF5 directory — using /tmp causes data and results to be deleted later

## Evidence

- [methods] Computing size factors with `computeSizeFactors()`: "Computing size factors with `computeSizeFactors()`"
- [other] During preprocessing, the fragments are centered, the coverage y is computed, the data is split by genomic tiles, and normalization factors for sequencing depth variation are computed using DESeq2.: [evidence span withheld — anchor status: not_found]
- [readme] ggd <- computeSizeFactors(ggd): "## compute size factors
ggd <- computeSizeFactors(ggd)"
- [results] ~40 s data processing of the ~13 min genome-wide yeast runtime: [evidence span withheld — anchor status: not_found]
