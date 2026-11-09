---
name: pearson-correlation-outlier-cell-filtering
description: Use when a scChIP-seq or scATAC-seq count matrix has passed read-count
  QC (e.g. ≥1600 unique reads, top 5% coverage removed) but may still contain isolated
  low-quality outlier cells before dimensionality reduction and clustering.
when_to_use_negative:
- Input is already a feature table (PCA embeddings or reduced dimensions) — correlation
  filtering must act on the raw per-cell region coverage matrix
- The dataset is bulk ChIP-seq or otherwise lacks per-cell barcodes, so pairwise cell
  correlations cannot be computed
- Cells have not passed basic read-count QC; applying correlation filtering first
  can remove cells for the wrong reason
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3673
tools:
- name: ChromSCape
  role: Implements the correlation-based cell filter (99th percentile Pearson threshold,
    p=1% default) after read-count QC within its Shiny workflow
  repo: https://github.com/vallotlab/ChromSCape_benchmarking
- name: R
  role: Language in which ChromSCape's filtering and downstream analysis are implemented
    and scripted
- name: Bioconductor
  role: Distribution channel of the ChromSCape package providing the filtering module
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19542-x
    title: Interactive analysis of single-cell epigenomic landscapes with ChromSCape
  source_content_hash: sha256:f24a6fdc63e731459b1ccdf691d1126ed4c66046abc97502657881edf743ae0f
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Pearson-correlation outlier cell filtering

## Summary

A ChromSCape cell-level filtering step that removes isolated single-cell epigenomic profiles whose Pearson pairwise correlation falls below a threshold (99th percentile) with at least 1% of cells, after minimum-coverage QC, to retain a coherent cell set (e.g. n=903 cells in the PDX tumor micro-environment deconvolution).

## When to use

Use when a scChIP-seq or scATAC-seq count matrix has passed read-count QC (e.g. ≥1600 unique reads, top 5% coverage removed) but may still contain isolated low-quality outlier cells before dimensionality reduction and clustering.

## When NOT to use

- Input is already a feature table (PCA embeddings or reduced dimensions) — correlation filtering must act on the raw per-cell region coverage matrix
- The dataset is bulk ChIP-seq or otherwise lacks per-cell barcodes, so pairwise cell correlations cannot be computed
- Cells have not passed basic read-count QC; applying correlation filtering first can remove cells for the wrong reason

## Inputs

- Filtered single-cell count matrix (cells × genomic regions/bins) from scChIP-seq (e.g. H3K27me3 count matrices from the figshare PDX stromal cell deposit)
- Cell-level QC-passed matrix with minimum 1600 unique reads per cell and top 5% coverage cells removed

## Outputs

- Correlation-filtered count matrix retaining only non-isolated cells (e.g. n=903 cells) and regions detected in ≥1% of cells
- Filtered matrix ready for normalization, PCA (first 50 PCs), t-SNE/UMAP, and ConsensusClusterPlus clustering

## How to apply

In ChromSCape, first apply read-count QC (minimum 1600 unique reads per cell, filtering out the top 5% coverage cells), then run the correlation-based filter: compute Pearson pairwise correlation scores between cells and remove cells displaying a score below a threshold t with at least p% of cells, where p is set at 1% by default (threshold at the 99th percentile). This removes cells whose chromatin profiles are uncorrelated with the bulk of the population, which is critical for sparse single-cell histone modification data. Optionally filter regions in the same pass, keeping only regions detected in at least 1% of cells (or supported by a user-defined percentage of cells with coverage >1000 reads). Verify the retained cell count against the expectation (e.g. n=903 cells for the four PDX H3K27me3 TME samples) before normalizing by total read count and region size and running PCA with the first 50 PCs.

## Related tools

- **ChromSCape** (Implements the correlation-based cell filter (99th percentile Pearson threshold, p=1% default) after read-count QC within its Shiny workflow) — https://github.com/vallotlab/ChromSCape_benchmarking
- **R** (Language in which ChromSCape's filtering and downstream analysis are implemented and scripted)
- **Bioconductor** (Distribution channel of the ChromSCape package providing the filtering module)

## Examples

```
# In ChromSCape Shiny app: 'Data processing' tab → 'Filtering' step → set min reads=1600, remove top 5% coverage, check 'Correlation filtering' (99th percentile, p=1%), keep regions in ≥1% of cells
```

## Evaluation signals

- Retained cell count matches the published filtered set (n=903 cells for the four PDX H3K27me3 samples)
- Every retained cell has a Pearson correlation score above threshold t with ≥1% of cells (no isolated profiles remain)
- Downstream consensus clustering at k=4 reproduces reported scores (mean consensus 0.91 for selected clusters vs 0.17 for others)
- Per-cluster differential enrichment yields 189/210/83/9 depleted regions for C1–C4, with cluster C3 almost exclusively HBCx-95 cells

## Limitations

- Threshold t is at the 99th percentile and p defaults to 1%; on very heterogeneous samples (e.g. multi-subtype PDX mixtures) it can remove genuinely distinct subpopulations rather than technical outliers
- Filtering assumes sparse single-cell histone modification data where outliers reflect low quality; the article notes existing tools are 'not fit for the low coverage and sparsity of these single-cell histone modifications datasets', so parameters must be tuned per dataset

## Evidence

- [discussion] Correlation-based filtering of isolated cells: "cells displaying a Pearson’s pairwise correlation score below a threshold t with at least p% of cells are filtered out (p is set at 1% by default)"
- [discussion] Preceding read-count QC thresholds: "a minimum coverage of 1600 unique reads per cell, filtering out the cells with the top 5% coverage"
- [discussion] Region-level filtering companion step: "keeping regions detected in at least 1% of cells"
- [intro] Rationale for aggressive filtering in sparse scChIP data: "Existing tools used to analyze bulk ChIP-seq experiments are not fit for the low coverage and sparsity of these single-cell histone modifications datasets"
