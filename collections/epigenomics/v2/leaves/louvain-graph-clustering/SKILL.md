---
name: louvain-graph-clustering
description: Use when single-cell ATAC-seq cells need to be grouped into clusters
  (e.g., to produce per-cluster consensus CNV genotypes with Copy-scAT's smoothClusters,
  or to define subpopulations prior to downstream smoothing/filtering).
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT cannot be used in that setting, so cluster-consensus CNV smoothing
  has no target signal.
- Input is single-cell RNA-seq rather than scATAC-seq accessibility data — the TF-IDF/SVD/UMAP/Louvain
  pipeline here is grounded in scATAC copy number inference.
- A dedicated control sample or orthogonal CNV genotyping already defines subclones,
  making accessibility-based clustering redundant for CNV consensus.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Copy-scAT
  role: R package whose smoothClusters step consumes the Louvain cell clusters to
    impute per-cluster consensus CNV values (boost 0, averaging with rounding)
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Implementation environment for Copy-scAT (developed with R 4.0.0)
  repo: github.com/spcdot/CopyscAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:e59696446bb06042ea1a2bdfff32b5b8abc75addda2dd5499a6d9e874395b3c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# louvain-graph-clustering

## Summary

Generate cell clusters from scATAC-seq accessibility data via TF-IDF normalization, SVD dimensionality reduction, UMAP embedding, and nearest-neighbor Louvain clustering, as used in Copy-scAT to supply alternative clustering information for smoothing per-cell CNV calls into consensus genotypes.

## When to use

Use when single-cell ATAC-seq cells need to be grouped into clusters (e.g., to produce per-cluster consensus CNV genotypes with Copy-scAT's smoothClusters, or to define subpopulations prior to downstream smoothing/filtering).

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot be used in that setting, so cluster-consensus CNV smoothing has no target signal.
- Input is single-cell RNA-seq rather than scATAC-seq accessibility data — the TF-IDF/SVD/UMAP/Louvain pipeline here is grounded in scATAC copy number inference.
- A dedicated control sample or orthogonal CNV genotyping already defines subclones, making accessibility-based clustering redundant for CNV consensus.

## Inputs

- single-cell chromatin accessibility matrix (scATAC-seq)
- cell cluster object (from TF-IDF/SVD/UMAP/Louvain pipeline)
- per-cell CNV list (initial CNV assignments from Copy-scAT)
- CNV cluster file produced upstream by identifyCNVClusters
- percentPositive threshold and removeEmpty flag

## Outputs

- cell cluster assignments (Louvain communities, or k-means clusters)
- UMAP embedding and SVD-reduced representation of cells
- smoothed per-cluster consensus CNV genotype table (smoothed_cnv_clusters.csv)

## How to apply

Starting from the single-cell accessibility matrix, apply TF-IDF normalization to the accessibility matrix, then reduce dimensionality with SVD, embed the reduced space with UMAP, and build a nearest-neighbor graph on which Louvain community detection is run to define cell clusters (k-means may be substituted if preferred). The resulting cluster assignments are combined with the CNV list and the CNV cluster file from identifyCNVClusters by calling smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive, removeEmpty) with boost value 0 and averaging with rounding, imputing one consensus chromosome value per cluster. Optionally use the removeEmpty flag to drop clusters below the percentPositive threshold. Rationale: cells sharing a given CNV tend to cluster together in scATAC experiments, so cluster-level consensus improves CNV accuracy for cells with sparse coverage.

## Related tools

- **Copy-scAT** (R package whose smoothClusters step consumes the Louvain cell clusters to impute per-cluster consensus CNV values (boost 0, averaging with rounding)) — github.com/spcdot/CopyscAT
- **R** (Implementation environment for Copy-scAT (developed with R 4.0.0)) — github.com/spcdot/CopyscAT

## Examples

```
smoothed <- smoothClusters(inputClusters = louvainClusters, inputCNVList = cnvList, inputCNVClusterFile = cnvClusters, percentPositive = 0.5, removeEmpty = TRUE); write.csv(smoothed, "smoothed_cnv_clusters.csv")
```

## Evaluation signals

- smoothed_cnv_clusters.csv exists and contains exactly one consensus genotype per cluster with valid integer CNV states
- Per-cluster consensus values are internally consistent across bins of the same chromosome segment
- Clusters below the percentPositive threshold are removed when removeEmpty is set
- Louvain clusters recapitulate CNV co-occurrence — cells sharing a given CNV should tend to fall in the same cluster
- SVD/UMAP embedding is computed from the TF-IDF-normalized accessibility matrix, not raw counts

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants with no or rare CNVs, limiting the value of cluster-consensus smoothing in such datasets
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified; further studies are needed
- Louvain is the default clustering choice; k-means may also be used but clustering based on accessibility rather than motif activity (e.g., ChromVAR) can show more pronounced effects
- CopyscAT is early beta software; documentation and testing are works in progress, and it should be installed within an R 4.0.0 environment

## Evidence

- [methods] Copy-scAT optionally smooths per-cell CNV assignments using alternative clustering information: [evidence span withheld — anchor status: not_found]
- [methods] TF-IDF normalization of the accessibility matrix, SVD dimensionality reduction, UMAP embedding, and nearest-neighbor Louvain clustering: [evidence span withheld — anchor status: not_found]
- [methods] Call smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive, removeEmpty): [evidence span withheld — anchor status: not_found]
- [intro] cells that share a given CNV tend to cluster together in scATAC experiments: "cells that share a given CNV tend to cluster together in scATAC experiments, suggesting that genetics may impart information that results in the emergence of specific epigenetic profiles"
- [readme] Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment.: "Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
