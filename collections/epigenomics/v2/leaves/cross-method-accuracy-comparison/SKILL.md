---
name: cross-method-accuracy-comparison
description: Use when a new cell type annotation method must be quantitatively compared
  to established baselines on shared scRNA-seq datasets, with accuracy expressed as
  consistent rates against curated reference cell types.
when_to_use_negative:
- The goal is per-cell rather than per-cluster annotation accuracy, since scCATCH
  and this comparison operate at the cluster level
- No manually curated reference cell types are available, so consistent rates cannot
  be computed against a ground truth
- The tissue/species of the dataset is not covered by the CellMatch reference database,
  which can itself cause incorrect annotation
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Cluster-based annotation method under evaluation; provides createscCATCH(),
    findmarkergene(), and findcelltype() to produce cluster-level cell types from
    marker genes scored against CellMatch
  repo: https://github.com/ZJUFanLab/scCATCH
- name: CellMatch
  role: Tissue-specific cellular taxonomy reference database underlying scCATCH's
    evidence-based scoring of marker genes
  repo: https://github.com/ZJUFanLab/scCATCH/tree/master/data
- name: Seurat
  role: Source of scRNA-seq data processing and cluster identities; scCATCH objects
    can be created directly from a Seurat object's RNA @data slot and Idents
- name: CellAssign
  role: Cell-based annotation baseline compared in the consistent-rate table
- name: Garnett
  role: Cell-based annotation baseline compared in the consistent-rate table
- name: SingleR
  role: Cell-based annotation baseline compared in the consistent-rate table
- name: scMap
  role: Cell-based annotation baseline compared in the consistent-rate table
- name: CHETAH
  role: Cell-based annotation baseline compared in the consistent-rate table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:51cdcef0b0e44a42d869234651745b1e6ed2fdb60a9290ab10e02a9415cbe945
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
  - CellMatch
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
yes_rate: 0.7857142857142857
n_claims: 14
---

# cross-method-accuracy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cluster-based cell type annotation method (scCATCH) against cell-based annotators (CellAssign, Garnett, SingleR, scMap, CHETAH) by computing per-cluster and overall consistent rates against manually curated reference cell types on benchmark datasets (Chen, 203 cells; Xin, 1,600 cells; Gierahn, 3,694 cells).

## When to use

Use when a new cell type annotation method must be quantitatively compared to established baselines on shared scRNA-seq datasets, with accuracy expressed as consistent rates against curated reference cell types.

## When NOT to use

- The goal is per-cell rather than per-cluster annotation accuracy, since scCATCH and this comparison operate at the cluster level
- No manually curated reference cell types are available, so consistent rates cannot be computed against a ground truth
- The tissue/species of the dataset is not covered by the CellMatch reference database, which can itself cause incorrect annotation

## Inputs

- scRNA-seq expression matrices from three benchmark datasets (Chen, 203 cells; Xin, 1,600 cells; Gierahn, 3,694 cells)
- Cluster labels per dataset (character vector, e.g. from Seurat Idents)
- Manually curated reference cell type annotations per dataset
- CellMatch tissue-specific cell taxonomy reference database (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references)
- Species, marker, tissue, and cancer settings for findmarkergene()

## Outputs

- Per-cluster and overall consistent rates for all six methods on Chen, Xin, and Gierahn
- table1_consistent_rates.csv mirroring Table 1 (scCATCH vs CellAssign, Garnett, SingleR, scMap, CHETAH)
- Cluster-level cell type annotations from scCATCH (findcelltype output) and per-cell annotations from each baseline

## How to apply

Run scCATCH on each benchmark dataset by building the object with createscCATCH() from the expression matrix and cluster labels, then findmarkergene() with species/marker/tissue/cancer settings and findcelltype() for cluster-level annotations; scCATCH considers genes upregulated in only one cluster (log10 fold change >= 0.25, p < 0.05, expressed in >= 25% of cells) as marker genes and scores cell types via the CellMatch database. Run the five cell-based baselines (CellAssign, Garnett, SingleR, scMap, CHETAH) on the same datasets to obtain per-cell annotations, aggregating to cluster level where required. For every method, count a cluster as consistent when its assigned type matches the manually curated reference type, and compute per-cluster and overall consistent rates. Assemble the rates into a Table-1-style comparison (scCATCH 100%/100%/100%; CellAssign 46%/99%/41%; Garnett 15%/22%/0.9%; SingleR 81%/95%/10%; scMap 75%/89%/69%; CHETAH 0%/0%/0% on Chen/Xin/Gierahn) and write to table1_consistent_rates.csv. Rationale: cluster-level annotation with a tissue-specific taxonomy database avoids the pseudo marker gene and multi-type-per-cluster problems that depress cell-based method accuracy.

## Related tools

- **scCATCH** (Cluster-based annotation method under evaluation; provides createscCATCH(), findmarkergene(), and findcelltype() to produce cluster-level cell types from marker genes scored against CellMatch) — https://github.com/ZJUFanLab/scCATCH
- **CellMatch** (Tissue-specific cellular taxonomy reference database underlying scCATCH's evidence-based scoring of marker genes) — https://github.com/ZJUFanLab/scCATCH/tree/master/data
- **Seurat** (Source of scRNA-seq data processing and cluster identities; scCATCH objects can be created directly from a Seurat object's RNA @data slot and Idents)
- **CellAssign** (Cell-based annotation baseline compared in the consistent-rate table)
- **Garnett** (Cell-based annotation baseline compared in the consistent-rate table)
- **SingleR** (Cell-based annotation baseline compared in the consistent-rate table)
- **scMap** (Cell-based annotation baseline compared in the consistent-rate table)
- **CHETAH** (Cell-based annotation baseline compared in the consistent-rate table)

## Examples

```
install.packages('scCATCH'); obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- The reproduced consistent rates for scCATCH on Chen, Xin, and Gierahn must equal 100%, 100%, and 100% respectively
- All baseline rates must match the reported Table 1 values within rounding tolerance (CellAssign 46%/99%/41%; Garnett 15%/22%/0.9%; SingleR 81%/95%/10%; scMap 75%/89%/69%; CHETAH 0%/0%/0%)
- A cluster is counted consistent only when its assigned type exactly matches the manually curated reference type; consistent rate = consistent clusters / total clusters per method per dataset
- Output CSV (table1_consistent_rates.csv) contains one consistent rate per method per dataset (6 methods x 3 datasets = 18 rates)
- Cell-based baselines are expected to show poor and variable accuracy, including near-0% consistent rates (Garnett 0.9%, CHETAH 0% on Gierahn), serving as a sanity check on the comparison pipeline

## Limitations

- scCATCH's performance majorly depends on the reference database; a limited CellMatch database can lead to incorrect annotation of cell types
- Software versions of CellAssign, Garnett, SingleR, scMap, and CHETAH are not stated in the paper, complicating exact reproduction
- Effects of multiple clustering algorithms on cell type annotation were not evaluated (deferred to future work)
- Cell-based baselines require aggregating per-cell annotations to cluster level, and their accuracy is poor and variable across datasets (e.g. Garnett and CHETAH as low as 0% on Gierahn)

## Evidence

- [results] Cell-based baselines showed poor and variable accuracy: Garnett and CHETAH achieved consistent rates as low as 0% on the Gierahn dataset, meaning none of the cells were accurately identified by these two methods.: [evidence span withheld — anchor status: not_found]
- [readme] Performance comparison repo (README): [evidence span withheld — anchor status: not_found]
