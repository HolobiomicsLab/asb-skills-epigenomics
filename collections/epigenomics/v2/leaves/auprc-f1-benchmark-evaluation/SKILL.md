---
name: auprc-f1-benchmark-evaluation
description: Use when benchmarking a binary enhancer-target interaction classifier
  (e.g., XGBoost on BENGI All-Pairs.Natural-Ratio pairs) and you need AUPRC/F1 estimates
  free of genomic leakage via chromosome-split cross-validation.
when_to_use_negative:
- Pairs extend beyond 500 KB or long-range interactions are of interest — this evaluation
  inherits the article's 500 KB search limit, and the authors state extending the
  interval requires future methodological improvements
- You plan to use random (non-chromosome) folds — random shuffling places genomically
  proximal pairs in both train and test, breaking the leakage guarantee and comparability
  with the Moore et al. 2020 results
- There are no labeled ground-truth interactions (e.g., prediction on a new cell type
  without BENGI labels) — this skill scores classifiers against existing labels; it
  does not biologically validate predictions
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: XGBoost
  role: 'Train-and-score engine: the XGBClassifier (logistic-regression objective,
    scale_pos_weight = 5, fixed GM12878-optimized hyperparameters) is fit in each
    chromosome-split fold and its interaction probabilities produce the AUPRC/F1 benchmark
    numbers'
- name: CENTRE
  role: 'Framework under evaluation: its centrePrediction() probabilities and the
    CT-specific/generic/combined feature conditions are the objects being scored by
    the AUPRC/F1 comparison against the distance baseline'
  repo: https://github.com/slrvv/CENTRE
- name: TargetFinder
  role: 'Source of the adopted evaluation protocol: the BENGI 12 cross-validation
    groups split by chromosome (Moore et al. 2020) that prevent testing in the same
    genomic regions as training'
  repo: https://github.com/shwhalen/targetfinder
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:b86185509939304f7dc9ac276bc269e16281f288e28d9016d90eba9b214c2bf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - R
  - TargetFinder
  - XGBoost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# auprc-f1-benchmark-evaluation

## Summary

Scores enhancer-target interaction classifiers on the All-Pairs.Natural-Ratio BENGI datasets using AUPRC and F1-score under the Moore et al. 2020 12-fold chromosome-split cross-validation scheme, so test folds always cover different genomic regions than training. Anchors every comparison to the genomic-distance baseline and the CT-specific vs generic vs combined feature-set ablation.

## When to use

Use when benchmarking a binary enhancer-target interaction classifier (e.g., XGBoost on BENGI All-Pairs.Natural-Ratio pairs) and you need AUPRC/F1 estimates free of genomic leakage via chromosome-split cross-validation.

## When NOT to use

- Pairs extend beyond 500 KB or long-range interactions are of interest — this evaluation inherits the article's 500 KB search limit, and the authors state extending the interval requires future methodological improvements
- You plan to use random (non-chromosome) folds — random shuffling places genomically proximal pairs in both train and test, breaking the leakage guarantee and comparability with the Moore et al. 2020 results
- There are no labeled ground-truth interactions (e.g., prediction on a new cell type without BENGI labels) — this skill scores classifiers against existing labels; it does not biologically validate predictions

## Inputs

- All-Pairs.Natural-Ratio BENGI ET-pair datasets (hg38 cCRE-ELS vs GENCODE v40 TSS, pairs within 500 KB, positives with TPM > 0, negatives matched to positives by TSS or cCRE-ELS)
- Per-pair feature matrices under three conditions: CT-specific only (CRUP-EP/CRUP-PP and their derived regulatory distances, RNA-seq TPM), generic only (Distance, cor_CRUP, combined_tests), and the combined full vector
- 12 chromosome-split cross-validation group assignments from the Moore et al. 2020 / BENGI scheme
- XGBoost model configuration (logistic objective, scale_pos_weight = 5, GM12878-optimized hyperparameters) or the pre-trained CENTRE model

## Outputs

- Per-dataset AUPRC and F1-score table for each feature condition (CT-specific, generic, combined) plus the genomic-distance baseline
- Feature-importance ranking across the nine feature positions (distance 1st, Wilcoxon tests combined 6th, CRUP correlation 9th)
- Fold-level probability predictions thresholded at 0.5 (interacting/not interacting) suitable for precision-recall curve construction

## How to apply

Split each All-Pairs.Natural-Ratio BENGI dataset (hg38 cCRE-ELS enhancers vs GENCODE v40 TSS, pairs restricted to 500 KB, positives with TPM > 0, negatives matched by TSS or cCRE-ELS) into the 12 CV groups provided by Moore et al. 2020, which are split by chromosome so that testing is always performed in different genomic regions than training. In each outer fold, train the XGBoost XGBClassifier with a logistic-regression objective for binary classification, scale_pos_weight = 5, and the GM12878-optimized settings (random_state = 0, colsample_bytree = 0.7, gamma = 1.0, learning_rate = 0.1, max_depth = 5, n_estimators = 300, reg_lambda = 0, subsample = 0.9), keeping hyperparameters fixed with no tuning inside the outer folds. Threshold predicted probabilities at 0.5 to label interacting pairs, then compute AUPRC (primary metric) and F1-score per dataset and per condition. Always run the genomic-distance baseline alongside each feature condition — CT-specific only (CRUP-EP/CRUP-PP-derived regulatory distances, RNA-seq TPM), generic only (Distance, cor_CRUP Pearson correlation, combined_tests Fisher's combined P-value) and the combined full vector — and tabulate the comparison. Judge correctness by the paper's reported ordering: the combined feature vector achieves the highest AUPRC, generic features alone are inferior even to the distance baseline on GM12878, and the feature ranking places genomic distance 1st, Wilcoxon tests combined 6th, CRUP correlation 9th.

## Related tools

- **XGBoost** (Train-and-score engine: the XGBClassifier (logistic-regression objective, scale_pos_weight = 5, fixed GM12878-optimized hyperparameters) is fit in each chromosome-split fold and its interaction probabilities produce the AUPRC/F1 benchmark numbers)
- **CENTRE** (Framework under evaluation: its centrePrediction() probabilities and the CT-specific/generic/combined feature conditions are the objects being scored by the AUPRC/F1 comparison against the distance baseline) — https://github.com/slrvv/CENTRE
- **TargetFinder** (Source of the adopted evaluation protocol: the BENGI 12 cross-validation groups split by chromosome (Moore et al. 2020) that prevent testing in the same genomic regions as training) — https://github.com/shwhalen/targetfinder

## Examples

```
pred <- centrePrediction(features_generic, features_celltype, model)
```

## Evaluation signals

- Under the combined feature vector, AUPRC must exceed both the CT-specific-only and generic-only conditions across the BENGI datasets, reproducing the paper's ordering
- On GM12878, both generic features (Fisher's combined probability, CRUP Pearson correlation) must score below the genomic-distance baseline
- Feature ranking should place genomic distance at position 1, Wilcoxon tests combined at position 6, CRUP correlation at position 9, with positions 2-5 occupied by CT-specific features (RNA-seq, Normalized Regulatory distance enhancers, ...)
- Fold-integrity check: the 12 CV groups are chromosome-disjoint, so no chromosome appears in both a training and a test fold, and no hyperparameter tuning occurs inside the outer folds
- F1 is computed at the 0.5 probability threshold (ET pairs with higher probability than 0.5 are labeled as interacting pairs) with the fixed XGBoost configuration (random_state = 0, colsample_bytree = 0.7, gamma = 1.0, learning_rate = 0.1, max_depth = 5, n_estimators = 300, reg_lambda = 0, subsample = 0.9)

## Limitations

- F1 scores remain low on many BENGI datasets and performance depends strongly on the choice of validation data, per the article's discussion
- Evaluation is bounded by the 500 KB interaction window; extending this interval is left to future methodological improvements
- Positive labels derive from specific assays such as GM12878 RNAPII-ChIAPET within BENGI, so AUPRC/F1 reflect the biases of that ground-truth source

## Evidence

- [methods] We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE: "We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE"
- [results] the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [results] ET pairs with higher probability than 0.5 are labeled as interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
- [results] Combining all the orthogonal variates ... into a single feature vector, CENTRE achieves the highest AUPRC: [evidence span withheld — anchor status: not_found]
- [results] both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method: "both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method"
- [discussion] even the choice of validation data strongly influences any performance measurement.: "even the choice of validation data strongly influences any performance measurement."
