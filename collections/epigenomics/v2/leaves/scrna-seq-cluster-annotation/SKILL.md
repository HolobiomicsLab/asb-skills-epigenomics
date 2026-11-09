---
name: scrna-seq-cluster-annotation
description: Use when scRNA-seq count matrices with pre-computed cluster assignments
  need automatic cell type labels, replacing subjective manual marker-gene annotation
  with a tissue-specific, evidence-scored reference lookup.
when_to_use_negative:
- When the tissue or cell types of interest are not represented in the CellMatch reference
  database — a limited reference database can lead to incorrect annotation
- When the goal is marker gene discovery rather than cluster-level cell type labeling
  (Seurat's one-against-all FindAllMarkers is the comparator workflow)
- When no cluster assignments exist yet — scCATCH annotates pre-identified clusters
  rather than performing clustering
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scCATCH
  role: 'Primary tool: createscCATCH object construction, findmarkergene() paired-comparison
    marker identification, findcelltype() evidence-scored annotation'
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Upstream preprocessing/clustering and one-against-all marker identification
    (FindAllMarkers) used as the comparator baseline
- name: CellMatch
  role: Tissue-specific cell taxonomy reference database bundled with scCATCH (353
    cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references)
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:fe49794e17e95c5b761ac31570d2b80449adb04195b8c13844fbb43f66954714
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
  - CHETAH
  - CellMatch
  - Seurat
  - SingleR
  - scCATCH
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scrna-seq-cluster-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic cell type annotation for scRNA-seq clusters using scCATCH, which identifies cluster marker genes by paired comparison and scores cell types against the tissue-specific CellMatch database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references) with evidence-based scoring.

## When to use

Use when scRNA-seq count matrices with pre-computed cluster assignments need automatic cell type labels, replacing subjective manual marker-gene annotation with a tissue-specific, evidence-scored reference lookup.

## When NOT to use

- When the tissue or cell types of interest are not represented in the CellMatch reference database — a limited reference database can lead to incorrect annotation
- When the goal is marker gene discovery rather than cluster-level cell type labeling (Seurat's one-against-all FindAllMarkers is the comparator workflow)
- When no cluster assignments exist yet — scCATCH annotates pre-identified clusters rather than performing clustering

## Inputs

- scRNA-seq count matrix (sc_data) with pre-computed cluster assignments (sc_cluster)
- Seurat object (converted to data matrix plus Idents)
- Tissue and species specification for the CellMatch reference (human or mouse)

## Outputs

- Per-cluster cell type annotations (top-1 scored cell type per cluster)
- Paired-comparison marker gene lists per cluster
- Cluster annotation consistency rates versus manual labels (e.g., table2_consistency_rates.csv, sccatch_annotations.csv)

## How to apply

Build a scCATCH object with createscCATCH(data = sc_data, cluster = sc_cluster), passing the count matrix and pre-computed cluster assignments (a Seurat object can be converted via Seurat_obj[['RNA']]@data and as.character(Idents(Seurat_obj))). Run findmarkergene(obj, species, marker, tissue, cancer) with the tissue-appropriate CellMatch reference; the paired-comparison strategy keeps only genes significantly upregulated in one cluster (log10 fold change >= 0.25, Wilcoxon p < 0.05) and expressed in >= 25% of cells, avoiding pseudo markers that arise in Seurat's one-against-all FindAllMarkers strategy. Then call findcelltype(obj) so each cluster's cell types are scored on validated marker genes and their supporting literature, and the top-1 scoring type is assigned. Compare resulting annotations against manually curated labels to compute a per-dataset cluster consistency rate; across six published datasets (Enge, Wu, Lindsey, Zheng, Zeisel, Heng) the expected consistency ranges from 67% to 100% with an average of 83% (e.g., Enge 4/6, Wu 7/7, Zeisel 10/10, Heng 9/12).

## Related tools

- **scCATCH** (Primary tool: createscCATCH object construction, findmarkergene() paired-comparison marker identification, findcelltype() evidence-scored annotation) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Upstream preprocessing/clustering and one-against-all marker identification (FindAllMarkers) used as the comparator baseline)
- **CellMatch** (Tissue-specific cell taxonomy reference database bundled with scCATCH (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references)) — https://github.com/ZJUFanLab/scCATCH/tree/master/data

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Per-dataset cluster consistency rates fall in the 67%–100% range with an overall mean of 83% (expected counts: Enge 4/6, Wu 7/7, Lindsey 7/7, Zheng 7/9, Zeisel 10/10, Heng 9/12)
- Known misannotations reproduce: Zheng clusters 1–2 annotated as naive/regulatory T cells; Heng 3 clusters annotated as neuroblasts and spiral ganglionic neurons
- Marker genes satisfy all three criteria: log10 fold change >= 0.25, Wilcoxon p < 0.05, and expression in >= 25% of cells
- Pseudo markers unique to the one-against-all strategy (e.g., CCL22, SWAP70, KLRF1 in Gierahn clusters 1, 3, 4) are absent from scCATCH's paired-comparison gene sets
- Output annotation table contains one top-1 cell type per input cluster with no missing assignments

## Limitations

- Performance majorly depends on the reference database; a limited reference database potentially led to incorrect annotation of cell types via scCATCH
- Effects of multiple clustering algorithms on cell type annotation were not evaluated (deferred to future work)
- No software versions stated for compared tools (CellAssign, Garnett, SingleR, scMap, CHETAH)
- No accession IDs or hosting URLs given for the CellMatch database beyond the scCATCH package data

## Evidence
