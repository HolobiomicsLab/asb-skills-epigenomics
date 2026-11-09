---
name: pseudo-marker-gene-detection
description: Use when annotating scRNA-seq clusters with Seurat FindAllMarkers (Wilcoxon
  rank-sum) and you need to flag candidate pseudo markers by comparing against scCATCH
  paired-comparison gene sets before trusting annotations.
when_to_use_negative:
- The input is not clustered scRNA-seq data — pseudo markers are defined relative
  to cluster-level upregulation, so no clusters means no pseudo markers
- You are already using scCATCH's paired-comparison (findmarkergene + findcelltype)
  for annotation, which by design excludes genes upregulated in more than one cluster
- The target tissue/cell types have poor coverage in the CellMatch reference database,
  since annotation quality (including pseudo-marker impact assessment) majorly depends
  on the reference database
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Used to run one-against-all cluster marker gene identification (FindAllMarkers,
    Wilcoxon rank-sum test) whose output is screened for pseudo markers
- name: scCATCH
  role: Provides the paired-comparison marker gene identification (findmarkergene)
    against which Seurat marker lists are intersected to flag pseudo markers
  repo: https://github.com/ZJUFanLab/scCATCH
- name: CellMatch
  role: Tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes,
    184 tissues, 2,096 human/mouse references) used to score cluster annotations from
    the marker gene lists
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:c0c966772c62dc6d784335cd6d46efff146ad5e0fee3ef4d46ae017fe53fa91f
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
  - CellMatch
  - Seurat
  - scCATCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pseudo-marker-gene-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects pseudo marker genes — genes significantly upregulated in at least two clusters rather than one — that arise from Seurat's one-against-all marker identification (FindAllMarkers) and lead to incorrect cell type annotation, by intersecting them with scCATCH's paired-comparison marker gene sets.

## When to use

Use when annotating scRNA-seq clusters with Seurat FindAllMarkers (Wilcoxon rank-sum) and you need to flag candidate pseudo markers by comparing against scCATCH paired-comparison gene sets before trusting annotations.

## When NOT to use

- The input is not clustered scRNA-seq data — pseudo markers are defined relative to cluster-level upregulation, so no clusters means no pseudo markers
- You are already using scCATCH's paired-comparison (findmarkergene + findcelltype) for annotation, which by design excludes genes upregulated in more than one cluster
- The target tissue/cell types have poor coverage in the CellMatch reference database, since annotation quality (including pseudo-marker impact assessment) majorly depends on the reference database

## Inputs

- scRNA-seq expression data matrix (e.g. sc_data) with cluster assignments (sc_cluster) for each cell
- Seurat object with cell clusters identified (FindAllMarkers input)
- CellMatch tissue-specific cell taxonomy reference database

## Outputs

- Per-cluster Seurat one-against-all marker gene lists (FindAllMarkers, Wilcoxon rank-sum test)
- Per-cluster scCATCH paired-comparison marker gene lists (findmarkergene output)
- List of candidate pseudo markers per cluster (genes present only in the Seurat list)
- Comparison table of Seurat vs scCATCH markers and annotation consistency (e.g. seurat_sccatch_marker_comparison.csv)

## How to apply

Run Seurat's one-against-all marker identification (FindAllMarkers, Wilcoxon rank-sum test) per cluster on the scRNA-seq dataset to obtain the default marker gene lists. In parallel, run scCATCH by building the object with createscCATCH(data, cluster) and calling findmarkergene(obj, species, marker, tissue, cancer), which uses a paired-comparison strategy with criteria of log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells. For each cluster, intersect the Seurat gene sets with the scCATCH paired-comparison gene sets and flag genes present only in the Seurat list as candidate pseudo markers — genes significantly upregulated in at least two clusters rather than one. Annotate clusters using the Seurat markers alone against the CellMatch reference (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) and count clusters consistent with manual labels to quantify annotation impairment. On the Gierahn dataset this approach showed Seurat alone correctly annotated only 2 of 5 clusters (40% consistency), with CCL22, SWAP70, and KLRF1 surfacing as unshared pseudo markers in clusters 1, 3, and 4.

## Related tools

- **Seurat** (Used to run one-against-all cluster marker gene identification (FindAllMarkers, Wilcoxon rank-sum test) whose output is screened for pseudo markers)
- **scCATCH** (Provides the paired-comparison marker gene identification (findmarkergene) against which Seurat marker lists are intersected to flag pseudo markers) — https://github.com/ZJUFanLab/scCATCH
- **CellMatch** (Tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) used to score cluster annotations from the marker gene lists) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer)
```

## Evaluation signals

- Every flagged pseudo marker must be present in the Seurat one-against-all list for a cluster but absent from the scCATCH paired-comparison list for the same cluster
- scCATCH marker genes must satisfy the filter: log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells
- On the Gierahn dataset, annotation with Seurat markers alone should reproduce 2 of 5 clusters consistent with manual labels (40% consistency), specifically cluster 2 (T cells) and cluster 5 (monocytes)
- CCL22, SWAP70, and KLRF1 should appear as unshared pseudo markers in Gierahn clusters 1, 3, and 4 respectively
- The full comparison table must be written to seurat_sccatch_marker_comparison.csv with per-cluster Seurat and scCATCH gene sets

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited reference database can potentially lead to incorrect annotation of cell types
- Effects of multiple clustering algorithms on cell type annotation were not evaluated (deferred to future work)
- No software versions are stated for the compared tools, so exact reproduction of the Seurat behavior may vary across versions

## Evidence

- [intro] pseudo marker genes definition: [evidence span withheld — anchor status: not_found]
