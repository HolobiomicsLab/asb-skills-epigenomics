---
name: nearest-neighbor-cell-clustering
description: Use when Copy-scAT per-cell CNV assignments are noisy (e.g., sparse scATAC
  coverage) and you want to smooth them into one consensus chromosome value per cell
  cluster using Louvain (or k-means) clustering of the accessibility matrix.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs, so cluster-consensus CNV smoothing has nothing to stabilize (a stated Copy-scAT
  limitation).
- You lack the required upstream artifacts (CNV list and identifyCNVClusters output),
  so smoothClusters cannot be called.
- A cluster-level consensus genotype table already exists and per-cell re-smoothing
  is unnecessary.
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Copy-scAT
  role: Provides smoothClusters and the Louvain-based nearest-neighbor clustering
    workflow used to smooth per-cell CNV assignments into cluster consensus genotypes
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Implementation environment for Copy-scAT (R 4.0.0 recommended in the README)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:f611e116386e72c9d22a66ebfd231ee43a393f5a8f9bc69704b3420983b40ecd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# nearest-neighbor-cell-clustering

## Summary

Generates nearest-neighbor (Louvain) cell clusters from scATAC accessibility data and uses them within Copy-scAT's smoothClusters module to convert noisy per-cell CNV calls into consensus CNV genotypes per cluster of cells, improving accuracy for cells with sparse coverage.

## When to use

Use when Copy-scAT per-cell CNV assignments are noisy (e.g., sparse scATAC coverage) and you want to smooth them into one consensus chromosome value per cell cluster using Louvain (or k-means) clustering of the accessibility matrix.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs, so cluster-consensus CNV smoothing has nothing to stabilize (a stated Copy-scAT limitation).
- You lack the required upstream artifacts (CNV list and identifyCNVClusters output), so smoothClusters cannot be called.
- A cluster-level consensus genotype table already exists and per-cell re-smoothing is unnecessary.

## Inputs

- cell cluster object (from TF-IDF-normalized scATAC accessibility matrix, SVD, UMAP, Louvain clustering)
- CNV list (per-cell CNV assignments from upstream Copy-scAT steps)
- CNV cluster file produced by identifyCNVClusters
- parameters: percentPositive threshold, removeEmpty flag

## Outputs

- smoothed_cnv_clusters.csv (one consensus integer CNV genotype per cluster per chromosome bin)
- cluster membership assignments (Louvain clusters, or k-means clusters)

## How to apply

Build cell clusters as parameterized in the paper's Methods: TF-IDF normalization of the accessibility matrix, SVD dimensionality reduction, UMAP embedding, and nearest-neighbor Louvain clustering (k-means is an accepted alternative). Then call smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive, removeEmpty), supplying the cluster object, the CNV list, and the CNV cluster file produced upstream by identifyCNVClusters; smoothing imputes a consensus chromosome value per cluster using a boost value of 0 with averaging and rounding. Optionally set removeEmpty to drop clusters that fail the percentPositive threshold. The rationale is that cells sharing a CNV tend to cluster together, so cluster-level consensus smooths sparse single-cell signals. The result is written to smoothed_cnv_clusters.csv.

## Related tools

- **Copy-scAT** (Provides smoothClusters and the Louvain-based nearest-neighbor clustering workflow used to smooth per-cell CNV assignments into cluster consensus genotypes) — github.com/spcdot/CopyscAT
- **R** (Implementation environment for Copy-scAT (R 4.0.0 recommended in the README))

## Examples

```
smoothed <- smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive = 0.5, removeEmpty = TRUE); write.csv(smoothed, "smoothed_cnv_clusters.csv")
```

## Evaluation signals

- smoothed_cnv_clusters.csv exists and parses as a table with one consensus genotype per cluster.
- All consensus CNV values in smoothed_cnv_clusters.csv are valid integer CNV states.
- Per-cluster consensus values are internally consistent across bins of the same chromosome segment.
- Clusters failing the percentPositive threshold are absent from the output when removeEmpty is enabled.
- Consensus values match averaging-with-rounding (boost = 0) of member cells' CNV calls for a spot-checked cluster.

## Limitations

- Copy-scAT is in early beta per the README; documentation and testing are incomplete.
- Requires an R 4.0.0 environment for development-tested operation.
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified; further studies are needed.
- Clustering results depend on the embedding/clustering choice; effects are less pronounced when clustering is based on motif activity scores (e.g., ChromVAR).

## Evidence

- [methods] clusterCNV(initialResultList, medianIQR, maxClust, minDiff): "clusterCNV(initialResultList, medianIQR, maxClust, minDiff)"
- [discussion] These effects are less pronounced when clustering is based on motif activity scores (e.g., ChromVAR): "These effects are less pronounced when clustering is based on motif activity scores (e.g., ChromVAR)"
- [other] Call smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive, removeEmpty) to impute a consensus chromosome value per cluster, using a boost value of 0 and averaging with rounding.: [evidence span withheld — anchor status: not_found]
- [readme] CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file.: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
