---
name: temperature-scaled-soft-label-distillation
description: Use when you have first-round MLP predictions on a target scATAC-seq
  gene score matrix and need to classify non-anchor cells by self-distillation from
  confident anchors.
when_to_use_negative:
- The target cells have not been through the first-round MLP reference-based prediction
  — no prediction probabilities exist to compute anchor-selection entropies
- Input is raw scATAC-seq data (fragment/bam files) rather than processed gene score
  features — the ArchR preprocessing step must run first
- A joint reference/target integration method (Seurat, Signac, scJoint) is required
  because Cellcano does not jointly operate on reference and target datasets
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: Cellcano
  role: Implements the two-round supervised algorithm; its second round trains the
    self-Knowledge Distiller (teacher at T2=3, student at T1=1, alpha=0.1, 30 epochs)
    on anchors and predicts non-anchor cells
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Processes raw scATAC-seq fragment/bam data into the gene score feature matrices
    that serve as KD model input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:2e445d99bb087f0ecd9b97b14048fd9b957c482299f700b25a90970c7097e02a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# temperature-scaled-soft-label-distillation

## Summary

Train a self-Knowledge Distiller (KD) model on high-confidence anchor cells with pseudo labels to classify remaining non-anchor target cells in Cellcano's second-round scATAC-seq celltyping, using temperature-scaled soft teacher probabilities plus hard pseudo-label cross-entropy.

## When to use

Use when you have first-round MLP predictions on a target scATAC-seq gene score matrix and need to classify non-anchor cells by self-distillation from confident anchors.

## When NOT to use

- The target cells have not been through the first-round MLP reference-based prediction — no prediction probabilities exist to compute anchor-selection entropies
- Input is raw scATAC-seq data (fragment/bam files) rather than processed gene score features — the ArchR preprocessing step must run first
- A joint reference/target integration method (Seurat, Signac, scJoint) is required because Cellcano does not jointly operate on reference and target datasets

## Inputs

- Target scATAC-seq gene score feature matrix (rows = cells) from the ArchR pipeline
- First-round MLP prediction probability matrix for target cells (used to compute per-cell entropies and pseudo labels)
- Anchor cell indices/labels (lowest-entropy target cells with first-round pseudo labels)

## Outputs

- Trained self-KD model serialized as 'kd_model.h5' (loadable for inference)
- 'nonanchor_predictions.csv' with one predicted cell-type label per non-anchor target cell

## How to apply

After the first round, compute the entropy of each target cell's prediction-probability vector and select cells with lower entropies as anchors, using their predicted labels as pseudo labels. Restrict the ArchR-derived gene score feature matrix to these anchors and build the KD teacher-student pair in Cellcano: a teacher MLP with two hidden layers of 64 and 16 units operated at distillation temperature T2=3, and a student of the same size at temperature T1=1. Train for 30 epochs with distillation loss weight alpha=0.1, combining hard pseudo-label cross-entropy with the soft-teacher distillation loss. Serialize the student (e.g. 'kd_model.h5'), then apply it to the gene score features of all remaining non-anchor cells and write one predicted cell-type label per cell to 'nonanchor_predictions.csv'. The rationale is that self-distillation on confident target anchors alleviates the distributional shift between reference and target data and improves prediction performance.

## Related tools

- **Cellcano** (Implements the two-round supervised algorithm; its second round trains the self-Knowledge Distiller (teacher at T2=3, student at T1=1, alpha=0.1, 30 epochs) on anchors and predicts non-anchor cells) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Processes raw scATAC-seq fragment/bam data into the gene score feature matrices that serve as KD model input)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- 'nonanchor_predictions.csv' exists and parses as a table with exactly one predicted label per non-anchor target cell
- The serialized model file (e.g. 'kd_model.h5') is loadable and usable for inference
- Anchor set is reproducible: cells selected as anchors correspond to the lowest entropies of the first-round prediction probabilities
- Training completes the specified schedule (30 epochs) with the distillation loss (alpha=0.1, T2=3 teacher / T1=1 student) converging without NaNs
- Predicted labels on non-anchor cells are drawn from the reference cell-type label vocabulary

## Limitations

- Adaptation to cross-modality prediction (e.g. using scRNA-seq as reference) and other scenarios such as single-cell DNA methylation celltyping is planned but not implemented
- Using multimodal reference data (joint scRNA- and scATAC-seq) to improve celltyping is an open future direction, not supported
- Cellcano has not been tested on M1 hardware per the README; GPU support requires CUDA toolkit and nvidia cudnn and is optional

## Evidence

- [intro] Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells.: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells."
- [intro] we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [abstract] The method alleviates the distributional shift between reference and target data and improves the prediction performance.: "The method alleviates the distributional shift between reference and target data and improves the prediction performance."
- [intro] Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [readme] The most convinient way is to install with `pip`.: [evidence span withheld — anchor status: external_doc]
