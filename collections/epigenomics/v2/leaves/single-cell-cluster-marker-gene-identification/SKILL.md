---
name: single-cell-cluster-marker-gene-identification
description: Use when you have a scRNA-seq expression matrix with predefined cluster
  labels and need a reliable per-cluster marker gene set before cell type annotation,
  especially to avoid pseudo marker genes from one-against-all comparisons.
when_to_use_negative:
- Input has no cluster labels yet — clustering must be performed first before paired
  marker gene identification
- The goal is differential expression per cluster against all other clusters combined
  (one-against-all), where pseudo marker genes are acceptable or expected by design
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: Provides createscCATCH() and findmarkergene() implementing the paired-comparison
    marker gene identification with the log10 FC >= 0.25, p < 0.05, >= 25% expression
    filters
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Alternative tool that derives cluster marker genes via one-against-all comparisons;
    scCATCH's paired approach is contrasted against it to avoid pseudo marker genes;
    Seurat objects can be converted into scCATCH objects
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:0b4c776c9237f642a1b04c3c9578dc680700fc960940e538528b9fec70694d8b
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-cluster-marker-gene-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies per-cluster marker genes from scRNA-seq expression matrices using scCATCH's paired-comparison strategy, which avoids the pseudo marker genes produced by one-against-all approaches (genes significantly upregulated in two or more clusters) and thereby prevents incorrect cell type annotation downstream.

## When to use

Use when you have a scRNA-seq expression matrix with predefined cluster labels and need a reliable per-cluster marker gene set before cell type annotation, especially to avoid pseudo marker genes from one-against-all comparisons.

## When NOT to use

- Input has no cluster labels yet — clustering must be performed first before paired marker gene identification
- The goal is differential expression per cluster against all other clusters combined (one-against-all), where pseudo marker genes are acceptable or expected by design

## Inputs

- scRNA-seq expression data matrix (sc_data)
- Character vector of cluster labels (sc_cluster)
- Species, marker, tissue, and cancer context arguments for findmarkergene()

## Outputs

- Per-cluster marker gene table (e.g. marker_genes.csv) with genes meeting log10 FC >= 0.25, Wilcoxon p < 0.05, and >= 25% cell-expression frequency
- scCATCH object updated with marker gene results, ready for findcelltype()

## How to apply

Wrap the expression matrix (sc_data) and cluster labels (sc_cluster) in a scCATCH object with createscCATCH(), then call findmarkergene() supplying species, marker, tissue, and cancer arguments to restrict the analysis context. For every pair of clusters, scCATCH applies the Wilcoxon rank-sum test and computes log10 fold change, retaining as candidate markers only genes significantly upregulated in one single cluster (log10 FC >= 0.25, Wilcoxon p < 0.05). A second filter keeps only genes expressed in >= 25% of the cells of the candidate cluster, which removes pseudo marker genes upregulated in two or more clusters — the flaw of the one-against-all design used in Seurat. The retained genes are assembled into a per-cluster marker gene table that feeds findcelltype() for evidence-based scoring against the CellMatch database.

## Related tools

- **scCATCH** (Provides createscCATCH() and findmarkergene() implementing the paired-comparison marker gene identification with the log10 FC >= 0.25, p < 0.05, >= 25% expression filters) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Alternative tool that derives cluster marker genes via one-against-all comparisons; scCATCH's paired approach is contrasted against it to avoid pseudo marker genes; Seurat objects can be converted into scCATCH objects)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmarker', tissue = 'Blood', cancer = 'normal')
```

## Evaluation signals

- Every reported marker gene in the output table satisfies all three criteria: log10 FC >= 0.25, Wilcoxon p < 0.05, and >= 25% cell-expression frequency
- No gene appears as a marker for more than one cluster — marker sets are cluster-unique, confirming pseudo marker genes (upregulated in two or more clusters) were removed
- Output table contains a marker gene set for every cluster in sc_cluster
- Downstream findcelltype() annotation scores can be reproduced from the retained marker genes matched against CellMatch (353 cell types, 686 subtypes, 184 tissue types)

## Limitations

- Downstream annotation performance depends on the reference database; a limited reference database can lead to incorrect cell type annotation
- Marker gene criteria are fixed at log10 FC >= 0.25, Wilcoxon p < 0.05, and >= 25% expression frequency; genes upregulated in two or more clusters are always discarded by design

## Evidence

- [intro] Pseudo marker genes defined by paired-comparison rationale: [evidence span withheld — anchor status: not_found]
