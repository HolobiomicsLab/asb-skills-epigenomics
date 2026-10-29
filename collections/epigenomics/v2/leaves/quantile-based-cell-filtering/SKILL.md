---
name: quantile-based-cell-filtering
description: Use when a supervised celltyping pipeline has produced first-round per-cell
  prediction probabilities for scATAC-seq target cells and a confident anchor subset
  is needed to train a second-round model.
when_to_use_negative:
- When a scRNA-seq reference must be used for cross-modality label transfer — Cellcano's
  two-round entropy-anchored scheme is designed for scATAC-seq reference data; cross-modality
  adaptation is planned but not implemented.
- When the goal is joint dimensionality reduction or integration of reference and
  target datasets (as in Seurat, Signac, scJoint) — Cellcano explicitly avoids jointly
  operating on both datasets.
- When prediction probabilities from a first round are unavailable; the filter requires
  per-cell class-probability vectors to compute entropy.
edam_operation: http://edamontology.org/operation_3695
tools:
- name: Cellcano
  role: 'Implements entropy-based anchor selection: computes per-cell prediction entropies
    from first-round MLP predictions and selects the lowest-entropy quantile (default
    0.4) as anchors for the second-round KD model'
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Preprocesses raw scATAC-seq data (fragment/bam files) into gene scores for
    both reference and target datasets, producing the feature matrix whose predictions
    feed the entropy filter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:a19fb7a3e204fb86b5281d6dab61210b49b05b9cbdb13f9ae7d43cd39d821023
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Quantile-based cell filtering

## Summary

Selects anchor cells by ranking target cells on per-cell prediction entropy from a first-round classifier and keeping only those below an entropy quantile cutoff (Cellcano default 0.4), so that second-round training uses the most confidently labeled cells.

## When to use

Use when a supervised celltyping pipeline has produced first-round per-cell prediction probabilities for scATAC-seq target cells and a confident anchor subset is needed to train a second-round model.

## When NOT to use

- When a scRNA-seq reference must be used for cross-modality label transfer — Cellcano's two-round entropy-anchored scheme is designed for scATAC-seq reference data; cross-modality adaptation is planned but not implemented.
- When the goal is joint dimensionality reduction or integration of reference and target datasets (as in Seurat, Signac, scJoint) — Cellcano explicitly avoids jointly operating on both datasets.
- When prediction probabilities from a first round are unavailable; the filter requires per-cell class-probability vectors to compute entropy.

## Inputs

- First-round prediction output with per-cell prediction probability matrix (cells x cell types) from Cellcano's MLP on target gene scores
- Target scATAC-seq gene-score matrix (from ArchR preprocessing)
- Entropy quantile cutoff parameter (default 0.4)

## Outputs

- anchor_cells.csv containing one row per selected anchor cell
- Pseudo-labeled anchor set (lowest-entropy quantile of target cells) for KD-model training

## How to apply

Load the first-round prediction output containing per-cell prediction probabilities (Cellcano's first-round MLP predictions on target gene scores). Compute Shannon entropy for each cell across its class-probability vector, then rank all target cells by entropy. Apply an entropy quantile cutoff to keep the lowest-entropy cells as anchors — Cellcano's software default is 0.4, i.e. the lowest 40% of cells, and the cutoff parameter (e.g. 0.3 for 30%) controls the anchor count directly. Write the selected anchor cell identifiers to anchor_cells.csv and pass these pseudo-labeled anchors to the second-round self-Knowledge Distiller (KD) model. The rationale is that low prediction entropy indicates confident first-round calls, mitigating distributional shift between the scATAC-seq reference and target datasets.

## Related tools

- **Cellcano** (Implements entropy-based anchor selection: computes per-cell prediction entropies from first-round MLP predictions and selects the lowest-entropy quantile (default 0.4) as anchors for the second-round KD model) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Preprocesses raw scATAC-seq data (fragment/bam files) into gene scores for both reference and target datasets, producing the feature matrix whose predictions feed the entropy filter)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- anchor_cells.csv exists and contains one row per selected anchor cell
- Number of anchor rows equals the cutoff fraction of the input target cells (e.g. 40% with the default 0.4 cutoff)
- Every retained anchor's prediction entropy is below the entropy quantile threshold computed over all target cells
- Set of anchor identifiers is a subset of the target dataset's cell identifiers
- Second-round KD model trained only on these anchors improves or maintains accuracy on non-anchor cells; Cellcano is reported robust against the anchor selection procedure

## Limitations

- Anchor count depends on a single quantile cutoff; too small a cutoff may leave too few anchors per cell type for reliable KD-model training.
- Results may be sensitive to the anchor selection procedure in principle, although the paper reports Cellcano is robust against it.
- Cellcano has not been tested on M1 hardware and cross-modality (scRNA-seq reference) adaptation is not yet implemented.

## Evidence

- [intro] Entropy filter mechanism: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [discussion] Robustness of the filter: "Cellcano is also robust against the anchor selection procedure and batch effects in the data."
- [readme] Software availability: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data"
