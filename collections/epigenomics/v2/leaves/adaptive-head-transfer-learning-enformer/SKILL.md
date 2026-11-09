---
name: adaptive-head-transfer-learning-enformer
description: Use when benchmarking a motif-based classifier (e.g., BOM) against a
  pretrained sequence model on binary or multiclass distal-CRE classification across
  cell types, and Enformer must be fine-tuned on identical train/test partitions.
when_to_use_negative:
- Enhancer activity depends on open chromatin, histone marks, or sequence context
  beyond the cropped CRE window, which BOM/Enformer-style cis-only models cannot capture
- The goal is direct motif-level interpretability of a regulatory vocabulary from
  motif counts — use BOM (XGBoost + SHAP) instead of fine-tuning a deep model
- Training data for the cell state is too limited, since sequence-based deep learning
  performance is linked to the limited amount of data available per cell state
edam_operation: http://edamontology.org/operation_0422
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3308
tools:
- name: Enformer
  role: Pretrained sequence transformer fine-tuned with an adaptive two-layer fully
    connected head on the CRE classification task
- name: Bag-of-Motifs (BOM)
  role: Motif-count + XGBoost classifier that the adapted Enformer serves as a deep-learning
    baseline against
  repo: github.com/ewonglab/BOM_package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:a737e553616f6667e99806456479ba55f0e2f743a5965d3bd503e9d3dfa9a987
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bag-of-Motifs (BOM)
  - DNABERT
  - Enformer
  - LS-GKM
  - gkmSVM
  edam_operation: http://edamontology.org/operation_0422
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# adaptive-head-transfer-learning-enformer

## Summary

Adapts the pretrained Enformer transformer to a new cell-type-specific CRE classification task by attaching an adaptive head of two fully connected linear layers, re-cropping Enformer's input to focus precisely on each CRE of interest, and exploring the hyperparameter space — used as a deep-learning baseline to benchmark BOM.

## When to use

Use when benchmarking a motif-based classifier (e.g., BOM) against a pretrained sequence model on binary or multiclass distal-CRE classification across cell types, and Enformer must be fine-tuned on identical train/test partitions.

## When NOT to use

- Enhancer activity depends on open chromatin, histone marks, or sequence context beyond the cropped CRE window, which BOM/Enformer-style cis-only models cannot capture
- The goal is direct motif-level interpretability of a regulatory vocabulary from motif counts — use BOM (XGBoost + SHAP) instead of fine-tuning a deep model
- Training data for the cell state is too limited, since sequence-based deep learning performance is linked to the limited amount of data available per cell state

## Inputs

- FASTA sequences generated from the mouse E8.25 input BED file (chromosome, start, stop, condition) with genome = Mmusculus (mm10)
- Binary cell-type-specific CRE labels and train/test partitions shared with the other baselines (LS-GKM, DNABERT, CNNs)
- Pretrained Enformer model weights

## Outputs

- Cell-type-specific CRE classification predictions for each of the 17 E8.25 cell types
- auPR and MCC metrics on the held-out test set, used to compute BOM's relative improvement over Enformer

## How to apply

Fine-tune the pretrained Enformer on the same sequences and splits used by all other baselines (sequences generated with generateAllFasta from a 4-column BED: chromosome, start, stop, condition, using the mm10 mouse genome). Replace Enformer's top with an adaptive layer consisting of two fully connected linear layers tailored to the new data and cell-type contexts. Adjust the input cropping to focus precisely on the CRE of interest to reduce noise and enhance precision. Perform extensive exploration of the hyperparameter space to improve accuracy. Evaluate on the held-out set with auPR and MCC, and compare against BOM's reported relative improvements (10.3% auPR, 33.4% MCC over Enformer across the 17 E8.25 mouse cell types).

## Related tools

- **Enformer** (Pretrained sequence transformer fine-tuned with an adaptive two-layer fully connected head on the CRE classification task)
- **Bag-of-Motifs (BOM)** (Motif-count + XGBoost classifier that the adapted Enformer serves as a deep-learning baseline against) — github.com/ewonglab/BOM_package

## Evaluation signals

- Verify the adaptive head consists of exactly two fully connected linear layers, not the original Enformer output head
- Verify input cropping is adjusted so Enformer's receptive field focuses precisely on each CRE of interest
- Recompute auPR and MCC on the held-out set and check that BOM's relative improvement over Enformer reproduces the reported 10.3% auPR and 33.4% MCC gains across the 17 E8.25 cell types
- Confirm Enformer was trained on identical sequences/splits as LS-GKM, DNABERT, and the CNN variants (same generateAllFasta FASTA)

## Limitations

- Sequence-based deep learning performance is likely linked to the limited amount of data available to train the large number of cell states
- The model is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used

## Evidence

- [discussion] We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts.: "We introduced an adaptive layer, consisting of two fully connected linear layers tailored to the new data and cell-type contexts."
- [discussion] We adjusted the cropping to focus precisely on the CRE of interest, thereby reducing noise to enhance the model’s precision.: "We adjusted the cropping to focus precisely on the CRE of interest, thereby reducing noise to enhance the model’s precision."
- [discussion] We also performed extensive exploration of the hyperparameter space to improve Enformer’s accuracy.: "We also performed extensive exploration of the hyperparameter space to improve Enformer’s accuracy."
