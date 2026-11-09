---
name: pca-dimensionality-reduction-single-cell
description: Use when a sparse single-cell epigenomic count matrix (reads-in-peaks
  scATAC-seq or scChIP-seq bins) has passed QC filtering and normalization and a low-dimensional
  embedding is needed for clustering or 2D visualization.
when_to_use_negative:
- Input is raw fastq/BAM/BED reads or an unfiltered count matrix — QC filtering and
  normalization must precede PCA in the ChromSCape workflow
- The goal is motif/k-mer-level feature extraction, where the benchmarking shows methods
  like chromVAR, BROCKMAN or SCRAT rather than peak-level PCA-style featurization
  are the relevant alternatives
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application that performs the PCA step (with
    centering, keeping the 50 first PCs) after QC filtering and normalization of single-cell
    epigenomic count matrices
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Implementation language for the ChromSCape PCA/reduction pipeline applied
    to the count matrix
- name: Bioconductor
  role: Distribution channel for ChromSCape, through which the PCA-based single-cell
    epigenomic workflow is installed and run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:2e27d59fc23a16a98195a4705e7d0a253419c50b5d4a1af1bb22c1da10756f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallotlab/ChromSCape_benchmarking
  tools:
  - Bioconductor
  - ChromSCape
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pca-dimensionality-reduction-single-cell

## Summary

Dimensionality-reduction step from the ChromSCape workflow: apply PCA with centering to a quality-controlled, normalized single-cell epigenomic count matrix (scATAC-seq reads-in-peaks or scChIP-seq 50kbp bins), keeping the first 50 PCs as the embedding on which clustering and t-SNE/UMAP visualization are performed.

## When to use

Use when a sparse single-cell epigenomic count matrix (reads-in-peaks scATAC-seq or scChIP-seq bins) has passed QC filtering and normalization and a low-dimensional embedding is needed for clustering or 2D visualization.

## When NOT to use

- Input is raw fastq/BAM/BED reads or an unfiltered count matrix — QC filtering and normalization must precede PCA in the ChromSCape workflow
- The goal is motif/k-mer-level feature extraction, where the benchmarking shows methods like chromVAR, BROCKMAN or SCRAT rather than peak-level PCA-style featurization are the relevant alternatives

## Inputs

- Quality-controlled, sparse single-cell count matrix (reads-in-peaks counts, e.g. gzipped single-cell BED-derived counts from GEO accession GSE99172)
- Count matrices of single-cell ChIP-seq samples in 50kbp genomic bins
- Matrix normalized by total read count and region size

## Outputs

- PCA embedding with the 50 first PCs retained per cell
- 2D t-SNE and UMAP visualizations computed on the PCA embedding
- Cluster assignments of cells derived from the PCA space

## How to apply

First filter cells and regions (e.g. minimum coverage of 1600 unique reads per cell, removal of top 5% coverage cells, keeping regions detected in at least 1% of cells), then normalize the filtered matrix by total read count and region size. Apply principal component analysis to the normalized matrix with centering, retaining the first 50 PCs for further analysis. Use the 50-PC embedding as input to unsupervised clustering (e.g. k=5 clusters on the GSE99172 scATAC-seq dataset) and apply t-SNE and UMAP on the PCA to visualize the data in two dimensions. If batches are present, ChromSCape offers a FastMNN (batchelor) batch-correction option before or in place of raw PCA embedding. The rationale is that PCA denoises the sparse signal so that cells group by epigenomic state — e.g. ChromSCape recovered K562 and GM12878 technical replicates in single clusters with assignment scores of 99.7% and 98.3%.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application that performs the PCA step (with centering, keeping the 50 first PCs) after QC filtering and normalization of single-cell epigenomic count matrices) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Implementation language for the ChromSCape PCA/reduction pipeline applied to the count matrix)
- **Bioconductor** (Distribution channel for ChromSCape, through which the PCA-based single-cell epigenomic workflow is installed and run)

## Evaluation signals

- Confirm centering was applied and exactly the first 50 PCs are retained, matching the ChromSCape specification
- Confirm PCA was run on a matrix that had been QC-filtered and normalized by total read count and region size, not on raw counts
- Check the clustering on the PCA embedding reproduces known structure: on GSE99172 with k=5, K562 and GM12878 replicate assignment scores of 99.7% and 98.3% and ~99.4% average correct-cluster assignment for TF1/K562/GM12878 cells
- Verify t-SNE and UMAP plots are computed on the PCA embedding (not on the raw matrix) and visually separate the expected sub-populations
- Inspect the PCA space for batch-driven separation; if present, apply the FastMNN batch-correction option rather than treating PC coordinates as final

## Limitations

- Single-cell epigenomic data are extremely sparse (low coverage), so PCA quality depends critically on the upstream QC thresholds (e.g. minimum 1600 unique reads per cell, top 5% coverage removal, regions detected in at least 1% of cells)
- Benchmarking comparisons via PCA-based pipelines were restricted in scope (e.g. a set of 1287 cells passing quality control thresholds, and random sub-sampling of 500 MDA-MB-468 cells), so embedding quality claims are bounded by those settings
- Batch effects can distort the PCA embedding and require the FastMNN batch-correction option; cells with low pairwise correlation to other cells may additionally need correlation-based filtering (99th percentile threshold, p=1% by default)

## Evidence

- [discussion] PCA applied with centering, 50 PCs kept: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] Normalization precedes PCA: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [discussion] t-SNE and UMAP applied on the PCA: "The t-distributed stochastic neighbor embedding (t-SNE) algorithm23 and UMAP24 is applied on the PCA to visualize the data in two dimensions"
- [discussion] QC thresholds for cell and region filtering before reduction: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [readme] ChromSCape count-matrix input format for the reduction pipeline: "Count matrices of each sample in 50kbp bins can be inputted directly in ChromSCape with the option ‘subsample - 500 cells’ to reproduce the analysis."
