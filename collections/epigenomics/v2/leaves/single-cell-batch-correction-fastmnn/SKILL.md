---
name: single-cell-batch-correction-fastmnn
description: Use when single-cell histone-modification datasets (e.g. scCUT&Tag GSE124680
  vs GSE124690 replicates) contain distinct technical batches that must be merged
  before UMAP projection or clustering without losing true K562-vs-H1 biological separation.
when_to_use_negative:
- The dataset contains only a single batch/replicate — mutual nearest neighbor correction
  has nothing to merge and can distort the embedding.
- Batch is fully confounded with biology (each batch contains a distinct cell type),
  so correction would remove the very population differences (e.g. K562 vs H1) you
  want to recover.
- Input is raw unfiltered reads or an unnormalized count matrix — normalization and
  centered PCA (50 PCs) must precede FastMNN in the ChromSCape workflow.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor app that exposes FastMNN batch correction as a built-in
    option within its QC → normalization → PCA → visualization workflow
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: batchelor
  role: Bioconductor R package supplying the mutual nearest neighbors fastMNN function
    used to merge replicate batches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:1b6a352526587b67d017147e45216f889f24b1c57fad674fc0e3525c5f9d58b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  - batchelor
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-batch-correction-fastmnn

## Summary

Merge single-cell epigenomic batches (e.g. two K562 scCUT&Tag replicates) by applying FastMNN mutual-nearest-neighbors batch correction, as implemented in ChromSCape via the batchelor R package, on a centered PCA embedding of normalized H3K27me3/H3K4me3 count matrices.

## When to use

Use when single-cell histone-modification datasets (e.g. scCUT&Tag GSE124680 vs GSE124690 replicates) contain distinct technical batches that must be merged before UMAP projection or clustering without losing true K562-vs-H1 biological separation.

## When NOT to use

- The dataset contains only a single batch/replicate — mutual nearest neighbor correction has nothing to merge and can distort the embedding.
- Batch is fully confounded with biology (each batch contains a distinct cell type), so correction would remove the very population differences (e.g. K562 vs H1) you want to recover.
- Input is raw unfiltered reads or an unnormalized count matrix — normalization and centered PCA (50 PCs) must precede FastMNN in the ChromSCape workflow.

## Inputs

- Per-batch single-cell count matrices (BED-derived, TSS ±2500bp or 50kbp binned), QC-filtered and normalized
- PCA embedding (first 50 centered PCs) per batch
- Batch labels identifying each replicate (e.g. GSE124680, GSE124690)
- Ground-truth cell labels (K562 vs H1) for ARI validation

## Outputs

- Batch-corrected low-dimensional embedding (FastMNN-corrected PCs)
- UMAP projection with batches merged
- Cluster assignments comparable to ground truth (ARI vs K562/H1 labels)

## How to apply

First build the per-batch count matrices in ChromSCape (R/Bioconductor Shiny app), aggregating signal into TSS ±2500bp regions or 50kbp genomic bins from gzipped single-cell BED files. Apply QC (minimum 1600 unique reads per cell, removal of top 5% coverage cells, keep regions detected in ≥1% of cells), then normalize by total read count and region size and run centered PCA keeping the first 50 PCs. Apply the FastMNN function from the batchelor package to the PCA embedding with batch labels identifying each replicate (e.g. GSE124680 K562 vs GSE124690 K562/H1). Project the corrected embedding with UMAP and verify that replicate batches co-mingle while true cell populations (K562 vs H1) remain separated, quantified by Adjusted Rand Index (target ARI = 0.976 in the source evaluation).

## Related tools

- **ChromSCape** (Shiny/R Bioconductor app that exposes FastMNN batch correction as a built-in option within its QC → normalization → PCA → visualization workflow) — https://github.com/vallotlab/ChromSCape_benchmarking
- **batchelor** (Bioconductor R package supplying the mutual nearest neighbors fastMNN function used to merge replicate batches)

## Examples

```
corrected <- batchelor::fastMNN(pca_embedding, batch = batch_labels)
```

## Evaluation signals

- Adjusted Rand Index between corrected-embedding cluster assignments and K562-vs-H1 ground-truth labels equals 0.976 (within rounding tolerance).
- UMAP of the corrected embedding shows the two K562 replicate batches (GSE124680/GSE124690) intermixed rather than forming batch-specific islands.
- Biological separation is preserved: K562 cells cluster apart from H1 cells after correction.
- The correction was applied to a centered PCA embedding limited to the first 50 PCs of matrices normalized by total read count and region size.
- For the scChIC-seq H3K4me3 dataset, the corrected UMAP reproduces the 7 clusters reported in the original study.

## Limitations

- Batch correction requires explicit batch labels and is exposed in ChromSCape only as an optional post-PCA step; mislabeled batches will drive erroneous merging.
- Benchmarking in the article was restricted to a subsampled set of 1287 cells (and a random 500-cell MDA-MB-468 subsample), so correction performance on larger cohorts is not established here.
- The benchmark required prior manual removal of known MDA-MB-468 CNV segments (77 Mbp), an extra curation step not handled by the batch-correction option itself.

## Evidence

- [discussion] FastMNN on centered 50-PC embedding: "A batch correction option using mutual nearest neighbors “FastMNN” function from “batchelor” package16 is implemented"
- [discussion] PCA precedes downstream analysis: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [other] Source-card application to K562 replicates: [evidence span withheld — anchor status: not_found]
