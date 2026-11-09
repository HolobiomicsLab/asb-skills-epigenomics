---
name: prediction-entropy-computation
description: Use when running Cellcano's two-round celltyping on scATAC-seq data and
  you need to derive anchor cells from first-round MLP prediction probabilities via
  an entropy quantile cutoff before training the KD model.
when_to_use_negative:
- 'Reference data are scRNA-seq rather than scATAC-seq: Cellcano is designed for scATAC-seq-as-reference
  celltyping, and cross-modality prediction using scRNA-seq as reference is planned
  but not implemented.'
- The analysis goal requires jointly operating on reference and target datasets, as
  Seurat, Signac, and scJoint do — Cellcano's entropy-based anchor scheme works without
  joint embedding.
- You need multi-modal (jointly scRNA- and scATAC-seq) references — this is an open
  future direction not yet supported.
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: Cellcano
  role: Computes per-cell prediction entropies from first-round MLP predictions and
    selects lowest-entropy target cells as anchors using the quantile cutoff; supports
    the second-round KD training
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Upstream pipeline that processes raw scATAC-seq data (fragment/bam files)
    into the gene scores used as first-round prediction input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:9964bf7fd1d8912bed024285cc58ec5f942ffa763100aa7b648e701843930891
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# prediction-entropy-computation

## Summary

Computes per-cell prediction entropy from first-round Cellcano MLP prediction probabilities and selects the lowest-entropy target cells as anchors for the second-round Knowledge Distiller model, controlled by an entropy quantile cutoff (software default 0.4).

## When to use

Use when running Cellcano's two-round celltyping on scATAC-seq data and you need to derive anchor cells from first-round MLP prediction probabilities via an entropy quantile cutoff before training the KD model.

## When NOT to use

- Reference data are scRNA-seq rather than scATAC-seq: Cellcano is designed for scATAC-seq-as-reference celltyping, and cross-modality prediction using scRNA-seq as reference is planned but not implemented.
- The analysis goal requires jointly operating on reference and target datasets, as Seurat, Signac, and scJoint do — Cellcano's entropy-based anchor scheme works without joint embedding.
- You need multi-modal (jointly scRNA- and scATAC-seq) references — this is an open future direction not yet supported.

## Inputs

- First-round prediction output with per-cell prediction probabilities (MLP trained on reference gene scores)
- Entropy quantile cutoff parameter (default 0.4)

## Outputs

- anchor_cells.csv containing one row per selected anchor cell (40% of target cells at the default cutoff)
- Set of anchor cells with pseudo labels for training the self-Knowledge Distiller (KD) model

## How to apply

Load the first-round prediction output containing per-cell prediction probabilities from the MLP trained on reference gene scores. Calculate the entropy for each target cell based on its prediction probabilities, then rank cells by entropy and select those below an entropy quantile cutoff as anchors — Cellcano's software default cutoff is 0.4, i.e. the 40% of target cells with the lowest entropies. Write the selected anchor cell identifiers to anchor_cells.csv; these anchors carry their first-round pseudo labels into the self-Knowledge Distiller (KD) model, which then predicts the remaining non-anchor cells. The quantile cutoff is the parameter that controls anchor count, and the paper reports Cellcano is robust against the anchor selection procedure, so moderate variation in the cutoff does not materially affect performance.

## Related tools

- **Cellcano** (Computes per-cell prediction entropies from first-round MLP predictions and selects lowest-entropy target cells as anchors using the quantile cutoff; supports the second-round KD training) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Upstream pipeline that processes raw scATAC-seq data (fragment/bam files) into the gene scores used as first-round prediction input)

## Examples

```
pip install Cellcano
```

## Evaluation signals

- anchor_cells.csv exists and contains one row per selected anchor cell
- Number of anchor rows equals the expected fraction of input target cells at the chosen cutoff (40% with the default 0.4 cutoff)
- Every anchor's entropy is below the entropy quantile threshold computed from the first-round prediction probability distribution
- Selected anchors carry first-round pseudo labels suitable as KD training targets, and the KD model subsequently predicts all remaining non-anchor target cells

## Limitations

- The anchor count is entirely governed by the entropy quantile cutoff (default 0.4); no per-class anchor balancing is described, so rare cell types may be under-represented among anchors.
- Anchor quality depends on first-round MLP prediction probabilities; distributional shift between reference and target could inflate entropies and degrade anchor selection, although the paper reports the method alleviates such shift and is robust to the anchor selection procedure.
- Cellcano requires a standard-computer-capable environment with tensorflow 2.7.1, anndata 0.7.4, scanpy 1.8.2, numpy 1.19.2, h5py 2.10.0, keras, and rpy2; raw fragment/bam input additionally requires ArchR. GPU is optional and has not been tested on M1 hardware.

## Evidence

- [intro] Entropy-based anchor selection from first-round probabilities: "we first calculate entropy for each cell based on the prediction probabilities from the first-round prediction and then select cells with lower entropies as anchors"
- [intro] KD model trained on anchors with pseudo labels: "Cellcano trains a self-Knowledge Distiller model (KD model)29 on anchors using the predicted pseudo labels, and then apply the trained KD model to predict cell types for remaining non-anchor cells."
- [discussion] Robustness to the anchor selection procedure: "Cellcano is also robust against the anchor selection procedure and batch effects in the data."
- [readme] Software availability and documentation: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications]"
- [abstract] Two-round supervised learning algorithm: "Here we develop Cellcano, a computational method based on a two-round supervised learning algorithm to identify cell types from scATAC-seq data."
