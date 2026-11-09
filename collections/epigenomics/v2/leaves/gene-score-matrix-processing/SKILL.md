---
name: gene-score-matrix-processing
description: Use when input is scATAC-seq raw data (fragment file or bam file) or
  gene score matrices, and you need to build Cellcano-ready reference/target matrices
  with F-test-selected features before supervised celltyping.
when_to_use_negative:
- Input is already an F-test-filtered feature table or 'selected_features.csv' exists
  with the 3000 genes
- The input is single-cell gene expression (scRNA-seq) data rather than scATAC-seq-derived
  gene scores
- You need cross-modality prediction (scRNA-seq reference for scATAC-seq target) —
  the article states this is planned but not implemented in Cellcano
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ArchR
  role: Processes raw scATAC-seq data (fragment or bam files) to obtain gene scores
    for both reference and target datasets
  repo: https://www.archrproject.com/
- name: Cellcano
  role: Consumes the gene score matrices and applies an F-test on reference gene scores
    to select cell-type-specific features for the MLP/KD models
  repo: https://marvinquiet.github.io/Cellcano/
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:76878177c943f2691a22e749e9dfa5c57d2af3dbb698905ae68bd5ae76977d7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-score-matrix-processing

## Summary

Prepare scATAC-seq gene score matrices for Cellcano celltyping by converting raw fragment/bam files into gene scores with ArchR and selecting cell-type-specific features from the reference matrix via an F-test (top 3000 genes).

## When to use

Use when input is scATAC-seq raw data (fragment file or bam file) or gene score matrices, and you need to build Cellcano-ready reference/target matrices with F-test-selected features before supervised celltyping.

## When NOT to use

- Input is already an F-test-filtered feature table or 'selected_features.csv' exists with the 3000 genes
- The input is single-cell gene expression (scRNA-seq) data rather than scATAC-seq-derived gene scores
- You need cross-modality prediction (scRNA-seq reference for scATAC-seq target) — the article states this is planned but not implemented in Cellcano

## Inputs

- Raw scATAC-seq fragment file or bam file (reference and/or target)
- Reference scATAC-seq gene score matrix
- Reference metadata file with known cell-type labels
- Target scATAC-seq gene score matrix

## Outputs

- Gene score matrix for the reference dataset (ArchR output)
- Gene score matrix for the target dataset (ArchR output)
- selected_features.csv: 3000 cell-type-specific genes with a gene identifier field

## How to apply

Process raw scATAC-seq data (fragment or bam files) through the ArchR pipeline to obtain gene score matrices for BOTH the reference and target datasets, since Cellcano was built to consume gene scores rather than bin counts. Load the reference gene score matrix together with its metadata of known cell-type labels, then apply an F-test to each gene testing for score differences across the annotated cell types. Rank all genes by F-statistic in descending order and retain the top 3000 as cell-type-specific features for model construction, writing them to 'selected_features.csv'. The choice of gene scores over bin counts is supported by the article's finding that gene scores work as equally well or better than bin counts as input. The same feature set feeds the first-round MLP training and, downstream, the two-round MLP/KD prediction.

## Related tools

- **ArchR** (Processes raw scATAC-seq data (fragment or bam files) to obtain gene scores for both reference and target datasets) — https://www.archrproject.com/
- **Cellcano** (Consumes the gene score matrices and applies an F-test on reference gene scores to select cell-type-specific features for the MLP/KD models) — https://marvinquiet.github.io/Cellcano/

## Examples

```
pip install Cellcano
```

## Evaluation signals

- 'selected_features.csv' exists, contains exactly 3000 gene rows, and includes the required gene identifier field
- Gene score matrices were generated for BOTH reference and target datasets, not just one
- Genes are ranked by descending F-statistic before truncating to the top 3000
- Reference gene score matrix rows align with the cell-type labels in the reference metadata file
- Downstream prediction output predict_celltypes.csv has exactly three columns (pred_celltype, firstround_pred_celltype, entropy) with one labeled row per target cell

## Limitations

- If the input is scATAC-seq raw data (fragment file or bam file), the R package ArchR must be installed as an additional dependency
- Cellcano has not been tested on Apple M1 hardware, which may block running the ArchR/Cellcano pipeline there
- Cellcano is designed for scATAC-seq references; cross-modality (scRNA-seq reference) and multimodal-reference celltyping are future directions not yet implemented

## Evidence

- [intro] ArchR generates gene scores for both datasets: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [intro] F-test feature selection on reference gene scores: "Cellcano applies F-test on reference gene scores to select cell-type-specific genes as features for model construction31"
- [discussion] Gene scores vs bin counts input rationale: "We show that using gene scores works as equally well or even better than using bin counts as input."
- [readme] README ArchR dependency for raw input: [evidence span withheld — anchor status: external_doc]
- [intro] Two-round algorithm consumes the prepared matrices: "Cellcano implements a two-round supervised learning algorithm. It first trains a multi-layer perceptron (MLP) on the reference dataset and predicts cell types in target data."
