---
name: precision-recall-performance-metrics-computation
description: Use when comparing binary enhancer-target (or other paired genomic) interaction
  classifiers on imbalanced BENGI-style datasets and you need AUPRC/F1 from chromosome-based
  cross-validated held-out predictions.
when_to_use_negative:
- You only have a single train/test split rather than the BENGI 12 chromosome-based
  CV folds — fold-concatenated held-out predictions are required for this evaluation
  scheme
- You need ROC-AUC or accuracy-style metrics; the article's benchmark is reported
  specifically in AUPRC and F1-score, which suit the class-imbalanced Natural-Ratio
  pairs
- Your task is training or hyperparameter optimization of the classifiers rather than
  computing benchmark metrics from held-out predictions
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CENTRE
  role: Pre-trained XGBoost classifier (logistic regression objective, scale_pos_weight=5)
    whose held-out BENGI fold predictions are scored with AUPRC/F1; pairs with probability
    > 0.5 are labeled interacting
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Provides the probability scores over BENGI ET pairs that are thresholded and
    aggregated per fold before metric computation
  repo: https://github.com/slrvv/CENTRE
- name: TargetFinder
  role: Baseline GradientBoostingClassifier (n_estimators=4000, learning_rate=0.1,
    max_depth=5, max_features='log2') reimplemented on hg19 BENGI annotation; its
    held-out predictions are scored with the same AUPRC/F1 metrics
  repo: https://github.com/shwhalen/targetfinder
- name: R
  role: Language of the ready-to-use CENTRE package used to generate held-out predictions
    and assemble the per-dataset comparison table
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:fdf599c45ea1f5d032f846761f081527fd9fd02920d3beeef80a13611719d288
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - R
  - TargetFinder
  - XGBoost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# precision-recall-performance-metrics-computation

## Summary

Computes AUPRC and F1-score for enhancer-target prediction methods from held-out cross-validation folds, as used to benchmark CENTRE against TargetFinder and Random Forest on the All-Pairs.Natural-Ratio BENGI datasets.

## When to use

Use when comparing binary enhancer-target (or other paired genomic) interaction classifiers on imbalanced BENGI-style datasets and you need AUPRC/F1 from chromosome-based cross-validated held-out predictions.

## When NOT to use

- You only have a single train/test split rather than the BENGI 12 chromosome-based CV folds — fold-concatenated held-out predictions are required for this evaluation scheme
- You need ROC-AUC or accuracy-style metrics; the article's benchmark is reported specifically in AUPRC and F1-score, which suit the class-imbalanced Natural-Ratio pairs
- Your task is training or hyperparameter optimization of the classifiers rather than computing benchmark metrics from held-out predictions

## Inputs

- Held-out prediction vectors (interaction probabilities or class labels) per method per cross-validation fold
- Ground-truth interaction labels from the All-Pairs.Natural-Ratio BENGI collection (labeled 'interacting' or 'not interacting', hg38 cCREs-ELS vs GENCODE v40 TSS pairs within 500 KB)
- 12 chromosome-based cross-validation group assignments provided with the BENGI datasets
- Trained classifier outputs: CENTRE XGBoost, TargetFinder GradientBoostingClassifier, reduced TargetFinder (13 features), and Random Forest predictions

## Outputs

- Per-dataset AUPRC values for every method-dataset combination
- Per-dataset F1-score values for every method-dataset combination
- Per-dataset comparison table reproducing the Figure 4A/4B ordering of CENTRE vs TargetFinder vs Random Forest

## How to apply

Run each classifier (e.g. CENTRE's pre-trained XGBoost model with logistic regression objective and scale_pos_weight=5, TargetFinder's GradientBoostingClassifier with n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', or a Random Forest with n_estimators=300, max_depth=5, max_samples=0.9, class_weight='balanced_subsample') using the 12 BENGI cross-validation groups split by chromosome so testing always occurs on different genomic regions than training. Binarize CENTRE/XGBoost outputs using the probability > 0.5 threshold to label interacting pairs. From the concatenated held-out outer CV fold predictions for every method-dataset combination, compute AUPRC and F1-score, and assemble a per-dataset comparison table across the 13 BENGI datasets and the five eQTL tissue datasets. Validate by checking that the resulting table reproduces the reported ordering (CENTRE outperforming TargetFinder in 11 of 13 datasets in Fig. 4A and the reduced TargetFinder model on five eQTL tissues in Fig. 4B).

## Related tools

- **CENTRE** (Pre-trained XGBoost classifier (logistic regression objective, scale_pos_weight=5) whose held-out BENGI fold predictions are scored with AUPRC/F1; pairs with probability > 0.5 are labeled interacting) — https://github.com/slrvv/CENTRE
- **XGBoost** (Provides the probability scores over BENGI ET pairs that are thresholded and aggregated per fold before metric computation) — https://github.com/slrvv/CENTRE
- **TargetFinder** (Baseline GradientBoostingClassifier (n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2') reimplemented on hg19 BENGI annotation; its held-out predictions are scored with the same AUPRC/F1 metrics) — https://github.com/shwhalen/targetfinder
- **R** (Language of the ready-to-use CENTRE package used to generate held-out predictions and assemble the per-dataset comparison table) — https://github.com/slrvv/CENTRE

## Examples

```
library(CENTRE); preds <- centrePrediction(features_generic, features_celltype, model); labels <- preds$probability > 0.5
```

## Evaluation signals

- AUPRC and F1 are computed strictly from held-out outer CV folds of the 12 chromosome-based groups, never from training-fold predictions
- CENTRE/XGBoost probabilities thresholded at 0.5 yield the binary labels used for F1: 'ET pairs with higher probability than 0.5 are labeled as interacting pairs'
- The assembled per-dataset table reproduces the Fig. 4A ordering: CENTRE outperforms TargetFinder in 11 out of 13 BENGI datasets
- On the five eQTL tissue datasets, the table matches the Fig. 4B result where CENTRE significantly outperforms the reduced TargetFinder model (13 features: DNase, H3K4me3, H3K27ac, CTCF plus distance)
- All ET pairs evaluated are within the 500 KB distance threshold and carry BENGI ground-truth 'interacting'/'not interacting' labels

## Limitations

- F1 scores remain low on many BENGI datasets; performance depends strongly on the choice of validation data ('even the choice of validation data strongly influences any performance measurement')
- Benchmark comparisons require mapping CENTRE hg38 feature values onto the hg19 BENGI annotation (averaged over pairs mapping to the same hg19 pair), which can blur pair-level precision
- ET pair search and evaluation are restricted to interactions within 500 KB; performance outside this interval is not measured

## Evidence

- [results] ET pairs with probability higher than 0.5 are labeled as interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
- [results] 12-fold chromosome-based CV on BENGI: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [methods] BENGI ground truth labels and All-Pairs.Natural-Ratio collection: "We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE"
- [methods] keep only pairs within 500 KB: "We kept only ET pairs that are located within 500 KB"
- [readme] README: pre-trained model usage threshold context: "CENTRE is a machine learning framework that predicts enhancer target 
interactions in a cell-type-specific manner, using only gene expression and 
ChIP-seq data for three histone modifications for"
