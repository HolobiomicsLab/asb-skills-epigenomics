---
name: class-imbalanced-binary-classification
description: Use when labeling enhancer-target pairs as interacting/not-interacting
  on imbalanced BENGI All-Pairs.Natural-Ratio datasets, or when comparing boosted-tree
  classifiers (CENTRE/XGBoost, TargetFinder GBM, Random Forest) under chromosome-split
  cross-validation with AUPRC and F1.
when_to_use_negative:
- The prediction problem is ranking-free binary accuracy reporting on naturally-ratio
  data, where the class skew makes accuracy misleading — use AUPRC/F1 instead
- Only generic correlation features (Fisher's combined probability, Pearson correlation)
  are available and no cell-type-specific signal exists — these were inferior even
  to the distance baseline on GM12878
- The interaction space extends beyond the 500 KB window accepted by CENTRE's feature
  extraction
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CENTRE
  role: Provides the pre-trained imbalance-aware XGBoost classifier (logistic regression
    objective, scale_pos_weight=5) applied to ET feature vectors
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Extreme gradient boosting backend used with a logistic regression objective
    and scale_pos_weight=5 to handle the imbalanced ET classes
- name: TargetFinder
  role: Benchmark classifier reimplemented as GradientBoostingClassifier (n_estimators=4000,
    learning_rate=0.1, max_depth=5, max_features='log2') for comparison on the same
    imbalanced datasets
  repo: https://github.com/shwhalen/targetfinder
- name: R
  role: Language in which the ready-to-use pre-trained CENTRE framework is distributed
    (Bioconductor package)
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:9113a7faa4022a69bd0adb7fb671744d861dff25f6c514dd38b60d56dd224bd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - R
  - TargetFinder
  - XGBoost
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Class-imbalanced binary classification

## Summary

Train and evaluate gradient-boosting classifiers on heavily imbalanced enhancer-target (ET) interaction datasets, using imbalance-aware settings (e.g., scale_pos_weight=5 in XGBoost, class_weight='balanced_subsample' in Random Forest) and AUPRC/F1-score under chromosome-based cross-validation.

## When to use

Use when labeling enhancer-target pairs as interacting/not-interacting on imbalanced BENGI All-Pairs.Natural-Ratio datasets, or when comparing boosted-tree classifiers (CENTRE/XGBoost, TargetFinder GBM, Random Forest) under chromosome-split cross-validation with AUPRC and F1.

## When NOT to use

- The prediction problem is ranking-free binary accuracy reporting on naturally-ratio data, where the class skew makes accuracy misleading — use AUPRC/F1 instead
- Only generic correlation features (Fisher's combined probability, Pearson correlation) are available and no cell-type-specific signal exists — these were inferior even to the distance baseline on GM12878
- The interaction space extends beyond the 500 KB window accepted by CENTRE's feature extraction

## Inputs

- Labeled ET pair feature table from BENGI All-Pairs.Natural-Ratio datasets (hg38 cCRE-ELS vs GENCODE v40 TSS pairs, distance-filtered to 500 KB)
- 12 cross-validation groups split by chromosome
- Feature vectors from CENTRE (generic + cell-type-specific features) or TargetFinder (303 EPW features or 13 reduced features)

## Outputs

- Per-pair interaction probability from the pre-trained XGBoost classifier
- Binary interacting/non-interacting labels (probability > 0.5)
- Per-dataset AUPRC and F1-score table comparing CENTRE, TargetFinder, and Random Forest over outer CV folds

## How to apply

Assemble a labeled ET pair feature table (cCRE-ELS to GENCODE v40 TSS pairs within 500 KB) with class ratios taken from the BENGI All-Pairs.Natural-Ratio ground truth, then apply the 12 chromosome-based cross-validation groups so testing always occurs on different genomic regions than training. Run the pre-trained CENTRE XGBoost classifier (logistic regression objective, scale_pos_weight=5) without further hyperparameter tuning, and reimplement TargetFinder's GradientBoostingClassifier (n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2') plus a Random Forest baseline (n_estimators=300, max_depth=5, max_samples=0.9, class_weight='balanced_subsample') for comparison. Threshold predicted probabilities at 0.5 to call interacting pairs. Compute AUPRC and F1-score per method-dataset combination from the pooled held-out folds rather than accuracy, since the natural-ratio class balance makes accuracy uninformative.

## Related tools

- **CENTRE** (Provides the pre-trained imbalance-aware XGBoost classifier (logistic regression objective, scale_pos_weight=5) applied to ET feature vectors) — https://github.com/slrvv/CENTRE
- **XGBoost** (Extreme gradient boosting backend used with a logistic regression objective and scale_pos_weight=5 to handle the imbalanced ET classes)
- **TargetFinder** (Benchmark classifier reimplemented as GradientBoostingClassifier (n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2') for comparison on the same imbalanced datasets) — https://github.com/shwhalen/targetfinder
- **R** (Language in which the ready-to-use pre-trained CENTRE framework is distributed (Bioconductor package)) — https://github.com/slrvv/CENTRE

## Examples

```
estimator = GradientBoostingClassifier(n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', random_state=0); scores = cross_val_score(estimator, predictors_df, labels, scoring='f1', cv=cv, n_jobs=-1)
```

## Evaluation signals

- AUPRC and F1-score are computed from held-out outer CV folds, and the per-dataset table should reproduce the reported Fig. 4A/B ordering in which CENTRE outperforms TargetFinder in 11 of 13 BENGI datasets
- Cross-validation must use the 12 chromosome-based groups so test regions never overlap training regions; a random (shuffled) fold split would inflate scores
- Interacting-pair calls must be made at the 0.5 probability threshold, as in the paper ('ET pairs with higher probability than 0.5 are labeled as interacting pairs')
- Baseline sanity check: generic features (Fisher's combined probability, Pearson correlation) should score worse than the distance baseline on GM12878 — if not, feature construction is wrong

## Limitations

- F1 scores remain low on many datasets and the choice of validation data strongly influences performance measurements
- BENGI ground truth derives from Hi-C/ChIA-PET assays whose loop-calling is unstable, limiting label reliability
- The classification scope is restricted to interactions within 500 KB; extending this interval requires future methodological improvements
- Performance comparison depends on the annotation harmonization step (hg38 feature values averaged over pairs mapping to the same hg19 pair for TargetFinder runs)

## Evidence

- [results] Pre-trained XGBoost assigns interaction probability: "Then a pre-trained extreme gradient boosting classifier (Chen and Guestrin 2016) computes a probability for an annotated ET pair to interact"
- [results] Chromosome-based cross-validation groups: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [results] Probability threshold for positive calls: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
- [readme] TargetFinder GBM parameters from README: [evidence span withheld — anchor status: external_doc]
- [discussion] Validation-data sensitivity caveat: "even the choice of validation data strongly influences any performance measurement."
