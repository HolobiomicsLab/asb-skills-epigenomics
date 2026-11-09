---
name: subcluster-identification-scatac-seq
description: Use when a reads-in-peaks scATAC-seq count matrix (e.g. GEO GSE99172)
  must be partitioned into sub-populations that can be traced back to cell lines,
  replicates, or patient-derived samples like SU070/SU353.
when_to_use_negative:
- Input is already a reduced feature/embedding space from another featurization method
  (e.g. SnapATAC, CisTopic, Cusanovich2018 feature matrices) rather than a raw reads-in-peaks
  count matrix.
- Data are bulk epigenomic samples — the article notes existing bulk ChIP-seq tools
  were the unfit baseline; this skill targets sparse single-cell datasets.
- The goal is differential enrichment or pathway interpretation only, without needing
  to first identify sub-populations.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_3934
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application used for QC filtering, normalization,
    PCA (50 PCs), clustering, t-SNE/UMAP visualization, and sub-population identification
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Language in which the count matrix is loaded, filtered, clustered, and metrics
    computed
- name: Bioconductor
  role: Distribution channel of the ChromSCape package
- name: Shiny
  role: Interactive graphical interface through which sub-populations are identified
    and visualized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:4f648ce44dd9dd71db7c04531d8224d550f15ad1c501632853ed9cae04e4660d
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
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# subcluster-identification-scatac-seq

## Summary

Identify cell sub-populations in single-cell ATAC-seq (and other sparse single-cell epigenomic) data by QC-filtering reads-in-peaks matrices, PCA with centering (50 PCs), clustering (e.g. k=5), and t-SNE/UMAP visualization with ChromSCape, then validate sub-population structure against known labels with assignment scores and Fisher's exact tests.

## When to use

Use when a reads-in-peaks scATAC-seq count matrix (e.g. GEO GSE99172) must be partitioned into sub-populations that can be traced back to cell lines, replicates, or patient-derived samples like SU070/SU353.

## When NOT to use

- Input is already a reduced feature/embedding space from another featurization method (e.g. SnapATAC, CisTopic, Cusanovich2018 feature matrices) rather than a raw reads-in-peaks count matrix.
- Data are bulk epigenomic samples — the article notes existing bulk ChIP-seq tools were the unfit baseline; this skill targets sparse single-cell datasets.
- The goal is differential enrichment or pathway interpretation only, without needing to first identify sub-populations.

## Inputs

- Reads-in-peaks count matrix downloaded as gzipped single-cell BED-derived count data (e.g. GEO GSE99172)
- Per-cell metadata (cell line / technical replicate / patient sample labels, e.g. TF1, K562, GM12878, SU070, SU353)
- Chosen number of clusters k (e.g. k=5)

## Outputs

- Cluster assignment per cell (e.g. clusters 1-5) with t-SNE and UMAP embeddings on the 50-PC PCA space
- Technical-replicate assignment scores (e.g. K562 99.7%, GM12878 98.3%) and average correct-cluster assignment (e.g. 99.4%)
- Fisher's exact test p-value comparing cluster proportions between samples (e.g. p=0.0025 for SU070 vs SU353) and sub-cluster partition of the dominant cluster C1
- Metrics table written to gse99172_chromscape_metrics.csv

## How to apply

Download the reads-in-peaks count matrix as gzipped single-cell BED-derived counts (e.g. GSE99172), load it in R and run ChromSCape quality control filtering: minimum coverage of 1600 unique reads per cell, removal of cells in the top 5% coverage, and keeping regions detected in at least 1% of cells. Normalize the filtered matrix by total read count and region size, then apply PCA with centering, retaining the first 50 PCs, and cluster cells (e.g. k=5) with t-SNE and UMAP on the PCA embedding for two-dimensional visualization. Compute replicate assignment scores (proportion of cells from each technical replicate falling in the expected cluster) and an average correct-cluster assignment across cell lines, and use Fisher's exact test to compare proportions of cells in a dominant cluster (e.g. monocyte-cluster C1 in patient-derived samples SU070 vs SU353), sub-clustering C1 if needed. Write recomputed metrics to a CSV (e.g. gse99172_chromscape_metrics.csv) and compare against paper-reported values.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application used for QC filtering, normalization, PCA (50 PCs), clustering, t-SNE/UMAP visualization, and sub-population identification) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Language in which the count matrix is loaded, filtered, clustered, and metrics computed)
- **Bioconductor** (Distribution channel of the ChromSCape package)
- **Shiny** (Interactive graphical interface through which sub-populations are identified and visualized)

## Evaluation signals

- K562 technical-replicate assignment score matches the reported 99.7% within a small tolerance.
- GM12878 technical-replicate assignment score matches the reported 98.3% within a small tolerance.
- Average correct-cluster assignment across TF1, K562 and GM12878 cells matches the reported 99.4%.
- Fisher's exact test p-value for monocyte-cluster proportions between SU070 and SU353 matches the reported 0.0025.
- t-SNE and UMAP plots on the PCA embedding show visually separated clusters consistent with cell line of origin (k=5).

## Limitations

- Sub-population recovery depends on QC thresholds (1600 reads minimum, top 5% coverage removal, 1% cell-detection region filter); changing them changes clusters and scores.
- Cluster labels are numeric (e.g. cluster 5 for K562, cluster 2 for GM12878) and not guaranteed to be stable across runs or ChromSCape versions, since most R dependency versions are unreported in the paper.
- Benchmarking scope was restricted (e.g. a 1287-cell subset and 500-cell random sub-sampling of MDA-MB-468, plus removal of 77 Mbp of known MDA-MB-468 CNV regions), so performance on other datasets may differ.

## Evidence

- [discussion] Sub-population identification on scATAC-seq: "It also manages to identify sub-populations within a complex scATAC-seq benchmarking dataset"
- [discussion] GSE99172 count matrix input: "The count matrix of reads in peaks was downloaded from GEO accession number “GSE99172”"
- [discussion] PCA dimensionality reduction step: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [intro] Interactive sub-population identification: "The user can interactively identify subpopulations with common epigenomes within heterogeneous samples"
- [readme] ChromSCape benchmarking scripts repository: "Scripts and dataset to reproduce benchmarking of SnapATAC, CisTopic, Cusanovich2018 and EpiScanpy &amp; scOrange on single-cell H3K27me3 cell mix"
