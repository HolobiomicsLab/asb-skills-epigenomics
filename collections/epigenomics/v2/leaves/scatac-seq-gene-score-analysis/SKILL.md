---
name: scatac-seq-gene-score-analysis
description: Use when preparing a reference scATAC-seq gene score matrix (with known
  cell type labels) for supervised celltyping and you must reduce thousands of genes
  to a compact, informative feature set.
when_to_use_negative:
- Input is already a curated feature set or peak/bin count matrix rather than a gene
  score matrix needing F-test feature selection
- The reference data lacks cell type labels, since the F-test requires annotated groups
  to test score differences across cell types
- The task is cross-modality label transfer from scRNA-seq (e.g., Seurat/scJoint-style),
  which Cellcano's gene-score-based supervised pipeline is not designed for
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Cellcano
  role: Implements the F-test feature selection on reference gene scores and downstream
    two-round supervised celltyping
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Processes raw scATAC-seq data to obtain gene scores for reference and target
    datasets prior to feature selection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:8c24350e73f2948525eb40e9ce94e120a761a1ee5afdec36ff284da306d0bdd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# scatac-seq-gene-score-analysis

## Summary

Selects cell-type-specific gene features from a reference scATAC-seq gene score matrix via an F-test, ranking genes by F-statistic and retaining the top 3000 as model-building features, as implemented in Cellcano prior to its two-round supervised learning.

## When to use

Use when preparing a reference scATAC-seq gene score matrix (with known cell type labels) for supervised celltyping and you must reduce thousands of genes to a compact, informative feature set.

## When NOT to use

- Input is already a curated feature set or peak/bin count matrix rather than a gene score matrix needing F-test feature selection
- The reference data lacks cell type labels, since the F-test requires annotated groups to test score differences across cell types
- The task is cross-modality label transfer from scRNA-seq (e.g., Seurat/scJoint-style), which Cellcano's gene-score-based supervised pipeline is not designed for

## Inputs

- Reference scATAC-seq gene score matrix (e.g., cells x genes matrix from ArchR gene activity scoring)
- Metadata file with known cell type labels for the reference cells

## Outputs

- selected_features.csv containing exactly 3000 gene identifiers ranked by F-statistic

## How to apply

Load the reference gene score matrix together with its metadata file of annotated cell type labels, both typically produced from raw data by the ArchR pipeline. Apply an F-test to each gene, testing whether gene scores differ across the annotated cell type labels in Cellcano's feature-selection step. Rank all genes by their F-statistic in descending order and retain the top 3000 genes as cell-type-specific features. Write the selected genes to 'selected_features.csv' with a gene identifier field, then feed these features into Cellcano's first-round MLP training on reference gene scores. The rationale is that genes with strong between-cell-type score variance are the informative features for discriminative celltyping models.

## Related tools

- **Cellcano** (Implements the F-test feature selection on reference gene scores and downstream two-round supervised celltyping) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Processes raw scATAC-seq data to obtain gene scores for reference and target datasets prior to feature selection)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- selected_features.csv exists and contains exactly 3000 gene rows
- The file includes the required gene identifier field for every selected gene
- Selected genes correspond to the highest F-statistics when recomputing the F-test across the annotated cell type labels
- Downstream MLP trained on the 3000 selected features yields valid first-round cell type predictions for anchor selection

## Limitations

- Feature selection depends on the quality and granularity of the reference cell type labels; the F-test can only separate labels present in the metadata
- Cellcano's gene-score-based supervised design assumes an scATAC-seq reference; cross-modality prediction using scRNA-seq as reference is planned but not implemented
- Cellcano requires the ArchR R package if starting from raw scATAC-seq data (fragment or bam files), and a Python stack including tensorflow 2.7.1, anndata 0.7.4, and scanpy 1.8.2

## Evidence

- [intro] F-test feature selection on reference gene scores: "Cellcano applies F-test on reference gene scores to select cell-type-specific genes as features for model construction31"
- [intro] ArchR produces gene scores: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [discussion] Gene scores as input perform well: "We show that using gene scores works as equally well or even better than using bin counts as input."
- [readme] Cellcano dependencies and raw-data requirement: [evidence span withheld — anchor status: external_doc]
