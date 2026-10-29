---
name: xgboost-gradient-boosting-model-training
description: Use when a labeled enhancer-promoter pair table (ET pairs with CT-specific
  and/or generic features) needs binary interaction classification, or when reproducing
  feature-set ablations (CT-specific vs generic vs combined) scored by AUPRC and F1.
when_to_use_negative:
- Input is already a trained model or prediction probability vector — this skill covers
  feature-matrix-to-classifier training, not post-hoc scoring
- Only generic (cell-type-agnostic) features are available and the goal is competitive
  prediction — these perform worse than the baseline distance method on GM12878
- The cell line lacks high-resolution interaction training data and the goal is cross-cell-line
  transfer — per the TargetFinder README, ChIP-seq variability prevents learning in
  one cell line and predicting in a new one
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: XGBoost
  role: Pre-trained extreme gradient boosting classifier used to compute the interaction
    probability for annotated ET pairs; XGBClassifier with logistic objective and
    scale_pos_weight = 5
- name: CENTRE
  role: Framework providing the feature computation (computeGenericFeatures, computeCellTypeFeatures)
    and the pre-trained model consumed by this training workflow
  repo: https://github.com/slrvv/CENTRE
- name: TargetFinder
  role: Prior best-performing method whose datasets and chromosome-split 12-fold CV
    scheme (Moore et al. 2020) are adopted for evaluation
  repo: https://github.com/shwhalen/targetfinder
- name: R
  role: Environment in which CENTRE is distributed as ready-to-use R software (Bioconductor-style
    package with xgboost dependency)
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:8acc6aa0afc8cc231625ee29be77cdb94a9dfe26664f3b8c2bd5339bc2a34dfe
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# xgboost-gradient-boosting-model-training

## Summary

Train an extreme gradient boosting (XGBoost) classifier to predict cell-type-specific enhancer-target (ET) interactions from CENTRE feature vectors, using chromosome-split 12-fold cross-validation on the All-Pairs.Natural-Ratio BENGI collection and AUPRC/F1 as the evaluation metrics.

## When to use

Use when a labeled enhancer-promoter pair table (ET pairs with CT-specific and/or generic features) needs binary interaction classification, or when reproducing feature-set ablations (CT-specific vs generic vs combined) scored by AUPRC and F1.

## When NOT to use

- Input is already a trained model or prediction probability vector — this skill covers feature-matrix-to-classifier training, not post-hoc scoring
- Only generic (cell-type-agnostic) features are available and the goal is competitive prediction — these perform worse than the baseline distance method on GM12878
- The cell line lacks high-resolution interaction training data and the goal is cross-cell-line transfer — per the TargetFinder README, ChIP-seq variability prevents learning in one cell line and predicting in a new one

## Inputs

- Processed All-Pairs.Natural-Ratio BENGI datasets (ET pairs on hg38 cCREs-ELS and GENCODE v40 TSS, restricted to pairs within 500 KB, positives with TPM > 0, negatives matched by TSS or cCRE-ELS)
- Feature matrix of CT-specific features (CRUP-EP/CRUP-PP probabilities and regulatory distances, RNA-seq TPM)
- Feature matrix of generic features (genomic distance, cor_CRUP Pearson correlation, Fisher's combined P-value)
- Binary interaction labels from the BENGI ground truth ('interacting' / 'not interacting')

## Outputs

- Trained XGBoost classifier assigning interaction probabilities to ET pairs
- Per-dataset, per-feature-condition AUPRC and F1-score tabulation across the three feature conditions

## How to apply

Assemble the feature matrices for each condition: CT-specific features only (CRUP-EP and CRUP-EP-derived regulatory distances, CRUP-PP and CRUP-PP-derived regulatory distances, RNA-seq TPM), generic features only (genomic distance, cor_CRUP Pearson correlation, Fisher's combined P-value as negative log), and the combined full vector. Train an XGBoost XGBClassifier with logistic regression objective for binary classification, scale_pos_weight = 5, and the GM12878-optimized parameters random_state = 0, colsample_bytree = 0.7, gamma = 1.0, learning_rate = 0.1, max_depth = 5, n_estimators = 300, reg_lambda = 0, subsample = 0.9. Evaluate with the 12-fold outer cross-validation scheme adopted from Moore et al. 2020, split by chromosome so testing is always performed in different genomic regions than training, with no hyperparameter tuning in the outer folds. Compute AUPRC and F1-score per dataset and per feature condition; ET pairs with predicted probability higher than 0.5 are labeled as interacting. The expected ordering is that combining all orthogonal feature sources yields the highest AUPRC, while generic features alone are inferior even to the baseline distance method.

## Related tools

- **XGBoost** (Pre-trained extreme gradient boosting classifier used to compute the interaction probability for annotated ET pairs; XGBClassifier with logistic objective and scale_pos_weight = 5)
- **CENTRE** (Framework providing the feature computation (computeGenericFeatures, computeCellTypeFeatures) and the pre-trained model consumed by this training workflow) — https://github.com/slrvv/CENTRE
- **TargetFinder** (Prior best-performing method whose datasets and chromosome-split 12-fold CV scheme (Moore et al. 2020) are adopted for evaluation) — https://github.com/shwhalen/targetfinder
- **R** (Environment in which CENTRE is distributed as ready-to-use R software (Bioconductor-style package with xgboost dependency)) — https://github.com/slrvv/CENTRE

## Examples

```
clf = xgb.XGBClassifier(objective='binary:logistic', scale_pos_weight=5, random_state=0, colsample_bytree=0.7, gamma=1.0, learning_rate=0.1, max_depth=5, n_estimators=300, reg_lambda=0, subsample=0.9); clf.fit(X_train, y_train)
```

## Evaluation signals

- The combined feature condition should achieve the highest AUPRC across the BENGI datasets, reproducing the paper's reported ordering of feature conditions
- Generic-features-only models should score below the baseline genomic-distance method (as reported on GM12878)
- CV folds must be split by chromosome so testing regions differ from training regions; verify no pair's chromosome appears in both train and test folds
- Predicted probabilities above 0.5 must map to 'interacting' labels when computing F1
- Class imbalance handling must be visible in the model config (scale_pos_weight = 5) given the natural-ratio negative sampling

## Limitations

- F1 scores remain low on many datasets; performance depends strongly on the choice of validation data
- The 500 KB distance threshold caps the interaction search space; extending this interval would require methodological improvements
- Hyperparameters were optimized on GM12878 (the consensus LCL training set); transfer to other cell types may require retuning
- Genomic distance holds the leading position in most datasets, so feature rankings can be dominated by distance rather than CT-specific signals

## Evidence

- [results] Then a pre-trained extreme gradient boosting classifier (Chen and Guestrin 2016) computes a probability for an annotated ET pair to interact: "Then a pre-trained extreme gradient boosting classifier (Chen and Guestrin 2016) computes a probability for an annotated ET pair to interact"
- [results] the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [methods] We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE: "We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE"
- [results] both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method: "both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method"
- [results] ET pairs with higher probability than 0.5 are labeled as interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
