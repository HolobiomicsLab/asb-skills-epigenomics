---
name: dimensionality-reduction-umap-visualization
description: Use when analyzing gzipped single-cell BED files from sparse single-cell
  histone-modification assays (e.g. H3K27me3 scCUT&Tag K562/H1, H3K4me3 scChIC-seq)
  and you need a 2D embedding to assess whether known cell populations or expected
  clusters separate.
when_to_use_negative:
- Input is already a low-dimensional embedding or precomputed feature table — PCA/UMAP
  recomputation is redundant
- Data is bulk ChIP-seq; the article states existing bulk ChIP-seq tools are 'not
  fit for the low coverage and sparsity of these single-cell histone modifications
  datasets' and this pipeline targets sparse single-cell assays
- Cells fall below the 1600 unique reads per cell coverage threshold — the projection
  is unreliable on unfiltered, low-coverage cells
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor application that builds the count matrix, performs PCA
    keeping 50 PCs, and applies UMAP/t-SNE on the PCA to visualize the data in two
    dimensions
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: batchelor (FastMNN)
  role: Provides mutual-nearest-neighbor batch correction applied before projecting
    replicate batches (GSE124680/GSE124690) together on the UMAP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:f6dcde7a87bffc16f69309c6aa3af17b77cf743b135a8faf26ab77220f7b2b58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  - batchelor (FastMNN)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# dimensionality-reduction-umap-visualization

## Summary

Visualize single-cell epigenomic datasets (scCUT&Tag, scChIC-seq, scChIP-seq) as a 2D UMAP computed on a PCA embedding of filtered count matrices, optionally after FastMNN batch correction, to reveal sub-populations and verify that known cell types or clusters are recovered.

## When to use

Use when analyzing gzipped single-cell BED files from sparse single-cell histone-modification assays (e.g. H3K27me3 scCUT&Tag K562/H1, H3K4me3 scChIC-seq) and you need a 2D embedding to assess whether known cell populations or expected clusters separate.

## When NOT to use

- Input is already a low-dimensional embedding or precomputed feature table — PCA/UMAP recomputation is redundant
- Data is bulk ChIP-seq; the article states existing bulk ChIP-seq tools are 'not fit for the low coverage and sparsity of these single-cell histone modifications datasets' and this pipeline targets sparse single-cell assays
- Cells fall below the 1600 unique reads per cell coverage threshold — the projection is unreliable on unfiltered, low-coverage cells

## Inputs

- Gzipped single-cell BED files from GEO (e.g. GSE124680, GSE124690, GSE105012)
- Filtered, normalized single-cell count matrices (TSS ±2500bp regions or 50kbp genomic bins)
- PCA embedding (first 50 centered PCs) of the filtered matrix
- Batch labels for replicate merging (e.g. GSE124680 vs GSE124690 K562 replicates)

## Outputs

- 2D UMAP coordinates of single cells computed on the 50-PC PCA embedding
- UMAP plot revealing cluster structure (e.g. K562 vs H1 separation; 7 observable clusters for H3K4me3 scChIC-seq)
- Cluster assignments for downstream ARI computation against ground-truth labels

## How to apply

First build a count matrix in ChromSCape by aggregating signal into regions around gene Transcription Start Sites (±2500bp) or successive 50kbp genomic bins, then apply quality-control filtering (minimum coverage of 1600 unique reads per cell, removal of cells in the top 5% coverage, keep regions detected in at least 1% of cells). Normalize the filtered matrices by total read count and region size, then apply PCA with centering and keep the first 50 PCs for further analysis. If data span multiple batches (e.g. GSE124680 and GSE124690 K562 replicates), apply mutual-nearest-neighbor batch correction with the FastMNN function from the batchelor package before projection. Finally project the data with UMAP (or t-SNE) on the PCA embedding to visualize cells in two dimensions, and interpret cluster structure on the resulting plot. The method relies on UMAP being computed on the reduced 50-PC space, not on the raw sparse matrix.

## Related tools

- **ChromSCape** (Shiny/R Bioconductor application that builds the count matrix, performs PCA keeping 50 PCs, and applies UMAP/t-SNE on the PCA to visualize the data in two dimensions) — https://github.com/vallotlab/ChromSCape_benchmarking
- **batchelor (FastMNN)** (Provides mutual-nearest-neighbor batch correction applied before projecting replicate batches (GSE124680/GSE124690) together on the UMAP)

## Evaluation signals

- ARI between ChromSCape cluster assignments and K562-vs-H1 ground-truth labels equals 0.976 (within rounding tolerance) for the scCUT&Tag H3K27me3 datasets
- scChIC-seq H3K4me3 UMAP clearly shows 7 clusters, matching the original study
- PCA is applied with centering and exactly the first 50 PCs are kept for UMAP projection
- The two K562 scCUT&Tag replicate batches (GSE124680, GSE124690) merge into one group, separate from H1 cells, after FastMNN correction
- Both values (ARI and observed cluster count) are recorded in chromscape_benchmark_validation.csv

## Limitations

- Benchmark comparison was restricted to a subsampled set of 1287 cells (and random sub-sampling of 500 MDA-MB-468 cells), limiting comparison scope
- Benchmark required prior removal of known CNV segments of MDA-MB-468 (77 Mbp), an extra manual curation step
- Cluster counts on the scChIC-seq UMAP are assessed by visually counting observable clusters, a subjective judgment
- A demonstration of ChromSCape is hosted on third-party shinyapps.io, with no guarantee of long-term availability

## Evidence

- [discussion] PCA with 50 kept PCs on normalized, filtered matrices: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] UMAP applied on PCA for 2D visualization: "The t-distributed stochastic neighbor embedding (t-SNE) algorithm23 and UMAP24 is applied on the PCA to visualize the data in two dimensions"
- [discussion] FastMNN batch correction before projection: "A batch correction option using mutual nearest neighbors “FastMNN” function from “batchelor” package16 is implemented"
- [discussion] Count matrix aggregation into bins or TSS regions as UMAP input: "ChromSCape creates the count matrix by aggregating the signal into successive genomic bins, peaks (BED file must be provided by user) or into regions around genes Transcription Start Sites"
- [abstract] ChromSCape identified as a Shiny/R Bioconductor application: "we present ChromSCape, a user-friendly interactive Shiny/R application distributed as a Bioconductor package"
