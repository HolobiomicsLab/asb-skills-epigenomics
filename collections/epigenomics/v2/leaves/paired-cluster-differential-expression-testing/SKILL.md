---
name: paired-cluster-differential-expression-testing
description: Use when assigning cell identities to pre-computed scRNA-seq clusters
  and you need marker genes that are specific to one cluster only, avoiding pseudo
  marker genes upregulated in two or more clusters.
when_to_use_negative:
- When the goal is a ranked list of genes enriched per cluster for exploratory plotting
  rather than strictly cluster-specific markers — a one-against-all approach (e.g.
  Seurat) may be preferred.
- 'When clusters have not yet been defined: this skill requires pre-existing cluster
  labels as input.'
- When no tissue-specific reference (CellMatch or custom cellmatch) is available —
  scCATCH performance majorly depends on the reference database, and a limited reference
  can lead to incorrect annotation.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Provides createscCATCH() and findmarkergene() for paired-comparison marker
    gene identification with the log10 FC >= 0.25, p < 0.05, >= 25% expression criteria
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Common upstream pipeline for scRNA-seq processing and clustering; Seurat objects
    can be converted via createscCATCH(data = Seurat_obj[['RNA']]@data, cluster =
    as.character(Idents(Seurat_obj)))
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:58b12fe4fc24c58613493981bfefe1b5a4085b52cedbc9406c516f5deb440412
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scCATCH
  repo_url: https://github.com/ZJUFanLab/scCATCH
  tools:
  - Seurat
  - scCATCH
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# paired-cluster-differential-expression-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify cluster marker genes in scRNA-seq data by paired (cluster-vs-cluster) Wilcoxon rank-sum comparisons instead of a one-against-all approach, so that only genes significantly upregulated in a single cluster are retained. This avoids pseudo marker genes that would lead to incorrect cell type annotation.

## When to use

Use when assigning cell identities to pre-computed scRNA-seq clusters and you need marker genes that are specific to one cluster only, avoiding pseudo marker genes upregulated in two or more clusters.

## When NOT to use

- When the goal is a ranked list of genes enriched per cluster for exploratory plotting rather than strictly cluster-specific markers — a one-against-all approach (e.g. Seurat) may be preferred.
- When clusters have not yet been defined: this skill requires pre-existing cluster labels as input.
- When no tissue-specific reference (CellMatch or custom cellmatch) is available — scCATCH performance majorly depends on the reference database, and a limited reference can lead to incorrect annotation.

## Inputs

- scRNA-seq expression matrix (sc_data)
- Character vector of cluster labels (sc_cluster)
- Species, marker, tissue, and cancer context arguments restricting the analysis

## Outputs

- Per-cluster marker gene table (e.g. marker_genes.csv) with genes meeting log10 FC >= 0.25, Wilcoxon p < 0.05, and >= 25% cell-expression frequency
- Updated scCATCH object carrying marker gene results for downstream findcelltype() annotation

## How to apply

Wrap the scRNA-seq expression matrix (sc_data) and cluster labels (sc_cluster) in a scCATCH object with createscCATCH(), then run findmarkergene() supplying species, marker, tissue, and cancer arguments to restrict the analysis context. For every pair of clusters, apply the Wilcoxon rank-sum test and compute log10 fold change, flagging genes that are significantly upregulated (log10 FC >= 0.25, Wilcoxon p < 0.05) in only one cluster. Then apply an expression-frequency filter keeping only candidates expressed in >= 25% of the cells of the candidate cluster, thereby removing pseudo marker genes upregulated in two or more clusters. Assemble the retained genes into a per-cluster marker gene table, which feeds downstream evidence-based scoring (findcelltype) against the CellMatch tissue-specific taxonomy database.

## Related tools

- **scCATCH** (Provides createscCATCH() and findmarkergene() for paired-comparison marker gene identification with the log10 FC >= 0.25, p < 0.05, >= 25% expression criteria) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Common upstream pipeline for scRNA-seq processing and clustering; Seurat objects can be converted via createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))))

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer)
```

## Evaluation signals

- Every reported marker gene meets all three criteria: log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of the candidate cluster's cells.
- No gene appears as a marker for more than one cluster — each retained gene is significantly upregulated in only one single cluster.
- The output marker gene table covers every input cluster with a per-cluster marker gene set.
- Downstream findcelltype() annotations based on these markers match expected tissue-specific cell types on benchmark datasets (scCATCH annotated 67%-100%, average 83%, of clusters across six datasets).

## Limitations

- A limited reference database can potentially lead to incorrect annotation of cell types via scCATCH.
- Genes upregulated in two or more clusters (pseudo marker genes) are discarded, so broadly shared activation signatures will not be reported as markers.
- Effects of multiple clustering algorithms on the resulting annotation were not evaluated.

## Evidence

- [intro] a bunch of pseudo marker genes (significantly upregulated at least two clusters rather than in one cluster) may occur, which would lead to incorrect cell type annotation: [evidence span withheld — anchor status: not_found]
