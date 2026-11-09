---
name: r-package-pipeline-execution
description: Use when a preprocessed scRNA-seq expression matrix with cluster assignments
  needs automated cell type annotation via an R package pipeline (e.g., scCATCH with
  CellMatch), following the article's documented function sequence.
when_to_use_negative:
- Input is not clustered scRNA-seq data (e.g., already-annotated cells or bulk RNA-seq)
  — the pipeline requires cluster-level input via createscCATCH
- Species markers outside human/mouse are needed without adapting the cellmatch reference
  — the standard database covers 2,096 human and mouse references
- Marker-gene identification rather than cell type annotation is the goal — Seurat's
  one-against-all approach or other marker tools may be more appropriate
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Core R package whose createscCATCH / findmarkergene / findcelltype functions
    implement the pipeline being executed
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Optional upstream source of processed scRNA-seq data; its RNA assay data slot
    and active identities feed createscCATCH
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:bed40b0c69f8177ff155b827ff15d241bd6fb3af388092ad3b81d2c46a5e9758
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
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# r-package-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes an R package's documented two-function pipeline (here scCATCH's findmarkergene() then findcelltype()) end-to-end on a single-cell RNA-seq dataset to produce automatic cell type annotations per cluster, reproducing the article's reported workflow and consistency metrics.

## When to use

Use when a preprocessed scRNA-seq expression matrix with cluster assignments needs automated cell type annotation via an R package pipeline (e.g., scCATCH with CellMatch), following the article's documented function sequence.

## When NOT to use

- Input is not clustered scRNA-seq data (e.g., already-annotated cells or bulk RNA-seq) — the pipeline requires cluster-level input via createscCATCH
- Species markers outside human/mouse are needed without adapting the cellmatch reference — the standard database covers 2,096 human and mouse references
- Marker-gene identification rather than cell type annotation is the goal — Seurat's one-against-all approach or other marker tools may be more appropriate

## Inputs

- Preprocessed scRNA-seq expression matrix (sc_data)
- Character vector of cluster assignments (sc_cluster) or a Seurat object
- Species, tissue, and cancer selections for the CellMatch reference (with optional custom cellmatch)

## Outputs

- scCATCH object with identified marker genes per cluster (findmarkergene output)
- Top-1 cell type annotation per cluster (findcelltype output)
- Per-cluster annotation table comparable to manually annotated cell types (e.g., table2_consistency_rates.csv)

## How to apply

Install the R package (scCATCH via install.packages('scCATCH') or devtools::install_github('ZJUFanLab/scCATCH'); requires R >4.0). Build the package object with createscCATCH(data = sc_data, cluster = sc_cluster), where sc_data is the scRNA-seq matrix and sc_cluster the cluster assignments (a Seurat object can be converted with Seurat_obj[['RNA']]@data and as.character(Idents(Seurat_obj))). Run findmarkergene(obj, species, marker, tissue, cancer) with the tissue-appropriate CellMatch reference — 353 cell types, 686 subtypes, 184 tissue types, 2,096 human/mouse references — applying marker criteria of log10 fold change ≥ 0.25, Wilcoxon p < 0.05, and expression in ≥ 25% of cells. Then run findcelltype(obj) to assign the top-1 scoring cell type per cluster based on validated marker genes and supporting literature. Select tissues/cancers via the documented wiki options; the pipeline can be validated against the scCATCH_performance_comparison repository (e.g., six datasets Enge, Wu, Lindsey, Zheng, Zeisel, Heng yielding 67%–100% cluster consistency, average 83%).

## Related tools

- **scCATCH** (Core R package whose createscCATCH / findmarkergene / findcelltype functions implement the pipeline being executed) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Optional upstream source of processed scRNA-seq data; its RNA assay data slot and active identities feed createscCATCH)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- The scCATCH object exists after createscCATCH and contains marker genes for every input cluster after findmarkergene
- Every cluster receives a top-1 cell type annotation after findcelltype
- Marker genes satisfy the thresholds: log10 fold change ≥ 0.25, Wilcoxon p < 0.05, expression in ≥ 25% of cells
- When run on the six benchmark datasets, per-cluster consistency rates fall in the 67%–100% range with a mean of 83% (expected counts: Enge 4/6, Wu 7/7, Lindsey 7/7, Zheng 7/9, Zeisel 10/10, Heng 9/12)
- Known misannotations reproduce (Zheng clusters 1–2 as naive/regulatory T cells; Heng clusters as neuroblasts/spiral ganglionic neurons), confirming faithful pipeline execution

## Limitations

- Performance depends on the reference database; a limited CellMatch reference can lead to incorrect cell type annotations
- Effects of multiple clustering algorithms on cell type annotation were not evaluated
- No software versions stated for compared tools in the discussion

## Evidence

- [readme] Create scCATCH object from Seurat object: [evidence span withheld — anchor status: not_found]
