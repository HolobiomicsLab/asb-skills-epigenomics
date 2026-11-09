---
name: paired-cluster-marker-gene-comparison
description: Use when scRNA-seq clusters were annotated with one-against-all marker
  genes (e.g., Seurat FindAllMarkers) and you need to check whether pseudo markers
  (genes significantly upregulated in at least two clusters) caused incorrect cell
  type annotations.
when_to_use_negative:
- Clusters have already been annotated with a paired-comparison or cell-based annotation
  method (e.g., scCATCH findcelltype) — there are no one-against-all pseudo markers
  to expose
- The tissue or species of interest is absent from the CellMatch reference database,
  since scCATCH's performance depends on reference coverage and a limited reference
  can itself cause incorrect annotation
- The analysis goal is cell-level annotation (mapping individual cells to reference
  profiles) rather than cluster-level marker gene quality assessment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: Provides the paired-comparison marker identification (findmarkergene) via
    createscCATCH and the CellMatch reference to benchmark one-against-all markers
    and flag pseudo markers
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Supplies the one-against-all cluster marker gene identification (FindAllMarkers,
    Wilcoxon rank-sum test) whose output is contrasted with scCATCH's paired comparison
  repo: https://github.com/ZJUFanLab/scCATCH_performance_comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:577a8b9bdf86e1a0211ca4bbd9a72634843f60ee92d9ec2a5d1d3a629fc04e6d
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# paired-cluster-marker-gene-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for detecting pseudo marker genes arising from one-against-all cluster marker identification (e.g., Seurat FindAllMarkers) by pairing it with scCATCH's paired-comparison strategy, then quantifying the impact of pseudo markers on cluster annotation accuracy (e.g., Seurat annotating only 2 of 5 Gierahn clusters correctly).

## When to use

Use when scRNA-seq clusters were annotated with one-against-all marker genes (e.g., Seurat FindAllMarkers) and you need to check whether pseudo markers (genes significantly upregulated in at least two clusters) caused incorrect cell type annotations.

## When NOT to use

- Clusters have already been annotated with a paired-comparison or cell-based annotation method (e.g., scCATCH findcelltype) — there are no one-against-all pseudo markers to expose
- The tissue or species of interest is absent from the CellMatch reference database, since scCATCH's performance depends on reference coverage and a limited reference can itself cause incorrect annotation
- The analysis goal is cell-level annotation (mapping individual cells to reference profiles) rather than cluster-level marker gene quality assessment

## Inputs

- scRNA-seq expression data matrix with cluster labels (e.g., Gierahn dataset and validation datasets from the scCATCH performance comparison vignette)
- Seurat object with cluster identities (Idents) for one-against-all FindAllMarkers
- CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references)
- Manual (ground-truth) cell type labels per cluster for consistency scoring

## Outputs

- Per-cluster Seurat one-against-all marker gene lists (Wilcoxon rank-sum test)
- Per-cluster scCATCH paired-comparison marker gene lists (log10FC >= 0.25, p < 0.05, expressed in >= 25% of cells)
- List of candidate pseudo markers per cluster (genes present only in the Seurat list, e.g., CCL22, SWAP70, KLRF1 in Gierahn clusters 1, 3, 4)
- Annotation consistency counts (clusters matching manual labels, e.g., 2/5 = 40% on Gierahn)
- Full Seurat vs. scCATCH marker comparison table (e.g., seurat_sccatch_marker_comparison.csv)

## How to apply

Run Seurat's one-against-all marker identification (FindAllMarkers, Wilcoxon rank-sum test) per cluster on each dataset to obtain default marker gene lists. In parallel, build a scCATCH object with createscCATCH(data, cluster) and run findmarkergene(obj, species, marker, tissue, cancer) with scCATCH's paired-comparison criteria: log10 fold change >= 0.25, p < 0.05, expression in >= 25% of cells. For each cluster, intersect the Seurat one-against-all gene sets with the scCATCH paired-comparison gene sets and flag genes present only in the Seurat list as candidate pseudo markers (genes significantly upregulated in at least two clusters rather than one). Annotate clusters using Seurat markers alone against the CellMatch reference (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) and count clusters consistent with manual labels. On the Gierahn dataset this yields 2 of 5 clusters (40%) correctly annotated, with CCL22, SWAP70, and KLRF1 appearing as unshared pseudo markers in clusters 1, 3, and 4; write the comparison table to CSV.

## Related tools

- **scCATCH** (Provides the paired-comparison marker identification (findmarkergene) via createscCATCH and the CellMatch reference to benchmark one-against-all markers and flag pseudo markers) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Supplies the one-against-all cluster marker gene identification (FindAllMarkers, Wilcoxon rank-sum test) whose output is contrasted with scCATCH's paired comparison) — https://github.com/ZJUFanLab/scCATCH_performance_comparison

## Examples

```
obj <- createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))); obj <- findmarkergene(obj, species = 'Human', marker = 'cellmarker', tissue = 'blood', cancer = 'normal')
```

## Evaluation signals

- Annotation consistency: count clusters whose Seurat-only annotation matches manual labels; on the Gierahn dataset expect 2 of 5 clusters (40%), with cluster 2 (T cells) and cluster 5 (monocytes) correct
- Pseudo marker detection: CCL22, SWAP70, and KLRF1 should surface as unshared pseudo markers in Gierahn clusters 1, 3, and 4
- scCATCH marker filters enforced: retained genes satisfy log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells
- Intersection check: genes present in both Seurat and scCATCH lists are 'shared'; genes present only in the Seurat list (upregulated in >= 2 clusters) are flagged as pseudo markers
- Output schema: a complete comparison table per cluster per dataset is written to CSV (e.g., seurat_sccatch_marker_comparison.csv)

## Limitations

- A limited CellMatch reference database can itself lead to incorrect annotation via scCATCH
- Effects of multiple clustering algorithms on cell type annotation were not evaluated (deferred to future work)
- No software versions stated for the compared tools in the article
- Pseudo markers are defined as genes significantly upregulated in at least two clusters rather than one; the method assumes clusters are already defined and does not address clustering quality

## Evidence

- [intro] a bunch of pseudo marker genes (significantly upregulated at least two clusters rather than in one cluster) may occur, which would lead to incorrect cell type annotation: [evidence span withheld — anchor status: not_found]
