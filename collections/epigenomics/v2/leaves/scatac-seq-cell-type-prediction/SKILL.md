---
name: scatac-seq-cell-type-prediction
description: Use when both a labeled scATAC-seq reference dataset (gene score matrix
  + cell-type metadata) and an unlabeled target scATAC-seq dataset are available and
  per-cell cell-type labels must be transferred without joint reference-target integration.
when_to_use_negative:
- The reference is scRNA-seq rather than scATAC-seq — Cellcano is designed for scATAC-seq-as-reference;
  cross-modality prediction (scRNA-seq reference) and multimodal references are planned
  but not implemented
- Input is already a feature table of non-gene-score features (e.g. bin counts) chosen
  deliberately — although gene scores work as equally well or better than bin counts,
  the workflow as described assumes ArchR-derived gene scores
- Only unsupervised cluster discovery is needed and no labeled reference exists —
  supervised Cellcano requires reference cell-type labels
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cellcano
  role: 'Core tool: runs the two-round supervised learning (MLP first round, KD second
    round) to predict cell types from scATAC-seq gene scores'
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: 'Upstream preprocessing: processes raw scATAC-seq data (fragment/bam files)
    into gene scores for both reference and target datasets'
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:9251bdfe2fe3628fea8f5de194d45ecafc029e383f802823d547b0101a774b62
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scatac-seq-cell-type-prediction

## Summary

Predict single-cell ATAC-seq cell types in a target dataset using Cellcano's two-round supervised learning: an MLP trained on reference gene scores, followed by a self-Knowledge Distiller (KD) model trained on low-entropy anchor cells from the target. Using a scATAC-seq reference outperforms unsupervised clustering and cross-modality label transfer in accuracy, robustness and scalability.

## When to use

Use when both a labeled scATAC-seq reference dataset (gene score matrix + cell-type metadata) and an unlabeled target scATAC-seq dataset are available and per-cell cell-type labels must be transferred without joint reference-target integration.

## When NOT to use

- The reference is scRNA-seq rather than scATAC-seq — Cellcano is designed for scATAC-seq-as-reference; cross-modality prediction (scRNA-seq reference) and multimodal references are planned but not implemented
- Input is already a feature table of non-gene-score features (e.g. bin counts) chosen deliberately — although gene scores work as equally well or better than bin counts, the workflow as described assumes ArchR-derived gene scores
- Only unsupervised cluster discovery is needed and no labeled reference exists — supervised Cellcano requires reference cell-type labels

## Inputs

- Reference gene score matrix (processed from scATAC-seq fragment/bam files via ArchR)
- Reference cell-type metadata (per-cell labels)
- Target gene score matrix (cells x genes)
- Entropy quantile cutoff parameter (default 0.4)

## Outputs

- predict_celltypes.csv with one row per target cell and columns pred_celltype, firstround_pred_celltype, entropy
- Per-cell prediction probabilities from the first-round MLP prediction

## How to apply

First process raw scATAC-seq data (fragment or bam files) into gene scores for both reference and target with the ArchR pipeline, and apply an F-test on reference gene scores to select cell-type-specific genes as features. Train the first-round multi-layer perceptron (MLP) on the reference gene score matrix with cell-type labels, predict cell types for target cells, and compute per-cell prediction entropy from the first-round prediction probabilities. Select cells with lower entropies as anchors — the anchor fraction is controlled by an entropy quantile cutoff (0.4 by default, i.e. 40% of target cells; e.g. 0.3 selects 30%) — and with >1,000 cells use default two-round prediction. Train the self-Knowledge Distiller (KD) model on the anchors using predicted pseudo labels, then apply it to predict cell types for remaining non-anchor cells; assemble the final table with columns pred_celltype, firstround_pred_celltype, and entropy. The two-round design alleviates the distributional shift between reference and target data, which is the rationale for improved prediction performance.

## Related tools

- **Cellcano** (Core tool: runs the two-round supervised learning (MLP first round, KD second round) to predict cell types from scATAC-seq gene scores) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Upstream preprocessing: processes raw scATAC-seq data (fragment/bam files) into gene scores for both reference and target datasets)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- predict_celltypes.csv exists and has exactly the three required columns (pred_celltype, firstround_pred_celltype, entropy)
- predict_celltypes.csv contains one labeled prediction for every target cell in the target gene score matrix
- With default two-round prediction, firstround_pred_celltype column reflects first-round MLP predictions for low-entropy anchor cells
- Number of anchor cells equals the entropy quantile cutoff fraction of target cells (e.g. 40% at the 0.4 default; 30% at 0.3)
- Cellcano was benchmarked on 50 celltyping tasks from human PBMCs and mouse brain; outperforming Seurat/Signac/scJoint label transfer on such a task indicates correct application

## Limitations

- Adaptation to cross-modality predictions (using scRNA-seq as reference) and to celltyping in single-cell DNA methylation is planned but not implemented
- Using multimodal reference data (jointly scRNA- and scATAC-seq) is an open future direction, not yet implemented
- Cellcano requires tensorflow (2.7.1), anndata (0.7.4), scanpy (1.8.2), numpy (1.19.2), h5py (2.10.0), keras and rpy2, plus R and ArchR if input is raw fragment/bam data; GPU is optional
- Prediction of cell numbers at or below 1,000 may not use default two-round prediction (firstround_pred_celltype appears when default two-round prediction is used or cell number is larger than 1,000)

## Evidence

- [abstract] two-round supervised learning algorithm and KD model design: "Here we develop Cellcano, a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data."
- [intro] first-round MLP and second-round KD on anchors: "In the first round, Cellcano trains an MLP model with reference gene scores and predicts cell types in target data."
- [intro] entropy-based anchor selection: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [intro] gene score preprocessing via ArchR: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [readme] installation via pip: "The most convinient way is to install with `pip`.

shell
pip install Cellcano
"
- [discussion] gene scores vs bin counts input choice: "We show that using gene scores works as equally well or even better than using bin counts as input."
