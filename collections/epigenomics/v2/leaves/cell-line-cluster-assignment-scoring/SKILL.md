---
name: cell-line-cluster-assignment-scoring
description: Use when cells from a scATAC-seq or scChIP-seq dataset have known labels
  (cell line, replicate, sample) and clusters computed by ChromSCape must be scored
  for label concordance or compared between samples with Fisher's exact test.
when_to_use_negative:
- Cells have no known label of origin (cell line, replicate, or sample) to validate
  clusters against — the scoring metric is undefined
- The goal is de novo sub-population discovery rather than benchmarking a clustering
  against ground-truth labels
- Input is raw sequencing reads (FASTQ/BAM) rather than a reads-in-peaks count matrix
  — the data-engineering pipeline to generate count matrices must be run first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application used to filter, normalize, run
    PCA (50 PCs), cluster (k=5), and visualize the GSE99172 count matrix prior to
    assignment scoring
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Environment in which the count matrix is loaded, Fisher's exact test is run,
    and metrics are written to CSV
- name: Bioconductor
  role: Distribution channel for the ChromSCape package used for the clustering pipeline
- name: Shiny
  role: Interactive front-end through which the ChromSCape workflow (QC, PCA, clustering,
    visualization) is driven
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:bc1f78b1ec9f4b98294e10fada63a237f87fc539f86e0b9ee06745d98bed0a85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cell-line-cluster-assignment-scoring

## Summary

Quantifies how well unsupervised clusters from ChromSCape correspond to known cell lines of origin by computing per-cell-line replicate assignment scores (e.g. K562 99.7%, GM12878 98.3%) on the GSE99172 scATAC-seq dataset, plus Fisher's exact tests comparing cluster proportions between samples.

## When to use

Use when cells from a scATAC-seq or scChIP-seq dataset have known labels (cell line, replicate, sample) and clusters computed by ChromSCape must be scored for label concordance or compared between samples with Fisher's exact test.

## When NOT to use

- Cells have no known label of origin (cell line, replicate, or sample) to validate clusters against — the scoring metric is undefined
- The goal is de novo sub-population discovery rather than benchmarking a clustering against ground-truth labels
- Input is raw sequencing reads (FASTQ/BAM) rather than a reads-in-peaks count matrix — the data-engineering pipeline to generate count matrices must be run first

## Inputs

- Reads-in-peaks count matrix downloaded as gzipped single-cell BED-derived count data from GEO accession GSE99172
- Cell/sample metadata mapping barcodes to cell lines (TF1, K562, GM12878), technical replicates, or patient samples (SU070, SU353)
- ChromSCape cluster assignments from k=5 clustering on the first 50 principal components

## Outputs

- Per-cell-line replicate assignment scores (expected: K562 ~99.7%, GM12878 ~98.3%)
- Average correct-cluster assignment across TF1/K562/GM12878 (expected ~99.4%)
- Fisher's exact test p-value comparing monocyte-cluster proportions between patient samples (expected ~0.0025)
- Metrics table written to gse99172_chromscape_metrics.csv

## How to apply

Download the reads-in-peaks count matrix for GEO accession GSE99172 and load it in R, then apply ChromSCape QC: a minimum coverage of 1600 unique reads per cell, removal of cells in the top 5% coverage, and keeping regions detected in at least 1% of cells. Normalize the filtered matrix by total read count and region size, run PCA with centering retaining the first 50 PCs, and cluster cells into k=5, visualizing with t-SNE and UMAP. For each cell line (K562, GM12878) compute the fraction of cells assigned to the cluster dominated by that cell line's technical replicates to obtain replicate assignment scores, and compute the average correct-cluster assignment across TF1, K562 and GM12878. For patient-derived samples (e.g. SU070 vs SU353), run a Fisher's exact test on the proportion of cells in the dominant monocyte cluster and sub-cluster it (e.g. C1) if needed. Write all recomputed metrics to a CSV such as gse99172_chromscape_metrics.csv.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application used to filter, normalize, run PCA (50 PCs), cluster (k=5), and visualize the GSE99172 count matrix prior to assignment scoring) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Environment in which the count matrix is loaded, Fisher's exact test is run, and metrics are written to CSV)
- **Bioconductor** (Distribution channel for the ChromSCape package used for the clustering pipeline)
- **Shiny** (Interactive front-end through which the ChromSCape workflow (QC, PCA, clustering, visualization) is driven)

## Examples

```
metrics <- fisher.test(matrix(c(n_mono_SU070, n_other_SU070, n_mono_SU353, n_other_SU353), nrow=2)); write.csv(replicate_scores, "gse99172_chromscape_metrics.csv")
```

## Evaluation signals

- K562 technical-replicate assignment score matches the reported 99.7% and GM12878 matches 98.3% within a small tolerance
- Average correct-cluster assignment across TF1, K562 and GM12878 cells matches the reported 99.4%
- Fisher's exact test p-value comparing monocyte-cluster proportions between SU070 and SU353 matches the reported 0.0025
- All recomputed metrics are present and schema-consistent in gse99172_chromscape_metrics.csv
- t-SNE and UMAP plots on the PCA embedding show clusters 5 and 2 grouping K562 and GM12878 technical replicates respectively

## Limitations

- Results depend on the exact QC thresholds (1600 unique reads minimum per cell, top 5% coverage removal, regions in ≥1% of cells) and k=5 clustering; deviating thresholds will change scores
- The benchmark comparison context is restricted in scope (e.g. subsampled sets such as 1287 cells or 500 MDA-MB-468 cells) in the article's benchmarking, so absolute scores may not generalize to other datasets
- Most R dependency versions are unreported, limiting exact environment reproducibility

## Evidence

- [discussion] a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] GSE99172 count matrix source: "The count matrix of reads in peaks was downloaded from GEO accession number “GSE99172”"
- [discussion] region filtering threshold: "keeping regions detected in at least 1% of cells"
- [discussion] PCA settings: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [readme] benchmarking repo contents: "Scripts and dataset to reproduce benchmarking of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy &amp; scOrange on single-cell H3K27me3 cell mix"
