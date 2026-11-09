---
name: entropy-based-anchor-cell-selection
description: Use when running two-round Cellcano prediction (target cell number >
  1,000 or default two-round mode) and you need confident target cells to serve as
  training anchors for the KD model.
when_to_use_negative:
- Target dataset has fewer than 1,000 cells or single-round prediction is requested
  — two-round prediction (and hence anchor selection) is not invoked by default
- You only have reference-side data and no target gene score matrix, since entropies
  are computed on target-cell first-round predictions
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Cellcano
  role: Implements the entropy computation, quantile-based anchor selection, and the
    subsequent KD-model training on anchors
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Upstream pipeline that produces the reference and target gene score matrices
    on which first-round predictions and entropies are computed
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:68b09b20991241b4a0dd4094e5971159f66e8a795215f0ab369c555adcf495d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# entropy-based-anchor-cell-selection

## Summary

Cellcano's intermediate step between its two supervised rounds: compute per-cell prediction entropy from first-round MLP probabilities on the scATAC-seq target gene score matrix and keep only the lowest-entropy cells as anchors for training the second-round Knowledge Distiller.

## When to use

Use when running two-round Cellcano prediction (target cell number > 1,000 or default two-round mode) and you need confident target cells to serve as training anchors for the KD model.

## When NOT to use

- Target dataset has fewer than 1,000 cells or single-round prediction is requested — two-round prediction (and hence anchor selection) is not invoked by default
- You only have reference-side data and no target gene score matrix, since entropies are computed on target-cell first-round predictions

## Inputs

- First-round Cellcano prediction output with per-cell prediction probabilities for target cells
- Target gene score matrix (from ArchR processing of scATAC-seq data)
- Entropy quantile cutoff parameter (default 0.4)

## Outputs

- Set of anchor cells with pseudo labels (firstround_pred_celltype) from the lowest-entropy quantile
- Per-cell entropy values carried into the final prediction table (columns: pred_celltype, firstround_pred_celltype, entropy)

## How to apply

After the first round, in which Cellcano trains a multi-layer perceptron (MLP) on reference gene scores and predicts cell types in the target gene score data, calculate entropy for each target cell from its first-round prediction probabilities. Rank cells by entropy and select the lowest-entropy fraction as anchors; the number of anchors is controlled by an entropy quantile cutoff, with 0.4 (i.e., 40% of target cells) as the software default. The selected anchors carry pseudo labels from the first round and are stored as the firstround_pred_celltype column, and Cellcano is reported to be robust against the anchor selection procedure, so the exact quantile is not highly sensitive. The anchors then train the self-Knowledge Distiller (KD) model, which predicts cell types for the remaining non-anchor cells.

## Related tools

- **Cellcano** (Implements the entropy computation, quantile-based anchor selection, and the subsequent KD-model training on anchors) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Upstream pipeline that produces the reference and target gene score matrices on which first-round predictions and entropies are computed)

## Examples

```
pip install Cellcano  # then run Cellcano first-round prediction; entropies and anchor selection are invoked automatically in default two-round mode
```

## Evaluation signals

- Number of selected anchors equals the entropy quantile fraction of the target cells (e.g., anchor_cells.csv rows = 40% of target cells at the default 0.4 cutoff)
- Every anchor has a non-empty pseudo label in the firstround_pred_celltype column
- Each anchor's entropy is at or below the quantile threshold and no anchor appears in the non-anchor set
- Final predict_celltypes.csv exists with exactly pred_celltype, firstround_pred_celltype, and entropy columns and one labeled row per target cell
- Anchor entropies are strictly lower than the mean entropy of the non-anchor (excluded) cells

## Limitations

- The anchor fraction is a heuristic quantile; while the paper reports robustness against the anchor selection procedure, pathological target distributions could still yield poor anchors
- Anchor quality depends entirely on first-round MLP prediction probabilities, so errors in the reference gene score matrix propagate into pseudo labels

## Evidence

- [intro] entropy-based anchor selection rationale: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [intro] anchor use in second round: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict"
- [discussion] robustness to anchor selection: "Cellcano is also robust against the anchor selection procedure and batch effects in the data."
- [other] default two-round trigger and output column: [evidence span withheld — anchor status: not_found]
- [other] default anchor cutoff: [evidence span withheld — anchor status: not_found]
