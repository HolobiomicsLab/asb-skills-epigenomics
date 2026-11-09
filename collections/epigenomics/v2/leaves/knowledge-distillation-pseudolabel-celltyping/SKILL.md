---
name: knowledge-distillation-pseudolabel-celltyping
description: 'Use when annotating scATAC-seq cells with an scATAC-seq reference: gene
  score matrices (e.g., from ArchR) exist for both reference and target, and two-round
  prediction with entropy-based anchor selection is feasible (default when target
  has > 1,000 cells).'
when_to_use_negative:
- The reference is scRNA-seq (cross-modality label transfer, as with Seurat/scJoint)
  — Cellcano is designed to use scATAC-seq as reference; cross-modality use is only
  a planned future direction
- Input is raw fragment/bam data without gene scores — ArchR must first generate gene
  scores before Cellcano can run
- The task is unsupervised clustering of scATAC-seq data rather than reference-supervised
  cell type identification
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: Cellcano
  role: 'Implements the two-round supervised learning algorithm: MLP first round,
    entropy-based anchor selection, and self-Knowledge Distiller second round on gene
    scores'
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Upstream preprocessing to convert raw scATAC-seq fragment/bam data into gene
    score matrices for both reference and target
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:6823a93e5fd6e365539629fc46e7c891369f90f71b6ea85fe268bb63c1cfd22d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Knowledge-distillation pseudolabel celltyping

## Summary

Cellcano's two-round supervised celltyping for scATAC-seq: train an MLP on reference gene scores, select low-entropy target cells as anchors, then train a self-Knowledge Distiller (KD) model on anchor pseudo labels to predict the remaining cells. The second round alleviates distributional shift between reference and target data and improves prediction performance.

## When to use

Use when annotating scATAC-seq cells with an scATAC-seq reference: gene score matrices (e.g., from ArchR) exist for both reference and target, and two-round prediction with entropy-based anchor selection is feasible (default when target has > 1,000 cells).

## When NOT to use

- The reference is scRNA-seq (cross-modality label transfer, as with Seurat/scJoint) — Cellcano is designed to use scATAC-seq as reference; cross-modality use is only a planned future direction
- Input is raw fragment/bam data without gene scores — ArchR must first generate gene scores before Cellcano can run
- The task is unsupervised clustering of scATAC-seq data rather than reference-supervised cell type identification

## Inputs

- Reference gene score matrix (e.g., ArchR-derived) with cell-type labels metadata
- Target gene score matrix (e.g., human PBMC demo target gene scores)
- Reference cell-type metadata table

## Outputs

- Per-cell prediction table (e.g., predict_celltypes.csv) with columns pred_celltype, firstround_pred_celltype, and entropy (one row per target cell)
- Per-cell first-round prediction entropies used for anchor selection

## How to apply

Process raw scATAC-seq data into gene scores with the ArchR pipeline, then apply an F-test on reference gene scores to select cell-type-specific genes as features. In the first round, train a multi-layer perceptron (MLP) on the reference gene score matrix with cell-type labels and predict cell types for target cells. Compute prediction entropy per target cell from first-round probabilities and select low-entropy cells as anchors (these carry firstround_pred_celltype and entropy values). In the second round, train the self-Knowledge Distiller (KD) model on anchor cells using predicted pseudo labels, then apply the trained KD model to predict cell types for the remaining non-anchor cells. Assemble the final per-cell table with columns pred_celltype, firstround_pred_celltype, and entropy; with default two-round prediction (or target cell number larger than 1,000) the output table includes the first-round prediction column.

## Related tools

- **Cellcano** (Implements the two-round supervised learning algorithm: MLP first round, entropy-based anchor selection, and self-Knowledge Distiller second round on gene scores) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Upstream preprocessing to convert raw scATAC-seq fragment/bam data into gene score matrices for both reference and target)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- predict_celltypes.csv exists and contains exactly the three required columns (pred_celltype, firstround_pred_celltype, entropy)
- One labeled prediction exists for every target cell in the demo target gene score matrix
- Anchor cells are exactly those target cells with the lowest first-round prediction entropies, and their pred_celltype equals firstround_pred_celltype
- Gene score features were selected via F-test on reference gene scores (cell-type-specific genes only)
- Cellcano's predictions outperform baseline label-transfer methods on benchmark celltyping tasks (e.g., the 50 human PBMC and mouse brain tasks)

## Limitations

- Cellcano requires an scATAC-seq reference; cross-modality prediction using scRNA-seq as reference is planned but not implemented
- Using multimodal (jointly scRNA- and scATAC-seq) reference data is an open future direction, not yet implemented
- Not tested on Apple M1 hardware (per README Issue #6)
- If raw scATAC-seq data (fragment or bam file) are the input, the R package ArchR must additionally be installed

## Evidence

- [abstract] two-round algorithm: "Here we develop Cellcano, a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data."
- [intro] first round MLP: "In the first round, Cellcano trains an MLP model with reference gene scores and predicts cell types in target data."
- [intro] entropy anchor selection: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [intro] second round KD: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells."
- [abstract] rationale: "The method alleviates the distributional shift between reference and target data and improves the prediction performance."
- [readme] installation: [evidence span withheld — anchor status: external_doc]
