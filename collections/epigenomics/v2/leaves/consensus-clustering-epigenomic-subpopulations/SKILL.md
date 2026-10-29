---
name: consensus-clustering-epigenomic-subpopulations
description: Use when sparse single-cell histone-modification count matrices from
  multiple samples must be partitioned into reproducible epigenomic subpopulations
  with per-cluster differentially enriched regions and consensus-score stability checks.
when_to_use_negative:
- Input is a scRNA-seq or scATAC-seq feature table rather than a sparse single-cell
  histone-modification count matrix
- Input data are already aggregated pseudo-bulk profiles and no single-cell resolution
  subpopulation discovery is required
- Only a generic clustering of an existing dimensionality-reduced embedding is needed
  without QC, correlation filtering, or differential enrichment
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2229
tools:
- name: ChromSCape
  role: 'End-to-end Shiny/R Bioconductor application: count matrix ingestion, QC,
    correlation filtering, normalization, PCA/t-SNE/UMAP, consensus clustering, and
    differential enrichment analysis of the scChIP-seq data'
- name: Shiny
  role: Interactive user interface layer through which the QC thresholds, clustering,
    and visualization steps are driven
- name: R
  role: Implementation language for ChromSCape and the downstream clustering/differential-enrichment
    scripts
- name: Bioconductor
  role: Distribution channel for the ChromSCape package used to run the analysis
- name: ConsensusClusterPlus
  role: Consensus clustering of the n=903 filtered cells at k=4 with computation of
    mean consensus scores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:94da751d6f6c621fc281184ba5065dd0e779d511c81344094ff8001dbe5730e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - ChromSCape
  - ConsensusClusterPlus
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# consensus-clustering-epigenomic-subpopulations

## Summary

Deconvolve heterogeneous single-cell H3K27me3 scChIP-seq count matrices (e.g., four PDX breast tumor models) into stable chromatin-based subpopulations using ChromSCape's QC, correlation filtering, PCA/t-SNE/UMAP, and ConsensusClusterPlus consensus clustering, followed by per-cluster differential enrichment.

## When to use

Use when sparse single-cell histone-modification count matrices from multiple samples must be partitioned into reproducible epigenomic subpopulations with per-cluster differentially enriched regions and consensus-score stability checks.

## When NOT to use

- Input is a scRNA-seq or scATAC-seq feature table rather than a sparse single-cell histone-modification count matrix
- Input data are already aggregated pseudo-bulk profiles and no single-cell resolution subpopulation discovery is required
- Only a generic clustering of an existing dimensionality-reduced embedding is needed without QC, correlation filtering, or differential enrichment

## Inputs

- H3K27me3 single-cell count matrices (scChIP-seq) for the four PDX models HBCx-22, HBCx-22-TamR, HBCx-95, HBCx-95-CapaR
- Data deposit: figshare project 'Single-Cell ChIP-seq of Mouse Stromal Cells in PDX tumour models of resistance' (project 66419)
- Count matrices in 50kbp genomic bins (per README, loadable directly in ChromSCape)
- Optional user-provided BED file of regions to exclude (e.g., CNV regions)

## Outputs

- cluster_assignments.csv
- consensus_scores.csv
- differential_enriched_regions_per_cluster.csv
- t-SNE/UMAP two-dimensional embeddings of filtered cells

## How to apply

Load the H3K27me3 single-cell count matrices into ChromSCape (Shiny/R, Bioconductor) and apply cell-level QC: retain cells with a minimum coverage of 1600 unique reads and remove cells in the top 5% coverage percentile. Apply correlation-based filtering, removing cells whose Pearson pairwise correlation score falls below a threshold t with at least p% of cells (p set at 1% by default), and keep regions detected in at least 1% of cells. Normalize the filtered matrix by total read count and region size, run PCA with centering keeping the first 50 PCs, visualize with t-SNE/UMAP, and use FastMNN batch correction for multi-sample data. Cluster the filtered cells (n=903 in the reference analysis) with ConsensusClusterPlus at k=4 and compute mean consensus scores, then run per-cluster differential enrichment analysis and tally differentially enriched regions per cluster. In the reference run, selected clusters achieved a mean consensus score of 0.91 versus 0.17 for other pairs, and 189/210/83/9 depleted regions were found for clusters C1-C4, with C3 composed almost exclusively of HBCx-95 (TNBC) cells.

## Related tools

- **ChromSCape** (End-to-end Shiny/R Bioconductor application: count matrix ingestion, QC, correlation filtering, normalization, PCA/t-SNE/UMAP, consensus clustering, and differential enrichment analysis of the scChIP-seq data)
- **Shiny** (Interactive user interface layer through which the QC thresholds, clustering, and visualization steps are driven)
- **R** (Implementation language for ChromSCape and the downstream clustering/differential-enrichment scripts)
- **Bioconductor** (Distribution channel for the ChromSCape package used to run the analysis)
- **ConsensusClusterPlus** (Consensus clustering of the n=903 filtered cells at k=4 with computation of mean consensus scores)

## Evaluation signals

- Cell set after QC and correlation filtering reaches the expected n=903 cells (min 1600 unique reads, top 5% coverage removed, correlation threshold with p=1%)
- Mean consensus score for selected k=4 clusters approximates 0.91, versus 0.17 for other cluster pairs
- Per-cluster differentially enriched (depleted) region counts match 189/210/83/9 for C1-C4
- Cluster C3 is composed almost exclusively of HBCx-95 (TNBC) cells, revealing the model-specific stromal population
- Output files cluster_assignments.csv, consensus_scores.csv, and differential_enriched_regions_per_cluster.csv are produced with consistent cell/cluster identifiers

## Limitations

- Existing bulk ChIP-seq analysis tools are not fit for the low coverage and sparsity of these single-cell histone modification datasets, so thresholds must be tuned carefully
- ChromSCape depends on MACS2 for the peak-calling module, but no MACS2 version is specified; most R dependency versions are unreported, limiting environment reproducibility
- FastMNN batch correction is an optional step; applying it incorrectly to single-sample data or mis-set thresholds can shift cluster composition away from the reference result

## Evidence

- [discussion] QC thresholds (1600 reads / top 5%): "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] Correlation-based cell filtering: "cells displaying a Pearson’s pairwise correlation score below a threshold t with at least p% of cells are filtered out (p is set at 1% by default)"
- [discussion] PCA parameterization: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] Batch correction option for multi-sample TME data: "A batch correction option using mutual nearest neighbors “FastMNN” function from “batchelor” package16 is implemented"
- [abstract] ChromSCape deployment: "we present ChromSCape, a user-friendly interactive Shiny/R application distributed as a Bioconductor package"
- [readme] Count matrix ingestion format: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’"
