---
name: wilcoxon-rank-sum-significance-testing
description: Use when identifying per-cluster marker genes from scRNA-seq data with
  pre-defined cluster labels, and you need to exclude genes upregulated in two or
  more clusters (pseudo marker genes) before cell type annotation.
when_to_use_negative:
- Input lacks cluster labels — the paired comparison requires pre-identified clusters
  (run clustering first, e.g., in Seurat)
- Goal is cell-based annotation from reference profiles rather than cluster-based
  marker gene identification (tools like SingleR or CellAssign are the alternative)
- A limited tissue-specific reference database is available — scCATCH performance
  depends on the reference database and may lead to incorrect annotation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Provides findmarkergene(), which performs paired-cluster Wilcoxon rank-sum
    testing and the fold-change/expression-frequency filters for marker gene identification
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Common upstream tool used to produce the scRNA-seq expression matrix and cluster
    labels that feed into createscCATCH(); Seurat's one-against-all marker identification
    is contrasted with scCATCH's paired comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:dcb02b6fc2460595c8a8eb9fa0ccf96e242843bca7ae14d08b4b66d6ce852d86
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
n_claims: 9
---

# wilcoxon-rank-sum-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the Wilcoxon rank-sum test within scCATCH's findmarkergene() to identify genes significantly upregulated in only one cluster versus every other cluster, avoiding the pseudo marker genes produced by one-against-all comparisons (e.g., in Seurat).

## When to use

Use when identifying per-cluster marker genes from scRNA-seq data with pre-defined cluster labels, and you need to exclude genes upregulated in two or more clusters (pseudo marker genes) before cell type annotation.

## When NOT to use

- Input lacks cluster labels — the paired comparison requires pre-identified clusters (run clustering first, e.g., in Seurat)
- Goal is cell-based annotation from reference profiles rather than cluster-based marker gene identification (tools like SingleR or CellAssign are the alternative)
- A limited tissue-specific reference database is available — scCATCH performance depends on the reference database and may lead to incorrect annotation

## Inputs

- scRNA-seq expression matrix (sc_data)
- Character vector of cluster labels (sc_cluster)
- Species, marker, tissue, and cancer arguments restricting analysis context

## Outputs

- Per-cluster marker gene table (e.g., marker_genes.csv) with genes meeting log10 FC >= 0.25, Wilcoxon p < 0.05, and >= 25% cell-expression frequency

## How to apply

Wrap the scRNA-seq expression matrix (sc_data) and cluster labels (sc_cluster) in a scCATCH object with createscCATCH(), then call findmarkergene() with species, marker, tissue, and cancer arguments. For every pair of clusters, run the Wilcoxon rank-sum test and compute log10 fold change, flagging genes significantly upregulated (log10 FC >= 0.25, p < 0.05) in only one single cluster. Apply the expression-frequency filter keeping genes expressed in >= 25% of the candidate cluster's cells. The paired-comparison rationale is that one-against-all approaches (as in Seurat) can yield pseudo marker genes — genes significantly upregulated in at least two clusters — leading to incorrect cell type annotation.

## Related tools

- **scCATCH** (Provides findmarkergene(), which performs paired-cluster Wilcoxon rank-sum testing and the fold-change/expression-frequency filters for marker gene identification) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Common upstream tool used to produce the scRNA-seq expression matrix and cluster labels that feed into createscCATCH(); Seurat's one-against-all marker identification is contrasted with scCATCH's paired comparison)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmarker', tissue = 'Pancreas', cancer = 'cancer')
```

## Evaluation signals

- Every reported marker gene meets all three criteria: log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of the candidate cluster's cells
- No gene appears as a marker for more than one cluster (pseudo marker genes upregulated in two or more clusters are removed)
- Output marker gene table covers each cluster with its own marker set before findcelltype() scoring
- Downstream annotation accuracy can be cross-checked against scCATCH's reported 67%-100% (average 83%) cluster annotation accuracy on published datasets

## Limitations

- A limited reference database can lead to incorrect cell type annotation via scCATCH
- Effects of multiple clustering algorithms on the annotation were not evaluated (deferred to future work)
- The test relies on pre-defined clusters; clustering quality upstream is not addressed by findmarkergene()

## Evidence

- [intro] pseudo marker genes criteria: [evidence span withheld — anchor status: not_found]
