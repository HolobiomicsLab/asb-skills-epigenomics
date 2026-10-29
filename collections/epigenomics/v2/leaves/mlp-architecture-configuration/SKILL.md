---
name: mlp-architecture-configuration
description: 'Use when building Cellcano''s supervised celltyping models: a first-round
  MLP on reference gene scores, or a second-round self-KD teacher-student model trained
  on low-entropy anchor cells to classify remaining non-anchor cells.'
when_to_use_negative:
- When the goal is cross-modality label transfer using an scRNA-seq reference — Cellcano
  requires an scATAC-seq reference and this extension is planned but not implemented
- When the target data are already cell-type labeled (no prediction task), since the
  KD model is trained on pseudo labels from first-round predictions
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cellcano
  role: Implements the two-round supervised learning algorithm, including the MLP
    and self-Knowledge Distiller (KD) model trained on anchor cells
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Processes raw scATAC-seq data (fragment/bam files) into gene scores used as
    MLP/KD model input features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:8af00bc785630f5c803a9f2b2b35a257709bd0419975faf70599379d07e3b4ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mlp-architecture-configuration

## Summary

Configure and train the supervised MLP architecture used by Cellcano for scATAC-seq cell typing, including the second-round self-Knowledge Distiller (KD) model trained on anchor cells with pseudo labels to predict non-anchor target cells.

## When to use

Use when building Cellcano's supervised celltyping models: a first-round MLP on reference gene scores, or a second-round self-KD teacher-student model trained on low-entropy anchor cells to classify remaining non-anchor cells.

## When NOT to use

- When the goal is cross-modality label transfer using an scRNA-seq reference — Cellcano requires an scATAC-seq reference and this extension is planned but not implemented
- When the target data are already cell-type labeled (no prediction task), since the KD model is trained on pseudo labels from first-round predictions

## Inputs

- Gene score feature matrix from the target scATAC-seq dataset (rows = cells), restricted to anchor cells for second-round training
- First-round predicted labels (pseudo/noisy labels) for the anchor cells
- Reference gene score matrix with cell-type labels for the first-round MLP (ArchR-derived)

## Outputs

- Serialized KD model file (e.g. 'kd_model.h5') loadable for inference
- Predicted cell-type labels for all non-anchor target cells, written to 'nonanchor_predictions.csv'

## How to apply

In the first round, train an MLP on reference gene scores (processed via the ArchR pipeline) and predict cell types in the target data. Compute the entropy of each target cell's first-round prediction probabilities and select cells with lower entropies as anchors, using their predicted labels as pseudo labels. For the second round, build a teacher-student self-KD architecture: a teacher MLP with two hidden layers of 64 and 16 units operated at distillation temperature T2=3, and a student of the same size operated at temperature T1=1. Train the KD model on the anchor cells for 30 epochs with distillation loss weight alpha=0.1, combining hard pseudo-label cross-entropy with the soft-teacher distillation loss, then serialize the model (e.g. 'kd_model.h5') and apply it to gene score features of all remaining non-anchor cells.

## Related tools

- **Cellcano** (Implements the two-round supervised learning algorithm, including the MLP and self-Knowledge Distiller (KD) model trained on anchor cells) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Processes raw scATAC-seq data (fragment/bam files) into gene scores used as MLP/KD model input features)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- 'nonanchor_predictions.csv' exists in the output directory
- 'nonanchor_predictions.csv' parses as a table with exactly one predicted label per non-anchor target cell
- The trained model file (e.g. 'kd_model.h5') is loadable and usable for inference
- Training completes the specified 30 epochs without error when run on the anchor-cell training set
- Second-round predictions on anchors (held in pseudo-label training) are consistent with their first-round low-entropy assignments

## Limitations

- GPU (cuda toolkit and nvidia cudnn) is optional but not required; Cellcano has not been tested on Apple M1 hardware
- Cross-modality prediction (scRNA-seq as reference) and multimodal (joint scRNA-/scATAC-seq) references are future directions, not implemented

## Evidence

- [intro] Second round KD on anchors: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells."
- [abstract] Two-round algorithm framing: "Here we develop Cellcano, a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data."
- [intro] Anchor selection by entropy: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [readme] Software purpose and stack: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data"
- [readme] Dependency stack incl. tensorflow: [evidence span withheld — anchor status: external_doc]
