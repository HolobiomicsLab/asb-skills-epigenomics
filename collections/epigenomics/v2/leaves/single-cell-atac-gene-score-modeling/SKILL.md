---
name: single-cell-atac-gene-score-modeling
description: Use when you have a target scATAC-seq gene score feature matrix plus
  first-round Cellcano predictions (probabilities and labels) and must retrain a teacher-student
  KD model on low-entropy anchor cells to label all non-anchor cells.
when_to_use_negative:
- Reference data is scRNA-seq and you need cross-modality label transfer — the article
  notes this adaptation of Cellcano is planned but not implemented
- You want to jointly use multimodal scRNA- and scATAC-seq reference data — explicitly
  listed as an open future direction, not supported
- You need joint operation on reference and target datasets as in Seurat/Signac/scJoint
  integration workflows — Cellcano deliberately does not jointly operate on the two
  datasets
edam_operation: http://edamontology.org/operation_2990
tools:
- name: Cellcano
  role: Provides the two-round supervised learning framework; its second round trains
    the self-Knowledge Distiller (teacher T2=3, student T1=1) on anchors to predict
    non-anchor cells
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Processes raw scATAC-seq data (fragment or bam files) to produce the gene
    score matrices used as model input for both reference and target datasets
  repo: https://www.archrproject.com/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:f89cd65678e54b44ab9a9e404084566247798e9b8b461ddbf975251ccd191cb7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-atac-gene-score-modeling

## Summary

Reconstructs Cellcano's second-round self-Knowledge-Distiller (KD) classifier: anchor cells selected by first-round prediction entropy are retrained with pseudo labels to classify the remaining non-anchor cells of a target scATAC-seq gene score matrix.

## When to use

Use when you have a target scATAC-seq gene score feature matrix plus first-round Cellcano predictions (probabilities and labels) and must retrain a teacher-student KD model on low-entropy anchor cells to label all non-anchor cells.

## When NOT to use

- Reference data is scRNA-seq and you need cross-modality label transfer — the article notes this adaptation of Cellcano is planned but not implemented
- You want to jointly use multimodal scRNA- and scATAC-seq reference data — explicitly listed as an open future direction, not supported
- You need joint operation on reference and target datasets as in Seurat/Signac/scJoint integration workflows — Cellcano deliberately does not jointly operate on the two datasets

## Inputs

- Target scATAC-seq gene score feature matrix (rows = cells, columns = F-test-selected cell-type-specific genes, produced by the ArchR pipeline)
- First-round prediction probability matrix and predicted labels for target cells (from the reference-trained MLP)
- Anchor cell selection derived from first-round prediction entropies, used as pseudo labels

## Outputs

- Serialized trained KD model file (e.g. 'kd_model.h5') in the output directory
- 'nonanchor_predictions.csv' with one predicted cell-type label per non-anchor target cell

## How to apply

First, restrict the target gene score matrix rows to the anchor cells — those whose first-round prediction probability vectors have the lowest entropies — and use their first-round predicted labels as pseudo/noisy training labels. Then build Cellcano's KD teacher-student architecture: a teacher MLP with two hidden layers of 64 and 16 units operating at distillation temperature T2=3, and a same-sized student at temperature T1=1. Train the self-KD model on the anchor cells for 30 epochs with distillation loss weight alpha=0.1, combining hard pseudo-label cross-entropy with the soft-teacher distillation loss, and serialize the trained model (e.g. 'kd_model.h5'). Finally apply the trained KD model to the gene score features of all remaining non-anchor cells and write one predicted cell-type label per cell to 'nonanchor_predictions.csv'. The rationale is Cellcano's two-round design: the second round alleviates the distributional shift between reference and target by adapting the model to target-space features, which the authors benchmarked as accurate, robust, and computationally efficient across 50 celltyping tasks.

## Related tools

- **Cellcano** (Provides the two-round supervised learning framework; its second round trains the self-Knowledge Distiller (teacher T2=3, student T1=1) on anchors to predict non-anchor cells) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Processes raw scATAC-seq data (fragment or bam files) to produce the gene score matrices used as model input for both reference and target datasets) — https://www.archrproject.com/

## Examples

```
pip install Cellcano
```

## Evaluation signals

- 'nonanchor_predictions.csv' exists in the output directory and parses as a table with exactly one predicted label per non-anchor target cell
- The serialized KD model file (e.g. 'kd_model.h5') is loadable and usable for inference
- The training set rows correspond exactly to anchor cells with the lowest first-round prediction entropies, labeled with first-round predicted pseudo labels
- Reproduced hyperparameters match the paper's setup: teacher/student MLPs of 64 and 16 hidden units, T2=3, T1=1, alpha=0.1, 30 epochs
- file_exists / script_runs checks on the Cellcano second-round run pass end-to-end

## Limitations

- Cellcano's cross-modality use (e.g. scRNA-seq as reference for scATAC-seq celltyping) and single-cell DNA methylation celltyping are planned but not implemented
- Multimodal (joint scRNA- and scATAC-seq) references are an open future direction, not yet supported
- Installation caveat: Cellcano has not been tested on Apple M1 machines (Issue #6 documents a workaround)
- GPU support is optional; a standard computer with sufficient RAM is required for in-memory operations

## Evidence

- [intro] Second-round KD trained on anchors with pseudo labels: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells."
- [intro] Anchor selection by prediction entropy: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [abstract] Two-round algorithm on gene scores: "Here we develop Cellcano, a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data."
- [readme] Raw data input requires ArchR: [evidence span withheld — anchor status: not_found]
- [discussion] Robustness of second-round anchor selection: "Cellcano is also robust against the anchor selection procedure and batch effects in the data."
