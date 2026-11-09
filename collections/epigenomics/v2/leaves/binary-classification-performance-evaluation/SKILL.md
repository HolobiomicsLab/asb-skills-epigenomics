---
name: binary-classification-performance-evaluation
description: Use when a trained motif-count classifier (e.g., BOM/binModel on cell-type-specific
  enhancer BED labels) must be scored, and held-out predictions need F1, auPR, or
  regression correlation metrics.
when_to_use_negative:
- The model has not yet been trained — metrics cannot be computed before binModel()
  builds the XGBoost classifiers
- The task is purely interpretive (identifying important motifs) rather than predictive
  performance assessment — use save_shap_multi()/shapPlots_multi() instead
- Labels are multi-class without a binary/multi binary formulation supported by predict_binary_multi()
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: BagOfMotifs (BOM)
  role: R/XGBoost package whose predict_binary_multi() function runs prediction and
    computes F1, auPR, and Pearson correlation performance metrics
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: Upstream motif scanner whose output motif occurrences feed the count matrices
    that the evaluated classifiers are trained on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:0aadbc8ec22d5c440d3b4de3465dc24eed61d6c7bb4a4295fff9821efe9abf59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BagOfMotifs (BOM)
  - FIMO
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# binary-classification-performance-evaluation

## Summary

Evaluates BOM's gradient-boosted (XGBoost) cis-regulatory element classifiers by running prediction on held-out data and computing F1 and area under the precision-recall curve (auPR) per dataset, plus Pearson correlation for the Drosophila STARR-seq regression task.

## When to use

Use when a trained motif-count classifier (e.g., BOM/binModel on cell-type-specific enhancer BED labels) must be scored, and held-out predictions need F1, auPR, or regression correlation metrics.

## When NOT to use

- The model has not yet been trained — metrics cannot be computed before binModel() builds the XGBoost classifiers
- The task is purely interpretive (identifying important motifs) rather than predictive performance assessment — use save_shap_multi()/shapPlots_multi() instead
- Labels are multi-class without a binary/multi binary formulation supported by predict_binary_multi()

## Inputs

- Trained BOM models produced by BagOfMotifs::binModel() from motif-count matrices
- Four-column text files (chromosome, start, stop, condition) with enhancer coordinates and cell-type condition labels for each benchmark dataset
- FIMO motif scan results obtained with gimme.vertebrate.v5.0.meme against the appropriate genome (e.g., BSgenome.Mmusculus.UCSC.mm10)

## Outputs

- Per-dataset classification metrics: F1 score and area under the precision-recall curve (auPR)
- Pearson correlation coefficient for the Drosophila STARR-seq regression dataset
- Predicted labels/scores for held-out cis-regulatory elements

## How to apply

After training the gradient-boosted models with BagOfMotifs::binModel() on the motif-count matrices derived from FIMO scans (gimme.vertebrate.v5.0.meme), run BagOfMotifs::predict_binary_multi() to generate predictions and performance metrics for each dataset. For classification datasets (mouse ENCODE, human blood, zebrafish, AML) report F1 and auPR; for the Drosophila STARR-seq task report Pearson correlation since it is a regression. Judge correctness by comparing computed metrics to the reported benchmark values within reasonable tolerance: ENCODE F1 ~0.92 and auPR ~0.98, blood F1 ~0.90, zebrafish F1 ~0.96, Drosophila Pearson ~0.78/0.42, AML F1 ~0.99. Use the same four-column input BED structure (chromosome, start, stop, condition) and the genome-matched BSgenome package to keep the evaluation comparable across the benchmark.

## Related tools

- **BagOfMotifs (BOM)** (R/XGBoost package whose predict_binary_multi() function runs prediction and computes F1, auPR, and Pearson correlation performance metrics) — https://github.com/ewonglab/BOM_package
- **FIMO** (Upstream motif scanner whose output motif occurrences feed the count matrices that the evaluated classifiers are trained on)

## Examples

```
library(BagOfMotifs); BagOfMotifs::binModel(); BagOfMotifs::predict_binary_multi()
```

## Evaluation signals

- Computed F1 and auPR for the ENCODE dataset match reported values within reasonable tolerance (F1 ~0.92, auPR ~0.98)
- Blood, zebrafish, and AML F1 scores are close to reported ~0.90, ~0.96, and ~0.99 respectively
- Drosophila STARR-seq Pearson correlation is close to reported ~0.78/0.42
- Metrics are reproducible when the pipeline (generateAllFasta → runFIMO → binModel → predict_binary_multi) is rerun on the same input BED files
- Predictions cover all condition labels present in the four-column input files

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence; open chromatin, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations, and TFs do not bind all their motifs
- BOM's simple motif-count representation does not capture motif grammar (order, orientation, spacing), which remains an important future direction

## Evidence

- [methods] Prediction and performance metrics computed with predict_binary_multi to obtain F1, auPR (classification datasets) and Pearson correlation (Drosophila STARR-seq regression) per dataset: [evidence span withheld — anchor status: not_found]
- [abstract] This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets.: "This minimalist representation, combined with gradient-boosted trees, enables the accurate prediction of cell-type-specific enhancers across mouse, human, zebrafish, and Arabidopsis datasets."
- [other] Validation metrics must match reported benchmark values within reasonable tolerance, e.g. ENCODE F1 ~0.92 and auPR ~0.98.: [evidence span withheld — anchor status: not_found]
- [readme] BOM is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states and interprets motif importance via SHAP scores.: "BAG-of-Motifs (BOM) is an R package that uses XGBoost to construct predictive models of cis-regulatory sequences between cell states."
