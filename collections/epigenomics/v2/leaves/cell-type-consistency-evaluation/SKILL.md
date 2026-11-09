---
name: cell-type-consistency-evaluation
description: Use when comparing cell type annotation accuracy of cluster-based vs
  cell-based methods on scRNA-seq datasets with known reference labels, or when quantifying
  how swapping the underlying reference database affects annotation consistency.
when_to_use_negative:
- Input scRNA-seq data has no clusters or no manually curated reference cell types
  to compare against — consistency rates cannot be computed
- Goal is de novo cell type discovery or annotation of tissues with no markers in
  the CellMatch database, rather than evaluating annotation accuracy against known
  labels
- Datasets are not human or mouse — CellMatch contains only human and mouse references
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCATCH
  role: Cluster-based automatic annotation via findmarkergene() and findcelltype();
    provides the CellMatch database
  repo: https://github.com/ZJUFanLab/scCATCH
- name: SingleR
  role: Cell-based baseline annotator; re-run with CellMatch as reference to test
    reference-database sensitivity
- name: CHETAH
  role: Cell-based baseline annotator; re-run with CellMatch-derived profiles to test
    reference sensitivity
- name: CellAssign
  role: Cell-based baseline annotator compared in the consistent-rate table
- name: Garnett
  role: Cell-based baseline annotator compared in the consistent-rate table
- name: scMap
  role: Cell-based baseline annotator compared in the consistent-rate table
- name: Seurat
  role: Preprocessing and clustering pipeline supplying expression matrices and cluster
    labels
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100882
    title: 'scCATCH: Automatic Annotation on Cell Types of Clusters from Single-Cell
      RNA Sequencing Data'
  source_content_hash: sha256:f61b93fc2463762681ce21853db0ffe31306f432d2f1dfa46874372cdd673b8a
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
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cell-type-consistency-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell type annotation methods (scCATCH, CellAssign, Garnett, SingleR, scMap, CHETAH) by computing cluster-level consistent rates against manually curated reference cell types on benchmark scRNA-seq datasets, and test how reference databases (e.g., CellMatch) change accuracy.

## When to use

Use when comparing cell type annotation accuracy of cluster-based vs cell-based methods on scRNA-seq datasets with known reference labels, or when quantifying how swapping the underlying reference database affects annotation consistency.

## When NOT to use

- Input scRNA-seq data has no clusters or no manually curated reference cell types to compare against — consistency rates cannot be computed
- Goal is de novo cell type discovery or annotation of tissues with no markers in the CellMatch database, rather than evaluating annotation accuracy against known labels
- Datasets are not human or mouse — CellMatch contains only human and mouse references

## Inputs

- scRNA-seq expression matrix (e.g., Chen 203 cells, Xin 1,600 cells, Gierahn 3,694 cells) with pre-computed cluster labels
- Manually curated reference cell type labels per dataset/cluster
- CellMatch tissue-specific cell taxonomy reference database (from the scCATCH package)
- Reference profiles for cell-based baselines (e.g., ImmGen/mouse RNA-seq, HPCA, Encode/Blueprint Epigenomics)

## Outputs

- Per-method, per-dataset consistent-rate table mirroring Table 1 (e.g., table1_consistent_rates.csv)
- CellMatch substitution consistency results (e.g., cellmatch_reference_comparison_results.csv) with one consistency rate per method per dataset
- Per-cluster annotation assignments for each method

## How to apply

For each benchmark dataset (Chen, 203 cells; Xin, 1,600 cells; Gierahn, 3,694 cells), build the scCATCH object with createscCATCH() from the expression matrix and cluster labels, run findmarkergene() with species/marker/tissue settings and findcelltype() to get cluster-level annotations. Run the cell-based baselines (CellAssign, Garnett, SingleR, scMap, CHETAH) on the same datasets to obtain per-cell annotations, aggregating to cluster level where required. Count a cluster as consistent when the assigned type matches the manually curated reference, and compute per-cluster and overall consistent rates for all methods. As a reference-sensitivity check, substitute CellMatch (353 cell types, 686 subtypes, 184 tissue types, 2,096 human and mouse references) as the underlying reference for SingleR and CHETAH and recompute the fraction of clusters whose annotations are consistent with the original-reference annotations. Expected Table 1 rates: scCATCH 100%/100%/100%; CellAssign 46%/99%/41%; Garnett 15%/22%/0.9%; SingleR 81%/95%/10%; scMap 75%/89%/69%; CHETAH 0%/0%/0% on Chen/Xin/Gierahn.

## Related tools

- **scCATCH** (Cluster-based automatic annotation via findmarkergene() and findcelltype(); provides the CellMatch database) — https://github.com/ZJUFanLab/scCATCH
- **SingleR** (Cell-based baseline annotator; re-run with CellMatch as reference to test reference-database sensitivity)
- **CHETAH** (Cell-based baseline annotator; re-run with CellMatch-derived profiles to test reference sensitivity)
- **CellAssign** (Cell-based baseline annotator compared in the consistent-rate table)
- **Garnett** (Cell-based baseline annotator compared in the consistent-rate table)
- **scMap** (Cell-based baseline annotator compared in the consistent-rate table)
- **Seurat** (Preprocessing and clustering pipeline supplying expression matrices and cluster labels)

## Examples

```
obj <- createscCATCH(data = sc_data, cluster = sc_cluster); obj <- findmarkergene(obj, species, marker, tissue, cancer); obj <- findcelltype(obj)
```

## Evaluation signals

- Reproduced scCATCH consistent rates must equal 100% on Chen, Xin, and Gierahn; all other method rates must match Table 1 within rounding tolerance
- Substituting CellMatch must raise SingleR's consistent rate from 0% to the 80%-90% range on validation datasets
- CHETAH annotations must remain unchanged after CellMatch substitution (no difference in consistent rate)
- Results CSV must contain exactly one consistent-rate value per method per dataset
- Consistent rate is computed as the fraction of clusters whose assigned type matches the manually curated reference

## Limitations

- scCATCH performance majorly depends on the reference database; a limited reference database can lead to incorrect annotation
- No software versions were stated for the compared tools, complicating exact reproduction
- Cell-based methods struggle when multiple cell types are present in one cluster; Garnett and CHETAH achieved consistent rates as low as 0% on the Gierahn dataset
- Effects of multiple clustering algorithms on cell type annotation were not evaluated

## Evidence

- [results] Cell-based baselines showed poor and variable accuracy: Garnett and CHETAH achieved consistent rates as low as 0% on the Gierahn dataset, meaning none of the cells were accurately identified by these two methods.: [evidence span withheld — anchor status: not_found]
- [discussion] Application of CellMatch to SingleR and CellAssign improved accuracies: "Application of CellMatch to SingleR and CellAssign resulted in significant improvement in accuracies of cell type annotations"
- [methods] CellMatch size from methods: "CellMatch includes a panel of 353 cell types and related 686 subtypes associated with 184 tissue types, and 2,096 references of human and mouse."
- [intro] Marker gene criteria: "genes significantly upregulated in only one cluster (log10 fold change ≥0.25, p < 0.05) and expressed in more than a quarter of cells (≥25%) would be considered marker genes"
- [readme] README install instructions: "install.packages("scCATCH")"
- [readme] Performance comparison repo description: [evidence span withheld — anchor status: not_found]
