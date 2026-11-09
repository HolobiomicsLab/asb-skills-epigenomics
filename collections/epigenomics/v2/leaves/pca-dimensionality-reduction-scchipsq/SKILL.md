---
name: pca-dimensionality-reduction-scchipsq
description: Use when a quality-controlled, sparse scChIP-seq count matrix (e.g. H3K27me3
  from PDX tumor micro-environment samples) needs a dense low-dimensional feature
  space before clustering or 2D visualization.
when_to_use_negative:
- Input is raw count data that has not passed QC and correlation filtering — ChromSCape
  normalizes and runs PCA only after quality control filtering of outlier cells
- Analysis goal is peak-level feature extraction with motif/topic models (e.g. CisTopic,
  SnapATAC, Cusanovich2018), which the article benchmarks as alternative featurization
  methods rather than PCA
- Regions have not been filtered (e.g. regions not detected in at least 1% of cells,
  or user-excluded BED regions such as CNV segments not yet removed)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: ChromSCape
  role: Interactive Shiny/R Bioconductor application that normalizes the QC-filtered
    matrix by total read count and region size and runs PCA with centering keeping
    the 50 first PCs
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Language in which the PCA/normalization steps of the ChromSCape pipeline are
    implemented and scripted
- name: Bioconductor
  role: Distribution channel of ChromSCape, which performs the PCA-based dimensionality
    reduction
- name: ConsensusClusterPlus
  role: Consumes the 50-PC feature space to compute consensus clustering of cells
    (e.g. k=4 for the TME PDX data)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:71a1d94ac5fa12e73eea3fabbc7489fadd787b46399dbb767b71ef96dffe76c5
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pca-dimensionality-reduction-scchipsq

## Summary

Dimensionality reduction of filtered, normalized single-cell H3K27me3 scChIP-seq count matrices by PCA with centering, retaining the first 50 PCs as the feature space used downstream for clustering (ConsensusClusterPlus), visualization, and differential enrichment analysis in ChromSCape.

## When to use

Use when a quality-controlled, sparse scChIP-seq count matrix (e.g. H3K27me3 from PDX tumor micro-environment samples) needs a dense low-dimensional feature space before clustering or 2D visualization.

## When NOT to use

- Input is raw count data that has not passed QC and correlation filtering — ChromSCape normalizes and runs PCA only after quality control filtering of outlier cells
- Analysis goal is peak-level feature extraction with motif/topic models (e.g. CisTopic, SnapATAC, Cusanovich2018), which the article benchmarks as alternative featurization methods rather than PCA
- Regions have not been filtered (e.g. regions not detected in at least 1% of cells, or user-excluded BED regions such as CNV segments not yet removed)

## Inputs

- Quality-controlled and correlation-filtered scChIP-seq count matrix (cells x genomic regions, e.g. 50kbp bins or peaks) from H3K27me3 single-cell ChIP-seq samples
- Per-cell total read counts and region sizes for normalization
- Sample/barcode metadata for downstream visualization and batch correction (optional)

## Outputs

- Cell embeddings on the first 50 principal components (n_cells x 50 matrix)
- PC coordinates used as input to t-SNE/UMAP plots and ConsensusClusterPlus consensus clustering (e.g. k=4 cluster_assignments.csv)
- Optionally, PC space after FastMNN batch correction for multi-sample integration

## How to apply

First apply cell QC (minimum 1600 unique reads per cell, remove top 5% coverage percentile) and correlation-based filtering, keeping regions detected in at least 1% of cells, to reach the retained cell set (e.g. n=903 for the TME data). Then normalize the filtered matrix by total read count and region size. Apply PCA to this normalized matrix with centering and keep the first 50 PCs for further analysis; these PCs are the input for t-SNE/UMAP 2D visualization, optional FastMNN batch correction for multi-sample data, and ConsensusClusterPlus consensus clustering (e.g. k=4 for the four PDX models). Rationale: PCA on read-depth- and region-size-normalized data compresses the sparse per-cell signal into a feature space where chromatin-based subpopulations separate while suppressing coverage-driven artifacts.

## Related tools

- **ChromSCape** (Interactive Shiny/R Bioconductor application that normalizes the QC-filtered matrix by total read count and region size and runs PCA with centering keeping the 50 first PCs) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Language in which the PCA/normalization steps of the ChromSCape pipeline are implemented and scripted)
- **Bioconductor** (Distribution channel of ChromSCape, which performs the PCA-based dimensionality reduction)
- **ConsensusClusterPlus** (Consumes the 50-PC feature space to compute consensus clustering of cells (e.g. k=4 for the TME PDX data))

## Examples

```
pca <- prcomp(t(mat_normalized), center = TRUE); pcs <- pca$x[, 1:50]
```

## Evaluation signals

- PCA is run on the normalized (total read count and region size) matrix, not on raw counts
- Exactly the first 50 PCs are retained (with centering) for all downstream clustering/visualization
- t-SNE/UMAP 2D embeddings computed on the PCA output should visually separate the expected subpopulations (e.g. cluster C3 composed almost exclusively of HBCx-95 cells)
- Consensus clustering on the PC space should reproduce mean consensus scores comparable to reported values (0.91 for selected clusters vs 0.17 for others at k=4)
- The cell set entering PCA matches the QC-filtered set (e.g. n=903 cells for the TME data)

## Limitations

- Benchmark comparisons were restricted to a subsampled set of 1287 cells and random sub-sampling of 500 MDA-MB-468 cells, potentially limiting scope
- Prior removal of known CNV segments of MDA-MB-468 (77 Mbp) was required, an extra manual curation step before dimensionality reduction
- Coverage and sparsity of single-cell histone modification datasets are low, so raw-matrix PCA without prior normalization/filtering is not fit for purpose
- Most R dependency versions are unreported, limiting environment reproducibility of the pipeline

## Evidence

- [discussion] PCA keeps 50 PCs: "principle component analysis (PCA) is applied to the matrix, with centering, and the 50 first PCs are kept for further analysis"
- [discussion] Normalization precedes PCA: "Post quality control filtering, the matrices are normalized by total read count and region size"
- [discussion] Visualization on PCA output: "The t-distributed stochastic neighbor embedding (t-SNE) algorithm23 and UMAP24 is applied on the PCA to visualize the data in two dimensions"
- [discussion] QC thresholds preceding PCA: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
