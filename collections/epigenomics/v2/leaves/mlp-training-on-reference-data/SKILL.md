---
name: mlp-training-on-reference-data
description: Use when cell-type labels must be transferred from an scATAC-seq reference
  gene score matrix (with cell-type metadata) to a target scATAC-seq gene score matrix,
  and first-round MLP predictions are needed to define anchor cells for two-round
  prediction.
when_to_use_negative:
- Cross-modality label transfer using scRNA-seq as the reference — the article notes
  this adaptation is planned but not implemented; Cellcano requires an scATAC-seq
  reference.
- The target already has trusted per-cell labels and only needs annotation formatting
  rather than prediction.
- Only raw fragment or bam files are available without running the ArchR gene score
  pipeline first — the MLP consumes gene scores, not raw scATAC-seq data.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Cellcano
  role: Implements the first-round MLP training on reference gene scores and the entropy-based
    anchor selection for two-round prediction
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Preprocesses raw scATAC-seq fragment/bam data to obtain gene scores for both
    reference and target datasets, the MLP's input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:dd524cc7b967377001e0f794513ca5b901946139df0d3af6fefb8b7c6bbfa579
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# mlp-training-on-reference-data

## Summary

Trains a multi-layer perceptron (MLP) on scATAC-seq reference gene scores with known cell-type labels as the first round of Cellcano's two-round supervised learning algorithm, producing per-cell predictions and entropies that anchor the second-round self-Knowledge Distiller.

## When to use

Use when cell-type labels must be transferred from an scATAC-seq reference gene score matrix (with cell-type metadata) to a target scATAC-seq gene score matrix, and first-round MLP predictions are needed to define anchor cells for two-round prediction.

## When NOT to use

- Cross-modality label transfer using scRNA-seq as the reference — the article notes this adaptation is planned but not implemented; Cellcano requires an scATAC-seq reference.
- The target already has trusted per-cell labels and only needs annotation formatting rather than prediction.
- Only raw fragment or bam files are available without running the ArchR gene score pipeline first — the MLP consumes gene scores, not raw scATAC-seq data.

## Inputs

- Reference gene score matrix (ArchR-derived) with per-cell cell-type metadata labels
- Target gene score matrix (ArchR-derived) with cells to be labeled
- F-test-selected cell-type-specific genes used as model features

## Outputs

- First-round predicted cell-type labels for every target cell (firstround_pred_celltype)
- Per-cell prediction probabilities and derived entropy values used for anchor selection
- Anchor cell set with pseudo labels feeding the second-round KD model
- predict_celltypes.csv with columns pred_celltype, firstround_pred_celltype, and entropy (one row per target cell)

## How to apply

Prepare the ArchR-derived gene score matrices for both reference and target data, and select cell-type-specific genes as features via an F-test on the reference gene scores. Train the MLP on the reference gene score matrix with the provided cell-type labels, then predict cell types for each target cell. From the first-round prediction probabilities, calculate an entropy value for each target cell and select cells with lower entropies as anchors; these anchors carry the firstround_pred_celltype label used as pseudo labels for the second-round self-Knowledge Distiller (KD) model. Two-round prediction is the default when the number of target cells is larger than 1,000. Rationale: using an scATAC-seq reference with a supervised MLP alleviates the distributional shift between reference and target and improves prediction performance relative to integration-based label transfer (Seurat, Signac, scJoint).

## Related tools

- **Cellcano** (Implements the first-round MLP training on reference gene scores and the entropy-based anchor selection for two-round prediction) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Preprocesses raw scATAC-seq fragment/bam data to obtain gene scores for both reference and target datasets, the MLP's input)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- predict_celltypes.csv exists and contains exactly the three required columns: pred_celltype, firstround_pred_celltype, and entropy.
- The output table has one labeled prediction row for every cell in the target gene score matrix.
- firstround_pred_celltype is populated when default two-round prediction is used or the cell number is larger than 1,000.
- Anchor cells selected as low-entropy cells are consistent with their first-round prediction probabilities (entropy computed from those probabilities).
- Predicted cell types on the human PBMC demo fall within the cell-type label vocabulary provided in the reference metadata.

## Limitations

- Two-round prediction (first-round anchors + KD second round) is triggered by default only when the cell number is larger than 1,000.
- Cellcano has not been tested on Apple M1 machines (noted in the README).
- Adaptations such as cross-modality prediction with scRNA-seq references or scDNA methylation celltyping are planned but not implemented.

## Evidence

- [intro] First round trains MLP on reference gene scores: "In the first round, Cellcano trains an MLP model with reference gene scores and predicts cell types in target data."
- [intro] Entropy-based anchor selection: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [abstract] Two-round supervised learning framework: "a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data"
- [intro] Gene score inputs via ArchR: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [readme] README scope statement: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications]"
