---
name: cellmatch-reference-mapping
description: Use when clustered scRNA-seq data needs cell type annotation and one-against-all
  marker lists (e.g. Seurat FindAllMarkers) may contain pseudo markers upregulated
  in multiple clusters, or when a reproducible, reference-database-based annotation
  is required.
when_to_use_negative:
- The CellMatch reference lacks the tissue or cell type of interest — the article
  notes the limited reference database potentially led to incorrect annotation, and
  users should supply a custom cellmatch in that case
- The goal is annotating individual cells against reference profiles rather than pre-defined
  clusters (cell-based methods such as SingleR, CellAssign, Garnett, scMap, CHETAH
  are the alternative category)
- Data are not yet clustered — cluster analysis must be performed first, since scCATCH
  annotates existing cluster identities rather than performing clustering
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scCATCH
  role: 'Implements the workflow: createscCATCH builds the object, findmarkergene
    performs paired-comparison marker identification, findcelltype scores cell types
    against CellMatch'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Provides one-against-all FindAllMarkers (Wilcoxon rank-sum) gene lists used
    as the baseline for pseudo-marker comparison against scCATCH output
- name: CellMatch
  role: Tissue-specific cell taxonomy reference database used by findcelltype for
    evidence-based cluster annotation
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:dee5d6f2f69d741aac9e1100788b2e46790026ee77d2f4082ba13e7f65c0aee3
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cellmatch-reference-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates pre-identified scRNA-seq clusters by matching paired-comparison marker genes against the CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissues, 2,096 human/mouse references) using scCATCH's evidence-based scoring, thereby avoiding pseudo marker genes from one-against-all marker identification.

## When to use

Use when clustered scRNA-seq data needs cell type annotation and one-against-all marker lists (e.g. Seurat FindAllMarkers) may contain pseudo markers upregulated in multiple clusters, or when a reproducible, reference-database-based annotation is required.

## When NOT to use

- The CellMatch reference lacks the tissue or cell type of interest — the article notes the limited reference database potentially led to incorrect annotation, and users should supply a custom cellmatch in that case
- The goal is annotating individual cells against reference profiles rather than pre-defined clusters (cell-based methods such as SingleR, CellAssign, Garnett, scMap, CHETAH are the alternative category)
- Data are not yet clustered — cluster analysis must be performed first, since scCATCH annotates existing cluster identities rather than performing clustering

## Inputs

- scRNA-seq expression matrix (e.g. a Seurat object's RNA @data slot or sc_data matrix)
- Cluster labels per cell (sc_cluster, e.g. as.character(Idents(Seurat_obj)))
- CellMatch tissue-specific cell taxonomy reference database (ship data of scCATCH, human and mouse)
- findmarkergene parameters: species, marker, tissue, cancer

## Outputs

- scCATCH object with per-cluster paired-comparison marker gene lists (findmarkergene)
- Per-cluster cell type annotations with top-1 evidence-based scores (findcelltype)
- Pseudo-marker comparison table (e.g. seurat_sccatch_marker_comparison.csv) flagging Seurat-only genes

## How to apply

Build the scCATCH object from the expression matrix and cluster labels with createscCATCH(data, cluster), then run findmarkergene(obj, species, marker, tissue, cancer), which applies the paired-comparison criteria log10 fold change >= 0.25, p < 0.05, and expression in >= 25% of cells so that only genes significantly upregulated in one cluster are retained. Then call findcelltype(obj), which scores cell types per cluster against CellMatch based on validated marker genes and their supporting literature and assigns the top-1 scoring type. To audit pseudo markers, intersect Seurat's one-against-all FindAllMarkers (Wilcoxon rank-sum) gene sets with the scCATCH paired-comparison sets and flag Seurat-only genes as candidate pseudo markers. In the Gierahn dataset this workflow showed Seurat-alone annotation was consistent with manual labels for only 2 of 5 clusters (40%; T cells in cluster 2, monocytes in cluster 5), with CCL22, SWAP70, and KLRF1 as unshared pseudo markers in clusters 1, 3, and 4.

## Related tools

- **scCATCH** (Implements the workflow: createscCATCH builds the object, findmarkergene performs paired-comparison marker identification, findcelltype scores cell types against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Provides one-against-all FindAllMarkers (Wilcoxon rank-sum) gene lists used as the baseline for pseudo-marker comparison against scCATCH output)
- **CellMatch** (Tissue-specific cell taxonomy reference database used by findcelltype for evidence-based cluster annotation) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Every reported marker gene satisfies the scCATCH criteria: log10 fold change >= 0.25, p < 0.05 (Wilcoxon test), and expression in >= 25% of cells in only one cluster
- Annotated cell types are consistent with manual labels; in the Gierahn benchmark expect ~100% consistency with scCATCH versus 40% (2/5 clusters) for Seurat markers alone
- Unshared pseudo markers (present only in the Seurat one-against-all list, e.g. CCL22, SWAP70, KLRF1 in clusters 1, 3, 4) are flagged by intersecting the two gene sets
- findcelltype assigns exactly one top-1 cell type per cluster, scored on validated marker genes and supporting literature
- The comparison table (e.g. seurat_sccatch_marker_comparison.csv) is produced with one row per cluster per method

## Limitations

- Performance majorly depends on the reference database; a limited CellMatch can lead to incorrect annotation
- Effects of multiple clustering algorithms on cell type annotation were not evaluated (deferred to future work)
- No software versions stated for compared tools; no accession IDs or hosting URLs given for CellMatch in the article

## Evidence

- [intro] a bunch of pseudo marker genes (significantly upregulated at least two clusters rather than in one cluster) may occur, which would lead to incorrect cell type annotation: [evidence span withheld — anchor status: not_found]
- [intro] Compared with every other cluster, genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in more than a quarter of cells (≥25%): "genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in more than a quarter of cells (≥25%) would be considered marker genes"
- [methods] CellMatch includes a panel of 353 cell types and related 686 subtypes associated with 184 tissue types, and 2,096 references of human and mouse.: "CellMatch includes a panel of 353 cell types and related 686 subtypes associated with 184 tissue types, and 2,096 references of human and mouse."
- [methods] obj <- findcelltype(obj): "# find cell type for each cluster
obj <- findcelltype(obj)"
