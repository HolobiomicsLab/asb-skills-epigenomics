---
name: knowledge-distillation-model-training
description: Use when a first-round MLP has already predicted cell types in a target
  scATAC-seq gene score matrix and you need to retrain on low-entropy anchor cells
  to classify the remaining non-anchor cells.
when_to_use_negative:
- 'You have not yet run the first round: anchor selection depends on first-round prediction
  entropies, so no pseudo labels or anchors exist to train the KD model.'
- The reference and target are scRNA-seq/scATAC-seq cross-modality label transfer
  — Cellcano is designed with scATAC-seq as reference; cross-modality adaptation is
  only a planned future direction.
- Input is raw scATAC-seq (fragment or bam file) that has not yet been converted to
  gene scores via the ArchR pipeline.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Cellcano
  role: Implements the two-round supervised learning algorithm, including the self-Knowledge
    Distiller trained on anchor cells with pseudo labels to predict non-anchor cells
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Processes raw scATAC-seq data (fragment or bam files) into gene score matrices
    for both reference and target datasets, which serve as KD model features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:5ec60668948cdf15113982ccbced573c24fa0278162388ff0de3547b05ae69df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# knowledge-distillation-model-training

## Summary

Train Cellcano's second-round self-Knowledge Distiller (KD) model on target-data anchor cells using first-round pseudo labels, then apply it to classify the remaining non-anchor cells. This adapts the model to the target distribution and alleviates distributional shift between reference and target scATAC-seq data.

## When to use

Use when a first-round MLP has already predicted cell types in a target scATAC-seq gene score matrix and you need to retrain on low-entropy anchor cells to classify the remaining non-anchor cells.

## When NOT to use

- You have not yet run the first round: anchor selection depends on first-round prediction entropies, so no pseudo labels or anchors exist to train the KD model.
- The reference and target are scRNA-seq/scATAC-seq cross-modality label transfer — Cellcano is designed with scATAC-seq as reference; cross-modality adaptation is only a planned future direction.
- Input is raw scATAC-seq (fragment or bam file) that has not yet been converted to gene scores via the ArchR pipeline.

## Inputs

- Gene score feature matrix for the target scATAC-seq dataset (rows = cells, restricted to anchor cells for training)
- First-round MLP prediction probabilities per target cell (used to compute entropies for anchor selection)
- First-round predicted cell-type labels for anchor cells (used as pseudo labels)
- Gene score features of all remaining non-anchor target cells (for inference)

## Outputs

- Trained self-Knowledge Distiller model serialized to a model file (e.g. 'kd_model.h5')
- Predicted cell-type labels for non-anchor cells written to 'nonanchor_predictions.csv' (one predicted label per non-anchor cell)

## How to apply

After the first-round MLP predicts cell types in the target gene score matrix (generated via the ArchR pipeline), compute the entropy of each cell's prediction probabilities and select cells with lower entropies as anchors. Restrict the gene score feature matrix rows to these anchor cells and use their first-round predicted labels as pseudo/noisy labels. Build the KD teacher-student architecture: a teacher MLP with two hidden layers of 64 and 16 units at distillation temperature T2=3, and a student of the same size at temperature T1=1; train for 30 epochs with distillation loss weight alpha=0.1, combining hard pseudo-label cross-entropy with the soft-teacher distillation loss. Serialize the trained KD model (e.g. 'kd_model.h5'), then apply it to the gene score features of all remaining non-anchor target cells and write their predicted labels to 'nonanchor_predictions.csv'.

## Related tools

- **Cellcano** (Implements the two-round supervised learning algorithm, including the self-Knowledge Distiller trained on anchor cells with pseudo labels to predict non-anchor cells) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Processes raw scATAC-seq data (fragment or bam files) into gene score matrices for both reference and target datasets, which serve as KD model features)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- 'nonanchor_predictions.csv' exists and parses as a table with exactly one predicted label per non-anchor cell
- The serialized KD model file (e.g. 'kd_model.h5') is loadable and usable for inference
- Anchor set is confirmed to consist of cells with the lowest first-round prediction entropies, carrying pseudo labels
- Teacher/student configuration matches the paper: two hidden layers of 64 and 16 units, T2=3 (teacher) and T1=1 (student), 30 epochs, alpha=0.1
- script_runs / file_exists checks on the Cellcano second-round run complete without error

## Limitations

- Predictions inherit noise from first-round pseudo labels, which is why anchor selection by low prediction entropy is used; Cellcano is described as robust to this procedure but label noise remains a caveat.
- GPU is optional; the package requires only enough RAM for in-memory operations and has not been tested on Apple M1.
- Cellcano is not yet adapted to cross-modality references (e.g. scRNA-seq reference for scATAC-seq celltyping) or single-cell DNA methylation celltyping — these are planned future directions.

## Evidence

- [intro] Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells.: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells."
- [intro] we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [abstract] The method alleviates the distributional shift between reference and target data and improves the prediction performance.: "The method alleviates the distributional shift between reference and target data and improves the prediction performance."
- [discussion] Cellcano is also robust against the anchor selection procedure and batch effects in the data.: "Cellcano is also robust against the anchor selection procedure and batch effects in the data."
- [readme] If the input is scATAC-seq raw data (i.e. fragment file or bam file), ArchR package has to be installed.: [evidence span withheld — anchor status: external_doc]
