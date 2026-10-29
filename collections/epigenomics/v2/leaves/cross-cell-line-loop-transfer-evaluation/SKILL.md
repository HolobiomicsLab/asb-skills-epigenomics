---
name: cross-cell-line-loop-transfer-evaluation
description: Use when a Hi-C loop-calling model trained on one cell line must be applied
  to Hi-C data from a different cell line, and transferability must be quantified
  via precision-recall curve comparison against the source-cell-line baseline.
when_to_use_negative:
- Input is single-cell Hi-C (scHi-C) data — the paper states GILoop 'only focuses
  on bulk Hi-C' and is not capable of handling extremely sparse scHi-C data
- The target cell line has a sequencing depth that differs from any pre-trained model
  provided in the GILoop repository — the discussion flags this as weak usability
- The goal is de novo loop calling on the training cell line rather than cross-cell-line
  transfer evaluation — use standard GILoop training/inference instead
edam_operation: http://edamontology.org/operation_0306
tools:
- name: GILoop
  role: Dual-branch neural network whose pre-trained GM12878 checkpoints are run in
    inference mode on target-cell-line Hi-C data to detect CTCF-mediated loops for
    the transfer evaluation
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Alternative Hi-C loop caller mentioned in the article context; not used in
    the transfer evaluation itself
  repo: PouletAxel/SIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:c64c422f5136e2559ca3477ba45f5996c6729558488dbadf143d2917070dea4a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-cell-line-loop-transfer-evaluation

## Summary

Evaluates whether a GILoop model trained on one cell line's Hi-C data (e.g., GM12878) can predict CTCF-mediated loops in another cell line (e.g., K562, HeLa-S3) with minimal performance decay, by running inference with pre-trained checkpoints and comparing precision-recall curves across source and target cell lines.

## When to use

Use when a Hi-C loop-calling model trained on one cell line must be applied to Hi-C data from a different cell line, and transferability must be quantified via precision-recall curve comparison against the source-cell-line baseline.

## When NOT to use

- Input is single-cell Hi-C (scHi-C) data — the paper states GILoop 'only focuses on bulk Hi-C' and is not capable of handling extremely sparse scHi-C data
- The target cell line has a sequencing depth that differs from any pre-trained model provided in the GILoop repository — the discussion flags this as weak usability
- The goal is de novo loop calling on the training cell line rather than cross-cell-line transfer evaluation — use standard GILoop training/inference instead

## Inputs

- Target-cell-line Hi-C dataset (e.g., K562 Hi-C, GEO GSE63525; HeLa-S3 Hi-C, 4DN 4DNESCMX7L58)
- Intra-chromosomal interaction matrices and graph structures preprocessed for GILoop's dual-branch input format
- Pre-trained GILoop checkpoints from the source cell line (GM12878), e.g., 20%-downsampled and 50%-downsampled models
- Source-cell-line (GM12878) baseline PR curves for comparison

## Outputs

- Per-transfer-setting precision-recall curves for target cell lines (K562, HeLa-S3)
- Overlaid source-vs-target PR curve figure replicating Figure 6 layout (e.g., figure6_transfer_pr_curves.png)
- Qualitative assessment of performance decay between source and transferred settings

## How to apply

Download the target cell line's Hi-C dataset (e.g., K562 from GEO accession GSE63525; HeLa-S3 from 4DN accession 4DNESCMX7L58) and preprocess each contact map into intra-chromosomal interaction matrices and graph structures matching GILoop's dual-branch (graph + image) input format. Select the pre-trained GM12878 checkpoint whose sequencing depth matches the target: per the reported case study, the 20%-downsampled model is applied to K562 and the 50% model to HeLa-S3. Run GILoop inference on each target cell line using the corresponding checkpoint, then compute precision-recall curves for each transfer setting and overlay them with the source-cell-line (GM12878) baseline curves in a single figure replicating the paper's Figure 6 layout. Acceptance criterion: the target-cell-line PR curves lie near-identically to the corresponding source-cell-line curves, indicating minimal performance decay consistent with the paper's transfer-learning case studies.

## Related tools

- **GILoop** (Dual-branch neural network whose pre-trained GM12878 checkpoints are run in inference mode on target-cell-line Hi-C data to detect CTCF-mediated loops for the transfer evaluation) — github.com/fzbio/GILoop
- **SIP** (Alternative Hi-C loop caller mentioned in the article context; not used in the transfer evaluation itself) — PouletAxel/SIP

## Examples

```
python demo.py
```

## Evaluation signals

- Target-cell-line (K562, HeLa-S3) PR curves lie near-identically to the corresponding GM12878 source-cell-line PR curves (minimal performance decay), consistent with Figure 6
- Each target cell line is inferred with the checkpoint whose downsampling fraction matches its sequencing depth (20% model for K562, 50% model for HeLa-S3)
- Predicted loops correspond to CTCF-mediated organizational/functional looping patterns as characterized in the paper
- Overlaid figure reproduces the layout of the paper's Figure 6 and is saved as figure6_transfer_pr_curves.png
- Input Hi-C matrices are properly preprocessed into intra-chromosomal interaction matrices and graph structures compatible with GILoop's dual-branch format

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)

## Evidence

- [abstract] Transfer-learning case studies across cell lines: "along with multiple transfer-learning case studies, we demonstrate that GILoop can accurately model the organizational and functional patterns of CTCF-mediated looping across different cell lines"
- [intro] Transferability with minimal performance decay: "the GILoop framework proposed in this work is transferable across cell lines. A trained GILoop model is able to predict CTCF-mediated loops in another cell line with minimal performance decay"
- [discussion] Sequencing-depth mismatch limitation: "it does lead to weak usability, if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository."
- [readme] GILoop pipeline scope from README: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
