---
name: single-cell-annotation-benchmarking
description: Use when evaluating or comparing scRNA-seq cell type annotation methods
  at the cluster level on benchmark datasets (Chen, Xin, Gierahn), or when reproducing
  the scCATCH vs CellAssign/Garnett/SingleR/scMap/CHETAH consistent-rate table.
when_to_use_negative:
- The goal is novel biological annotation of a new dataset rather than benchmarking
  method accuracy against a curated reference
- No manually curated reference cell types are available for the dataset, so consistent
  rates cannot be computed
- The evaluation target is marker-gene identification quality (where Seurat is the
  comparator) rather than cluster-level annotation accuracy
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Cluster-based annotation method under benchmark; run via createscCATCH, findmarkergene,
    and findcelltype on each dataset
  repo: https://github.com/ZJUFanLab/scCATCH
- name: Seurat
  role: Provides cluster labels/data processing for benchmark datasets; scCATCH objects
    can be created from Seurat objects
- name: SingleR
  role: Cell-based baseline annotating each cell against reference profiles, aggregated
    to cluster level for consistent-rate scoring
- name: CellAssign
  role: Cell-based baseline annotating each cell against reference profiles, aggregated
    to cluster level for consistent-rate scoring
- name: Garnett
  role: Cell-based baseline annotating each cell, aggregated to cluster level for
    consistent-rate scoring
- name: scMap
  role: Cell-based baseline mapping expression profiles to reference cell types, aggregated
    to cluster level for consistent-rate scoring
- name: CHETAH
  role: Cell-based baseline annotating each cell, aggregated to cluster level for
    consistent-rate scoring
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:bea4ca0541cc5d90c986497540af6ed4b65998b9f64c25680ef1072e76fe96fc
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
  - CellAssign
  - Garnett
  - Seurat
  - SingleR
  - scCATCH
  - scMap
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-annotation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the cluster-level consistent-rate comparison of scCATCH against cell-based annotation methods (CellAssign, Garnett, SingleR, scMap, CHETAH) on the Chen, Xin, and Gierahn benchmark datasets, following the scCATCH_performance_comparison workflow and Table 1 conventions.

## When to use

Use when evaluating or comparing scRNA-seq cell type annotation methods at the cluster level on benchmark datasets (Chen, Xin, Gierahn), or when reproducing the scCATCH vs CellAssign/Garnett/SingleR/scMap/CHETAH consistent-rate table.

## When NOT to use

- The goal is novel biological annotation of a new dataset rather than benchmarking method accuracy against a curated reference
- No manually curated reference cell types are available for the dataset, so consistent rates cannot be computed
- The evaluation target is marker-gene identification quality (where Seurat is the comparator) rather than cluster-level annotation accuracy

## Inputs

- Expression matrices of the three benchmark scRNA-seq datasets (Chen, 203 cells; Xin, 1,600 cells; Gierahn, 3,694 cells)
- Pre-computed cluster labels for each benchmark dataset (e.g. Seurat ident vectors)
- Manually curated reference cell type labels per dataset for consistency scoring
- CellMatch reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references) from the scCATCH package

## Outputs

- Cluster-level cell type annotations per method per dataset
- Per-cluster and overall consistent rates for scCATCH, CellAssign, Garnett, SingleR, scMap, and CHETAH
- A consistent-rate results table mirroring Table 1 (e.g. table1_consistent_rates.csv)

## How to apply

Install scCATCH from CRAN (install.packages('scCATCH')) or via devtools::install_github('ZJUFanLab/scCATCH'), and pull benchmark data and pre-computed cluster labels from ZJUFanLab/scCATCH_performance_comparison. For each dataset (Chen, 203 cells; Xin, 1,600 cells; Gierahn, 3,694 cells), build an scCATCH object with createscCATCH(data, cluster), run findmarkergene() with species/marker/tissue/cancer settings, then findcelltype() to obtain cluster-level annotations. In parallel, run the cell-based baselines (CellAssign, Garnett, SingleR, scMap, CHETAH) on the same datasets to get per-cell annotations, aggregating to cluster level where required. Count a cluster as consistent when the assigned type matches the manually curated reference, and compute per-cluster and overall consistent rates for all six methods. Assemble the rates into a table mirroring Table 1 (scCATCH 100%/100%/100%; CellAssign 46%/99%/41%; Garnett 15%/22%/0.9%; SingleR 81%/95%/10%; scMap 75%/89%/69%; CHETAH 0%/0%/0% on Chen/Xin/Gierahn) and validate reproduction within rounding tolerance.

## Related tools

- **scCATCH** (Cluster-based annotation method under benchmark; run via createscCATCH, findmarkergene, and findcelltype on each dataset) — https://github.com/ZJUFanLab/scCATCH
- **Seurat** (Provides cluster labels/data processing for benchmark datasets; scCATCH objects can be created from Seurat objects)
- **SingleR** (Cell-based baseline annotating each cell against reference profiles, aggregated to cluster level for consistent-rate scoring)
- **CellAssign** (Cell-based baseline annotating each cell against reference profiles, aggregated to cluster level for consistent-rate scoring)
- **Garnett** (Cell-based baseline annotating each cell, aggregated to cluster level for consistent-rate scoring)
- **scMap** (Cell-based baseline mapping expression profiles to reference cell types, aggregated to cluster level for consistent-rate scoring)
- **CHETAH** (Cell-based baseline annotating each cell, aggregated to cluster level for consistent-rate scoring)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Reproduced scCATCH consistent rates equal 100%, 100%, and 100% on Chen, Xin, and Gierahn respectively
- Reproduced baseline consistent rates match Table 1 values within rounding tolerance (e.g. SingleR 81%/95%/10%; CHETAH 0%/0%/0%)
- The output table contains exactly one consistent rate per method (6 methods) per dataset (3 datasets), i.e. 18 cells
- A cluster is counted consistent only when the assigned cell type matches the manually curated reference label
- Annotation counts reflect the dataset sizes: 203 (Chen), 1,600 (Xin), and 3,694 (Gierahn) cells

## Limitations

- The performance of scCATCH majorly depends on the reference database; a limited reference database potentially led to incorrect annotation of cell types
- No software versions are stated for the compared tools (CellAssign, Garnett, SingleR, scMap, CHETAH), affecting strict reproducibility
- Cell-based baselines show high variability across datasets (e.g. Garnett and CHETAH at 0% consistent on Gierahn), so conclusions do not transfer uniformly across tissues

## Evidence

- [readme] Repository containing source code and results of performance comparison among scCATCH, CellAssign, Garnett, SingleR, scMap, CHETAH, and CellMatch database: [evidence span withheld — anchor status: not_found]
