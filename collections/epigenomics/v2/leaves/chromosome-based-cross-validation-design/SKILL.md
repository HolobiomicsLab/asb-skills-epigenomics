---
name: chromosome-based-cross-validation-design
description: Use when evaluating enhancer-target interaction prediction models on
  genomic coordinate data (e.g. BENGI All-Pairs.Natural-Ratio ET pairs) where random
  folds would leak spatially autocorrelated loci between train and test chromosomes.
when_to_use_negative:
- The input is a single pre-computed feature table with no genomic coordinates, so
  chromosome-based splitting cannot be defined
- The goal is hyperparameter tuning on one cell line (e.g. the GM12878 consensus LCL
  training set) rather than an unbiased outer evaluation
- The question does not concern spatial generalization — e.g. evaluating feature importances
  on the full training set
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: CENTRE
  role: Pre-trained XGBoost-based ET prediction framework evaluated under the chromosome-split
    12-fold outer CV scheme
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Classifier trained and scored within each chromosome-based outer CV fold (logistic
    objective, scale_pos_weight = 5)
- name: TargetFinder
  role: Benchmark classifier (GradientBoostingClassifier, 303 EPW features) run under
    the same chromosome-split CV scheme for comparison
  repo: https://github.com/shwhalen/targetfinder
- name: BENGI
  role: Ground-truth benchmark collection supplying the labeled All-Pairs.Natural-Ratio
    datasets and the 12 chromosome-split CV groups
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:75ab2497d0c9b2cb259e160984df356d1538a8bf1757eac4f6cc18a2d7046d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - BENGI
  - CENTRE
  - R
  - TargetFinder
  - XGBoost
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chromosome-based-cross-validation-design

## Summary

A validation design for enhancer-target interaction classifiers (e.g. CENTRE's XGBoost model) in which the 12 cross-validation groups are split by chromosome, so held-out test folds always lie in different genomic regions than the training folds. This prevents genomic-region leakage when evaluating ET pair prediction on BENGI datasets.

## When to use

Use when evaluating enhancer-target interaction prediction models on genomic coordinate data (e.g. BENGI All-Pairs.Natural-Ratio ET pairs) where random folds would leak spatially autocorrelated loci between train and test chromosomes.

## When NOT to use

- The input is a single pre-computed feature table with no genomic coordinates, so chromosome-based splitting cannot be defined
- The goal is hyperparameter tuning on one cell line (e.g. the GM12878 consensus LCL training set) rather than an unbiased outer evaluation
- The question does not concern spatial generalization — e.g. evaluating feature importances on the full training set

## Inputs

- Processed All-Pairs.Natural-Ratio BENGI dataset tables with ET pairs (cCRE-ELS enhancer, GENCODE v40 TSS target), labels, and per-pair genomic coordinates
- The 12 chromosome-based CV group assignments shipped with the BENGI datasets (split by chromosome)
- Feature matrices (CT-specific features such as RNA-seq TPM and CRUP-EP/CRUP-PP-derived regulatory distances, and generic features such as genomic Distance and Fisher's combined P-value)

## Outputs

- Per-fold held-out predictions (ET pair, predicted interaction probability, label) restricted to test chromosomes
- Per-dataset AUPRC and F1-score computed from the pooled outer-CV predictions
- Method-dataset comparison table (e.g. CENTRE vs TargetFinder vs Random Forest) reproducing Figure 4A/B-style rankings

## How to apply

Adopt the 12 cross-validation groups split by chromosome provided with the BENGI datasets (Moore et al. 2020), assigning each fold whole chromosomes so that testing is always performed in different genomic regions than training. Restrict inputs to the processed All-Pairs.Natural-Ratio BENGI datasets (hg38 cCREs-ELS vs GENCODE v40 TSS pairs filtered to within 500 KB, positives with TPM > 0, negatives matched by TSS or cCRE-ELS). Train the classifier (e.g. XGBoost with logistic objective, scale_pos_weight = 5, learning_rate = 0.1, max_depth = 5, n_estimators = 300, subsample = 0.9, colsample_bytree = 0.7) inside the chromosome-split outer folds without any hyperparameter tuning in the outer folds. For each method-dataset combination, pool held-out fold predictions and compute AUPRC and F1-score to yield the per-dataset comparison. The rationale is that enhancer-promoter contacts cluster in 3D genomic space, so chromosome-level separation is required for an honest estimate of generalization to unseen genomic regions.

## Related tools

- **CENTRE** (Pre-trained XGBoost-based ET prediction framework evaluated under the chromosome-split 12-fold outer CV scheme) — https://github.com/slrvv/CENTRE
- **XGBoost** (Classifier trained and scored within each chromosome-based outer CV fold (logistic objective, scale_pos_weight = 5))
- **TargetFinder** (Benchmark classifier (GradientBoostingClassifier, 303 EPW features) run under the same chromosome-split CV scheme for comparison) — https://github.com/shwhalen/targetfinder
- **BENGI** (Ground-truth benchmark collection supplying the labeled All-Pairs.Natural-Ratio datasets and the 12 chromosome-split CV groups)

## Examples

```
XGBClassifier(objective='binary:logistic', scale_pos_weight=5, learning_rate=0.1, max_depth=5, n_estimators=300, subsample=0.9, colsample_bytree=0.7, gamma=1.0, reg_lambda=0, random_state=0)
```

## Evaluation signals

- Every test fold contains chromosomes absent from every training fold — verify no ET pair appears in both train and test splits
- Exactly 12 outer-CV groups are used, matching the chromosome-based CV groups provided by Moore et al. 2020
- No hyperparameter optimization occurs inside outer folds; parameters are fixed (e.g. scale_pos_weight = 5, n_estimators = 300)
- AUPRC and F1 are reported per dataset from pooled held-out fold predictions, and the resulting method ordering matches the paper's reported equivalence/superiority of CENTRE over TargetFinder (11 of 13 datasets in Fig. 4A)
- All evaluated ET pairs are within 500 KB and satisfy the positive TPM > 0 / negative-matching filter before splitting

## Limitations

- F1 scores remain low on many datasets, and the choice of validation data itself strongly influences any performance measurement
- Evaluation is restricted to ET pairs within 500 KB; performance beyond this interval is untested
- CENTRE's pre-trained model was optimized on the whole consensus LCL dataset, so results on unrelated tissues depend on that training choice

## Evidence

- [results] the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [methods] We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE: "We used the All-Pairs.Natural-Ratio BENGI collection of datasets for the training and evaluation of CENTRE"
- [methods] We kept only ET pairs that are located within 500 KB: "We kept only ET pairs that are located within 500 KB"
- [results] ET feature vectors are then fed to a pre-trained XGBOOST classifier, and a probability of an interaction is assigned to ET pairs: "ET feature vectors are then fed to a pre-trained XGBOOST classifier, and a probability of an interaction is assigned to ET pairs"
