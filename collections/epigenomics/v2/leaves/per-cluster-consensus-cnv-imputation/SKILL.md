---
name: per-cluster-consensus-cnv-imputation
description: Use when per-cell CNV calls from scATAC-seq data are noisy or sparse
  and cluster-level consensus genotypes are needed — e.g. after identifyCNVClusters
  has produced a CNV cluster file and Louvain cell clusters exist.
when_to_use_negative:
- Malignancies driven primarily by single nucleotide variants with no or rare CNVs
  — Copy-scAT cannot infer CNVs in this setting
- When per-cell CNV calls are already high-confidence and cluster-level imputation
  would erase genuine intra-cluster heterogeneity
- When no upstream CNV cluster file from identifyCNVClusters is available, since smoothClusters
  requires it as input
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: Copy-scAT
  role: R package providing smoothClusters for per-cluster consensus CNV imputation
    from scATAC-seq data
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment for Copy-scAT (development ongoing under R 4.0.0; install
    within an R 4.0.0 environment)
  repo: github.com/spcdot/CopyscAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:c0fae2c907fb918a8b509334088817179594570cda0f763467d156553d9e1bc2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# per-cluster-consensus-cnv-imputation

## Summary

Copy-scAT's optional smoothing step that imputes a consensus per-chromosome CNV genotype for each cluster of scATAC-seq cells, improving CNV calls for cells with sparse coverage by leveraging alternative clustering (Louvain by default) rather than raw per-cell bin values.

## When to use

Use when per-cell CNV calls from scATAC-seq data are noisy or sparse and cluster-level consensus genotypes are needed — e.g. after identifyCNVClusters has produced a CNV cluster file and Louvain cell clusters exist.

## When NOT to use

- Malignancies driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot infer CNVs in this setting
- When per-cell CNV calls are already high-confidence and cluster-level imputation would erase genuine intra-cluster heterogeneity
- When no upstream CNV cluster file from identifyCNVClusters is available, since smoothClusters requires it as input

## Inputs

- cell cluster object (Louvain clusters from TF-IDF + SVD + UMAP of the scATAC accessibility matrix)
- CNV list (per-cell CNV assignments from upstream Copy-scAT steps)
- CNV cluster file produced by identifyCNVClusters

## Outputs

- smoothed_cnv_clusters.csv — per-cluster consensus CNV genotype table with integer CNV states per chromosome/bin

## How to apply

Load the three Copy-scAT inputs required by smoothClusters: the cell cluster object, the CNV list, and the CNV cluster file produced upstream by identifyCNVClusters. Generate cell clusters as parameterized in the Methods — TF-IDF normalization of the accessibility matrix, SVD dimensionality reduction, UMAP embedding, and nearest-neighbor Louvain clustering (k-means may also be used). Call smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive, removeEmpty) to impute a consensus chromosome value per cluster using a boost value of 0 and averaging with rounding. Optionally drop clusters failing the percentPositive threshold via the removeEmpty flag, then write the result to smoothed_cnv_clusters.csv. The rationale is that cells with low fragment coverage have unreliable per-bin CNV values, so cluster-level averaging stabilizes genotype assignments.

## Related tools

- **Copy-scAT** (R package providing smoothClusters for per-cluster consensus CNV imputation from scATAC-seq data) — github.com/spcdot/CopyscAT
- **R** (Runtime environment for Copy-scAT (development ongoing under R 4.0.0; install within an R 4.0.0 environment)) — github.com/spcdot/CopyscAT

## Examples

```
smoothed <- smoothClusters(inputClusters, inputCNVList, inputCNVClusterFile, percentPositive = 0.5, removeEmpty = TRUE); write.csv(smoothed, "smoothed_cnv_clusters.csv")
```

## Evaluation signals

- smoothed_cnv_clusters.csv exists and contains one consensus genotype row per cluster
- All consensus CNV values are valid integer CNV states (averaging performed with rounding)
- Per-cluster consensus values are internally consistent across bins of the same chromosome segment
- Clusters not meeting the percentPositive threshold are dropped when removeEmpty is enabled (boost value of 0)
- Consensus states agree with the CNV cluster assignments in the identifyCNVClusters output file

## Limitations

- Smoothing is optional and relies on the assumption that cells within a cluster share a CNV genotype; it can mask genuine subclonal heterogeneity
- Default clustering is Louvain; alternative clusterings (e.g. k-means) may produce different consensus genotypes
- Copy-scAT cannot be used with malignancies driven primarily by SNVs with no or rare CNVs
- CopyscAT is in early beta; documentation and testing are incomplete, and it targets large-scale and local chromosome alterations

## Evidence

- [methods] smoothClusters workflow: [evidence span withheld — anchor status: not_found]
- [methods] smoothClusters call: [evidence span withheld — anchor status: not_found]
- [readme] README scope: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
- [abstract] tool description: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
