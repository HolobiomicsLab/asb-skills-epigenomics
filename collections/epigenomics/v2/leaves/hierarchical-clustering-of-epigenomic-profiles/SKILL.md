---
name: hierarchical-clustering-of-epigenomic-profiles
description: Use when a sparse single-cell count matrix (scChIP-seq/scATAC-seq bins
  or peaks) with known or hypothesized sub-populations must be grouped via hierarchical
  clustering on a PCA/feature-space embedding and quantitatively benchmarked by ARI.
when_to_use_negative:
- Input is already a reduced feature/embedding matrix from a method other than those
  being benchmarked and no ground-truth labels exist for ARI computation
- The dataset is bulk ChIP-seq or lacks single-cell resolution — the article notes
  existing bulk ChIP-seq tools are not fit for these sparse single-cell data, and
  vice versa this benchmark targets single-cell profiles
- The goal is differential enrichment or gene/pathway interpretation downstream of
  known clusters rather than cluster discovery or method benchmarking
edam_operation: http://edamontology.org/operation_0407
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3169
tools:
- name: ChromSCape
  role: Shiny/R Bioconductor app that QC-filters, normalizes and PCA-reduces (50 PCs)
    the single-cell count matrix before hierarchical clustering of sub-populations
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: SnapATAC
  role: Comparator method producing a reduced feature space from 50 kbp genomic bins
    for the same clustering/ARI evaluation
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: CisTopic
  role: Comparator method producing a topic-model feature space on pseudo-bulk peaks
    (merged_peaks.5000.bed) for clustering and ARI scoring
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: Cusanovich2018
  role: Comparator method producing its feature embedding on the same count matrix
    for clustering and ARI scoring
  repo: https://github.com/pinellolab/scATAC-benchmarking
- name: EpiScanpy
  role: Python/Scanpy-based comparator run following its scATAC-seq tutorial to generate
    the feature space prior to clustering and ARI computation
  repo: https://github.com/colomemaria/epiScanpy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:78a881e376faf6502e96c4b9218755cf1a2c0c701d990bfc0df1f48bdec69740
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - CisTopic
  - Cusanovich2018
  - EpiScanpy
  - R
  - SnapATAC
  edam_operation: http://edamontology.org/operation_0407
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# hierarchical-clustering-of-epigenomic-profiles

## Summary

Clusters single-cell epigenomic profiles (e.g., an in-silico H3K27me3 scChIP-seq mix of Jurkat, Ramos, HBCx-22 and MDA-MB-468) by applying hierarchical clustering on each method's reduced feature space (e.g., ChromSCape's 50 retained PCs) and scoring separation against known cell-line labels with the adjusted Rand index (ARI).

## When to use

Use when a sparse single-cell count matrix (scChIP-seq/scATAC-seq bins or peaks) with known or hypothesized sub-populations must be grouped via hierarchical clustering on a PCA/feature-space embedding and quantitatively benchmarked by ARI.

## When NOT to use

- Input is already a reduced feature/embedding matrix from a method other than those being benchmarked and no ground-truth labels exist for ARI computation
- The dataset is bulk ChIP-seq or lacks single-cell resolution — the article notes existing bulk ChIP-seq tools are not fit for these sparse single-cell data, and vice versa this benchmark targets single-cell profiles
- The goal is differential enrichment or gene/pathway interpretation downstream of known clusters rather than cluster discovery or method benchmarking

## Inputs

- Normalized scChIP-seq H3K27me3 count matrix in 50 kbp genomic bins for an in-silico mix of Jurkat, Ramos, HBCx-22 and MDA-MB-468 cells (GSE117309, GSE152502)
- Reduced feature spaces (e.g., 50 PCA components from ChromSCape; topic/latent feature matrices from CisTopic, Cusanovich2018, SnapATAC, EpiScanpy)
- Ground-truth per-cell cell-line labels for the 1287 QC-passing cells

## Outputs

- Hierarchical clustering assignments (sub-population labels per cell) for each method's reduced feature space
- Adjusted Rand index values per method, saved to ari_results.csv
- Benchmarking figures (e.g., Figures 2a and 2b) produced by cross_analysis.R

## How to apply

First reduce the normalized count matrix to a low-dimensional feature space: for ChromSCape, apply PCA with centering on the QC-filtered, count-normalized matrix and keep the first 50 PCs; for comparators (SnapATAC on 50 kbp bins, CisTopic, Cusanovich2018, EpiScanpy), run each tool's reference pipeline from the scATAC-benchmarking scripts. Apply hierarchical clustering to the reduced feature space of every method. Compute the adjusted Rand index of the resulting partitions against the ground-truth cell-line labels (the 1287-cell in-silico mix passing QC thresholds). Accept the result when each recomputed ARI matches the reported value within ~±0.05 (ChromSCape ≈0.998, Cusanovich2018 ≈0.996, CisTopic ≈0.996, EpiScanpy ≈0.940, SnapATAC ≈0.822).

## Related tools

- **ChromSCape** (Shiny/R Bioconductor app that QC-filters, normalizes and PCA-reduces (50 PCs) the single-cell count matrix before hierarchical clustering of sub-populations) — https://github.com/vallotlab/ChromSCape_benchmarking
- **SnapATAC** (Comparator method producing a reduced feature space from 50 kbp genomic bins for the same clustering/ARI evaluation) — https://github.com/pinellolab/scATAC-benchmarking
- **CisTopic** (Comparator method producing a topic-model feature space on pseudo-bulk peaks (merged_peaks.5000.bed) for clustering and ARI scoring) — https://github.com/pinellolab/scATAC-benchmarking
- **Cusanovich2018** (Comparator method producing its feature embedding on the same count matrix for clustering and ARI scoring) — https://github.com/pinellolab/scATAC-benchmarking
- **EpiScanpy** (Python/Scanpy-based comparator run following its scATAC-seq tutorial to generate the feature space prior to clustering and ARI computation) — https://github.com/colomemaria/epiScanpy

## Examples

```
Rscript ChromSCape_benchmarking/cross_analysis.R
```

## Evaluation signals

- Recomputed ARI for ChromSCape on the in-silico H3K27me3 mix is ≈0.998 (tolerance ±0.05)
- Comparator ARIs fall near their reported values: Cusanovich2018 ≈0.996, CisTopic ≈0.996, EpiScanpy ≈0.940, SnapATAC (50 kbp bins) ≈0.822
- Clustering recovers four cell-type partitions matching the Jurkat/Ramos/HBCx-22/MDA-MB-468 labels
- Input restricted to the 1287 cells passing QC thresholds (minimum 1600 unique reads per cell, top 5% coverage cells removed)
- ari_results.csv contains exactly five ARI values, one per benchmarked method

## Limitations

- Benchmark restricted to a subsampled set of 1287 cells (with 500 MDA-MB-468 cells randomly subsampled), limiting comparison scope
- Required prior removal of known CNV segments of MDA-MB-468 (77 Mbp), an extra manual curation step
- SnapATAC was run on 50 kbp bins, and its ARI (≈0.822) is configuration-dependent; large transitory objects for CisTopic and SnapATAC had to be stored as RData rather than regenerated
- Cluster quality depends on upstream QC/normalization choices (min read count threshold, region filtering at ≥1% of cells, top 5% coverage removal)

## Evidence

- [discussion] PCA keeps 50 PCs for clustering: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] Common QC'd input across methods: "a set of 1287 cells passing the quality control thresholds were selected to be used as input for all methods"
- [readme] Benchmark scripts produce feature spaces from the H3K27me3 mix: "produce the reduced feature space of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy from count H3K27me3 scChIP-seq data"
- [readme] cross_analysis.R generates benchmark figures: "The ‘cross_analysis.R’ will produce the Figures 2a and b in the ‘Cross_analysis’ folder."
- [discussion] ChromSCape performs as well or better than state of the art: "We show that ChromSCape performs as well or better than state of the art single-cell epigenomic analytic tools"
