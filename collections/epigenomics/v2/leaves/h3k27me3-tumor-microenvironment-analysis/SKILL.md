---
name: h3k27me3-tumor-microenvironment-analysis
description: Use when analyzing sparse single-cell H3K27me3 scChIP-seq count matrices
  from mixed PDX tumor micro-environment samples to identify chromatin-defined subpopulations
  and subtype-specific stromal cells.
when_to_use_negative:
- Input is already a clustered or annotated feature/assignment table rather than raw
  per-cell count matrices
- The data are scRNA-seq or scATAC-seq count matrices — this skill is specific to
  sparse single-cell histone modification (H3K27me3) scChIP-seq data
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application used for QC, correlation filtering,
    normalization, PCA, t-SNE/UMAP, consensus clustering, and differential enrichment
    analysis of the H3K27me3 scChIP-seq matrices
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: ConsensusClusterPlus
  role: Consensus clustering of the 903 filtered cells at k=4 and computation of mean
    consensus scores
- name: R
  role: Language in which the ChromSCape analysis and downstream CSV outputs are generated
- name: Shiny
  role: User interface through which the step-by-step ChromSCape workflow is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:d8dc71419908baf4de4ceb6e779c68ddf9d63cf3b31a4779afe186855c09c4e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - ConsensusClusterPlus
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# h3k27me3-tumor-microenvironment-analysis

## Summary

Reproduce the ChromSCape deconvolution of H3K27me3 scChIP-seq data from four mouse breast tumor PDX models (HBCx-22, HBCx-22-TamR, HBCx-95, HBCx-95-CapaR) into stable chromatin-based cell populations, using k=4 ConsensusClusterPlus clustering and per-cluster differential enrichment.

## When to use

Use when analyzing sparse single-cell H3K27me3 scChIP-seq count matrices from mixed PDX tumor micro-environment samples to identify chromatin-defined subpopulations and subtype-specific stromal cells.

## When NOT to use

- Input is already a clustered or annotated feature/assignment table rather than raw per-cell count matrices
- The data are scRNA-seq or scATAC-seq count matrices — this skill is specific to sparse single-cell histone modification (H3K27me3) scChIP-seq data

## Inputs

- Four H3K27me3 single-cell ChIP-seq count matrices (HBCx-22, HBCx-22-TamR, HBCx-95, HBCx-95-CapaR) from figshare project 66419
- BED-format regions optionally excluded (e.g., CNV regions)

## Outputs

- cluster_assignments.csv
- consensus_scores.csv
- differential_enriched_regions_per_cluster.csv
- t-SNE/UMAP two-dimensional embeddings of filtered cells

## How to apply

Download the four H3K27me3 single-cell count matrices from the figshare project 'Single-Cell ChIP-seq of Mouse Stromal Cells in PDX tumour models of resistance' (project 66419) and load them into ChromSCape (Bioconductor/Shiny/R). Apply cell-level QC keeping cells with at least 1600 unique reads, removing the top 5% coverage percentile, then apply correlation-based filtering (Pearson pairwise correlation below threshold with at least 1% of cells) and keep regions detected in at least 1% of cells to retain the n=903 cell set. Normalize by total read count and region size, run centered PCA keeping the first 50 PCs, visualize with t-SNE/UMAP, and use FastMNN batch correction for the multi-sample data. Cluster the 903 cells with ConsensusClusterPlus at k=4, compute mean consensus scores, and run per-cluster differential enrichment analysis, expecting cluster C3 to be composed almost exclusively of HBCx-95 (TNBC) cells.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application used for QC, correlation filtering, normalization, PCA, t-SNE/UMAP, consensus clustering, and differential enrichment analysis of the H3K27me3 scChIP-seq matrices) — https://github.com/vallotlab/ChromSCape_benchmarking
- **ConsensusClusterPlus** (Consensus clustering of the 903 filtered cells at k=4 and computation of mean consensus scores)
- **R** (Language in which the ChromSCape analysis and downstream CSV outputs are generated)
- **Shiny** (User interface through which the step-by-step ChromSCape workflow is executed)

## Examples

```
Rscript -e "library(ChromSCape); launchApp()"
```

## Evaluation signals

- Mean consensus score for selected clusters should reach ~0.91 versus ~0.17 for non-selected clusters at k=4
- Cell count after QC and correlation filtering should be n=903
- Per-cluster differentially enriched (depleted) region counts should be 189/210/83/9 for clusters C1–C4
- Cluster C3 should be composed almost exclusively of HBCx-95 (TNBC) cells, revealing the stromal population specific to that model
- Output files cluster_assignments.csv, consensus_scores.csv, and differential_enriched_regions_per_cluster.csv must be produced with consistent cell IDs

## Limitations

- Benchmark comparison was restricted to a subsampled set of 1287 cells and random sub-sampling of 500 MDA-MB-468 cells, a potentially limiting comparison scope
- Existing tools for bulk ChIP-seq are not fit for the low coverage and sparsity of single-cell histone modification datasets, so filtering thresholds are critical
- Most R dependency versions are unreported, limiting environment reproducibility; the pipeline relies on raw fastq processing (PCR, RT and window duplicate removal) via the scChIPseq_DataEngineering pipeline

## Evidence

- [abstract] we deconvolve chromatin landscapes within the tumor micro-environment, identifying distinct H3K27me3 landscapes associated with cell identity and breast tumor subtype: "we deconvolve chromatin landscapes within the tumor micro-environment, identifying distinct H3K27me3 landscapes associated with cell identity and breast tumor subtype"
- [methods] Minimum coverage of 1600 unique reads and top 5% removal: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [methods] Correlation-based cell filtering with p=1% default: "cells displaying a Pearson’s pairwise correlation score below a threshold t with at least p% of cells are filtered out (p is set at 1% by default)"
- [results] PCA with centering keeping first 50 PCs: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] Matrices availability on figshare: "(matrices available at https://figshare.com/projects/Single-Cell_ChIP-seq_of_Mouse_Stromal_Cells_in_PDX_tumour_models_of_resistance/66419)"
- [readme] Count matrices in 50kbp bins input to ChromSCape: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape"
