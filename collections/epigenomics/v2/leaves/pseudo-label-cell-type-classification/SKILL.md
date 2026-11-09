---
name: pseudo-label-cell-type-classification
description: Use when you have target scATAC-seq gene scores plus first-round MLP
  predictions with per-cell probabilities, and need to classify non-anchor cells by
  self-training on confident (low-entropy) pseudo-labeled anchors.
when_to_use_negative:
- When the reference is scRNA-seq (cross-modality prediction) — the article notes
  this adaptation 'is planned but not implemented'
- When you only have raw fragment/bam files and no gene score matrix — ArchR preprocessing
  into gene scores must happen first
- When target cells already have trusted ground-truth labels, making pseudo-label
  self-distillation unnecessary
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cellcano
  role: 'Implements the second-round self-Knowledge Distiller (KD) model: entropy-based
    anchor selection, teacher-student distillation training on anchors with pseudo
    labels, and prediction of non-anchor cells'
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Upstream pipeline that processes raw scATAC-seq data (fragment/bam files)
    into the gene scores used as KD model input features
  repo: https://www.archrproject.com/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:5abe81484fcc95a3b088043305b414a0b17ba1cd2de848bfdd534a4540e59b50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pseudo-label-cell-type-classification

## Summary

Second round of Cellcano's two-round supervised algorithm: train a self-Knowledge Distiller (KD) model on low-entropy 'anchor' cells from the target scATAC-seq data using first-round predicted pseudo labels, then apply the trained KD model to classify remaining non-anchor target cells.

## When to use

Use when you have target scATAC-seq gene scores plus first-round MLP predictions with per-cell probabilities, and need to classify non-anchor cells by self-training on confident (low-entropy) pseudo-labeled anchors.

## When NOT to use

- When the reference is scRNA-seq (cross-modality prediction) — the article notes this adaptation 'is planned but not implemented'
- When you only have raw fragment/bam files and no gene score matrix — ArchR preprocessing into gene scores must happen first
- When target cells already have trusted ground-truth labels, making pseudo-label self-distillation unnecessary

## Inputs

- Gene score feature matrix for the target scATAC-seq dataset (rows = cells, columns = F-test-selected cell-type-specific genes)
- First-round MLP prediction probabilities for all target cells (used to compute per-cell entropies and pseudo labels)
- Anchor cell indices/labels selected by lowest first-round prediction entropy

## Outputs

- Serialized trained self-KD model file (e.g. 'kd_model.h5')
- Predicted cell-type labels for all non-anchor target cells ('nonanchor_predictions.csv', one label per cell)

## How to apply

From the first-round MLP predictions on target gene scores, compute the entropy of each cell's prediction probability vector and select cells with lower entropies as anchors, using their predicted labels as pseudo labels. Build the self-KD teacher-student architecture: a teacher MLP with two hidden layers of 64 and 16 units operated at distillation temperature T2=3, and a student of the same size at temperature T1=1. Train on the anchor cells' gene score feature matrix for 30 epochs with distillation loss weight alpha=0.1, combining hard pseudo-label cross-entropy with soft-teacher distillation loss. Serialize the trained model (e.g. 'kd_model.h5') and apply it to the gene score features of all remaining non-anchor cells to write predicted cell-type labels (e.g. 'nonanchor_predictions.csv'). The rationale is that anchoring on confidently predicted target cells alleviates the distributional shift between reference and target data and improves prediction performance.

## Related tools

- **Cellcano** (Implements the second-round self-Knowledge Distiller (KD) model: entropy-based anchor selection, teacher-student distillation training on anchors with pseudo labels, and prediction of non-anchor cells) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Upstream pipeline that processes raw scATAC-seq data (fragment/bam files) into the gene scores used as KD model input features) — https://www.archrproject.com/

## Examples

```
pip install Cellcano
```

## Evaluation signals

- 'nonanchor_predictions.csv' exists and parses as a table with exactly one predicted label per non-anchor target cell
- The serialized model file (e.g. 'kd_model.h5') is loadable and usable for inference
- Anchor set corresponds to cells with the lowest first-round prediction-probability entropies, carrying their predicted labels as pseudo labels
- Training hyperparameters match the Cellcano recipe: teacher/student hidden layers of 64 and 16 units, T2=3 (teacher), T1=1 (student), 30 epochs, distillation loss weight alpha=0.1
- The script completes a full file_exists / script_runs check on the Cellcano second-round run

## Limitations

- Cross-modality reference (e.g. scRNA-seq as reference for scATAC-seq celltyping) is discussed as a future direction but not implemented
- Multimodal reference data (jointly scRNA- and scATAC-seq) is not yet implemented
- Cellcano requires gene scores as input; raw data must first be processed through the ArchR pipeline (ArchR not tested on M1 macOS)

## Evidence

- [intro] Anchor selection by entropy: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [intro] Second-round KD training and non-anchor prediction: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types"
- [abstract] Two-round supervised framing: "a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data"
- [readme] Software availability and scope: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data"
