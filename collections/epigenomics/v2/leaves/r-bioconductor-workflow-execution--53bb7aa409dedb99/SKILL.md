---
name: r-bioconductor-workflow-execution
description: Use when annotating cell types of clusters in an scRNA-seq dataset in
  R, or when reproducing the scCATCH/SingleR/CHETAH performance comparison from the
  scCATCH_performance_comparison repository.
when_to_use_negative:
- The input is not a clustered scRNA-seq dataset — scCATCH annotates pre-identified
  clusters via marker genes, so it cannot run without cluster labels.
- The task is single-cell-level annotation with cell-based methods only; cluster-based
  annotation via scCATCH is a different strategy (though SingleR/scMap/CHETAH are
  cell-based and are run here only for benchmarking).
- The species' markers are outside human or mouse unless a custom cellmatch table
  with markers from other species is supplied via the wiki-documented customization.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: scCATCH
  role: Core R package that performs cluster marker gene identification (findmarkergene)
    and evidence-scored cell type annotation (findcelltype) against CellMatch
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Provides pre-processed scRNA-seq objects and cluster identities that can be
    fed directly into createscCATCH
- name: SingleR
  role: Cell-based annotation method benchmarked in the comparison workflow with CellMatch
    substituted as the reference
- name: CHETAH
  role: Cell-based annotation method benchmarked in the comparison workflow using
    CellMatch-derived profiles as the reference
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:95091140f7489b02faa7bd7555212410d6b9b37a59e2f0a2d84bd7411c228588
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
  - Seurat
  - SingleR
  - scCATCH
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# r-bioconductor-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute multi-method cell type annotation workflows (scCATCH, SingleR, CHETAH) in R, installing tools from CRAN/GitHub via devtools and running the scCATCH functions createscCATCH, findmarkergene, and findcelltype on scRNA-seq matrices with cluster labels.

## When to use

Use when annotating cell types of clusters in an scRNA-seq dataset in R, or when reproducing the scCATCH/SingleR/CHETAH performance comparison from the scCATCH_performance_comparison repository.

## When NOT to use

- The input is not a clustered scRNA-seq dataset — scCATCH annotates pre-identified clusters via marker genes, so it cannot run without cluster labels.
- The task is single-cell-level annotation with cell-based methods only; cluster-based annotation via scCATCH is a different strategy (though SingleR/scMap/CHETAH are cell-based and are run here only for benchmarking).
- The species' markers are outside human or mouse unless a custom cellmatch table with markers from other species is supplied via the wiki-documented customization.

## Inputs

- scRNA-seq expression data matrix (sc_data)
- Character vector of cluster labels (sc_cluster) or a Seurat object with ident clusters
- Species, marker, tissue, and cancer arguments for findmarkergene()
- CellMatch reference database bundled with the scCATCH package (data/)

## Outputs

- scCATCH object annotated with a cell type (top-1 score) per cluster
- Per-method annotation labels for SingleR and CHETAH runs on validation datasets
- Consistency-rate results table (e.g., cellmatch_reference_comparison_results.csv)

## How to apply

Install R (>= 4.0) and scCATCH either from CRAN (install.packages('scCATCH')) or from GitHub with devtools::install_github('ZJUFanLab/scCATCH'). Build a scCATCH object from the expression matrix and cluster labels with createscCATCH(data = sc_data, cluster = sc_cluster); a Seurat object can be converted via obj <- createscCATCH(data = Seurat_obj[['RNA']]@data, cluster = as.character(Idents(Seurat_obj))). Then run findmarkergene(obj, species, marker, tissue, cancer) to identify cluster markers — genes significantly upregulated in only one cluster (log10 fold change >= 0.25, p < 0.05) and expressed in >= 25% of cells — followed by findcelltype(obj) to score cell types against the CellMatch database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references) and assign the top-1 type per cluster. For the comparison workflow, load the three validation datasets with pre-computed cluster labels from the scCATCH_performance_comparison repository and run SingleR and CHETAH with CellMatch substituted as the underlying reference. Decision points include selecting species, tissue, and cancer arguments, and optionally supplying a custom cellmatch table.

## Related tools

- **scCATCH** (Core R package that performs cluster marker gene identification (findmarkergene) and evidence-scored cell type annotation (findcelltype) against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Provides pre-processed scRNA-seq objects and cluster identities that can be fed directly into createscCATCH)
- **SingleR** (Cell-based annotation method benchmarked in the comparison workflow with CellMatch substituted as the reference)
- **CHETAH** (Cell-based annotation method benchmarked in the comparison workflow using CellMatch-derived profiles as the reference)

## Examples

```
install.packages("scCATCH"); obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- scCATCH accurately annotated 67%-100% (average, 83%) of clusters in six published scRNA-seq datasets — cluster-level annotation accuracy should fall in this range for comparable data.
- Marker genes must satisfy log10 fold change >= 0.25, p < 0.05 (Wilcoxon test), and be expressed in >= 25% of cells.
- In the CellMatch-substitution experiment, SingleR's consistent rate should rise from 0% to the 80%-90% range while CHETAH's annotations remain unchanged.
- The results table cellmatch_reference_comparison_results.csv should contain one consistency rate per method per validation dataset.
- Each cluster in the scCATCH object should receive exactly one top-1 cell type annotation derived from the highest evidence-based score.

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited reference database can lead to incorrect annotation of cell types.
- No software versions were stated for the compared tools (CellAssign, Garnett, SingleR, scMap, CHETAH), complicating exact reproduction.
- Effects of multiple clustering algorithms on cell type annotation were not evaluated and were deferred to future work.
- Available tissues and cancers for findmarkergene are restricted to a documented list on the scCATCH wiki page.

## Evidence
