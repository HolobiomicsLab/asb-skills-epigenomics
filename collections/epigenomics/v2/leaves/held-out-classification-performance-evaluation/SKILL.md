---
name: held-out-classification-performance-evaluation
description: Use when a trained sequence-based classifier (e.g. BOM/XGBoost on motif
  counts) must be scored on a held-out test set of labeled CRE sequences, and per-class
  and aggregate performance metrics are needed for validation.
when_to_use_negative:
- When the goal is motif interpretation (SHAP) rather than classification accuracy
  — use save_shap_multi()/shapPlots_multi() instead
- When input data are already feature/model-output tables and no held-out sequence
  labels exist to score against
- When the question concerns motif grammar or TF co-binding spacing, which BOM's order-independent
  motif counts cannot capture
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: BagOfMotifs (BOM)
  role: R package whose predict_binary_multi() function performs held-out prediction
    and computes binary and multiclass classification performance metrics
  repo: https://github.com/ewonglab/BOM_package
- name: XGBoost
  role: Gradient-boosted tree algorithm underlying the binary and multiclass models
    being evaluated on held-out CRE sequences
- name: FIMO
  role: Scans CRE sequences for motifs whose occurrence counts feed the models being
    evaluated (set FIMO_updated = FALSE for FIMO < 5.5.4)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:c8da1eba257591386c2bb36d88e18b4888c859dd1f502629fa3db8124375f4fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BagOfMotifs (BOM)
  - FIMO
  - XGBoost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# held-out-classification-performance-evaluation

## Summary

Evaluates binary and multiclass cell-type classifiers of distal CREs on held-out sequences using precision, recall, F1, auROC and auPR, as implemented in BagOfMotifs::predict_binary_multi() for the mouse E8.25 snATAC-seq CRE classification task.

## When to use

Use when a trained sequence-based classifier (e.g. BOM/XGBoost on motif counts) must be scored on a held-out test set of labeled CRE sequences, and per-class and aggregate performance metrics are needed for validation.

## When NOT to use

- When the goal is motif interpretation (SHAP) rather than classification accuracy — use save_shap_multi()/shapPlots_multi() instead
- When input data are already feature/model-output tables and no held-out sequence labels exist to score against
- When the question concerns motif grammar or TF co-binding spacing, which BOM's order-independent motif counts cannot capture

## Inputs

- Trained binary and multiclass XGBoost models (from BagOfMotifs::binModel())
- Held-out test set of distal CRE sequences with known cell-type labels (2,489 sequences in the mouse E8.25 task)
- FIMO motif occurrence count table per CRE, scanned against gimme.vertebrate.v5.0.meme motifs

## Outputs

- bom_model_metrics.csv with reproduced binary and multiclass performance metrics (precision, recall, F1, auROC, auPR, assignment accuracy)
- Per-cell-type predicted labels for the binary (one-vs-rest) and 17-class multiclass settings

## How to apply

After training the binary and multiclass XGBoost models with BagOfMotifs::binModel() on FIMO-derived motif counts of distal CREs, call BagOfMotifs::predict_binary_multi() to predict held-out cell-type labels for both the binary and 17-class settings. Compute precision, recall, F1, auROC and auPR per cell type and overall, plus overall CRE assignment accuracy. Write the reproduced metrics to bom_model_metrics.csv. Accept the reproduction if held-out binary performance reaches auROC ≈ 0.98, auPR ≈ 0.98 and ≈ 93% CRE assignment accuracy, and multiclass performance reaches precision ≈ 0.99, recall ≈ 0.88 and auPR ≈ 0.99; deviations beyond these ranges indicate the model, motif scan, or train/test split deviates from the published protocol.

## Related tools

- **BagOfMotifs (BOM)** (R package whose predict_binary_multi() function performs held-out prediction and computes binary and multiclass classification performance metrics) — https://github.com/ewonglab/BOM_package
- **XGBoost** (Gradient-boosted tree algorithm underlying the binary and multiclass models being evaluated on held-out CRE sequences)
- **FIMO** (Scans CRE sequences for motifs whose occurrence counts feed the models being evaluated (set FIMO_updated = FALSE for FIMO < 5.5.4))

## Examples

```
BagOfMotifs::predict_binary_multi()  # predict held-out labels and compute binary/multiclass metrics
```

## Evaluation signals

- Held-out binary metrics reach auROC ≈ 0.98 and auPR ≈ 0.98 on the 2,489 held-out sequences
- Binary per-class average precision ≈ 0.93, recall ≈ 0.92, F1 ≈ 0.92, and ≈ 93% of CREs assigned to their cell type of origin
- Multiclass model reaches precision ≈ 0.99, recall ≈ 0.88 and auPR ≈ 0.99 across the 17 E8.25 cell types
- Reproduced metrics are written to bom_model_metrics.csv and match the published values within tolerance

## Limitations

- Multiclass recall (≈ 0.88) is notably lower than precision (≈ 0.99), so some cell types are under-predicted despite high auPR
- BOM relies on known TF motifs and neglects informative sequences not captured by current motif annotations
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin and enhancer activity can be influenced by regions beyond that sequence length

## Evidence

- [results] On 2,489 held-out sequences, BOM correctly assigned 93% of CREs to their cell type of origin, with average precision, recall and F1 of 0.93, 0.92 and 0.92 (auROC = 0.98; auPR = 0.98): [evidence span withheld — anchor status: not_found]
- [methods] Prediction and performance via predict_binary_multi: "# Prediction and performance
BagOfMotifs::predict_binary_multi()"
- [readme] BOM README on XGBoost classification performance: "Leveraging available TF binding motif profiles and the Extreme Gradient Boosting (XGBoost) algorithm, BOM has achieved high performance in classifying context-specific cis-regulatory elements."
