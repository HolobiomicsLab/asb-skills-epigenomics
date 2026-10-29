---
name: gradient-boosting-classifier-reimplementation
description: Use when benchmarking a new enhancer-target prediction method against
  TargetFinder or Random Forest baselines and you need identically parameterized GBM/RF
  comparisons on BENGI All-Pairs.Natural-Ratio datasets with AUPRC/F1 outputs.
when_to_use_negative:
- The goal is to train a new enhancer-target predictor from user-supplied RNA-seq
  plus H3K27ac/H3K4me1/H3K4me3 ChIP-seq rather than to benchmark against baselines
  — use CENTRE's own package functions instead
- Only cell lines with high-resolution interaction data are available but you need
  cross-cell-line prediction — the TargetFinder README explicitly warns the model
  cannot be learned on one cell line and applied to a new cell line lacking such data
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TargetFinder
  role: Reference method whose GradientBoostingClassifier configuration (n_estimators=4000,
    learning_rate=0.1, max_depth=5, max_features='log2', 303 EPW features) is reimplemented
    as the main benchmark baseline
  repo: https://github.com/shwhalen/targetfinder
- name: XGBoost
  role: Pre-trained extreme gradient boosting classifier used by CENTRE (logistic
    regression objective, scale_pos_weight=5) that is evaluated under the same CV
    scheme for comparison
  repo: https://github.com/slrvv/CENTRE
- name: R
  role: Language in which the pre-trained CENTRE framework is provided as ready-to-use
    software and in which the hg38-to-hg19 feature averaging is performed
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:8180905de0bc266f6104e9c5aa448b2bc50b23230750e2eaeae71d41456bb788
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shwhalen/targetfinder
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
yes_rate: 0.8
n_claims: 15
---

# gradient-boosting-classifier-reimplementation

## Summary

Reimplements the TargetFinder gradient boosting classifier (and a Random Forest baseline) with original hyperparameters on BENGI enhancer-promoter pairs so that CENTRE's XGBoost performance can be compared fairly against established methods under the same chromosome-based cross-validation scheme.

## When to use

Use when benchmarking a new enhancer-target prediction method against TargetFinder or Random Forest baselines and you need identically parameterized GBM/RF comparisons on BENGI All-Pairs.Natural-Ratio datasets with AUPRC/F1 outputs.

## When NOT to use

- The goal is to train a new enhancer-target predictor from user-supplied RNA-seq plus H3K27ac/H3K4me1/H3K4me3 ChIP-seq rather than to benchmark against baselines — use CENTRE's own package functions instead
- Only cell lines with high-resolution interaction data are available but you need cross-cell-line prediction — the TargetFinder README explicitly warns the model cannot be learned on one cell line and applied to a new cell line lacking such data

## Inputs

- Processed All-Pairs.Natural-Ratio BENGI datasets (hg38 cCREs-ELS vs GENCODE v40 TSS pairs, distance-filtered to 500 KB)
- 12 cross-validation groups split by chromosome per BENGI dataset
- TargetFinder pre-generated EPW training data (e.g. paper/targetfinder/<cell line>/output-epw/training.csv.gz or training.h5) on hg19 annotation
- hg19-to-hg38 pair mapping for averaging CENTRE feature values over duplicated pairs
- Reduced 13-feature set (DNase, H3K4me3, H3K27ac, CTCF assays plus distance) for tissue eQTL datasets

## Outputs

- Per-method per-dataset AUPRC and F1-score computed from held-out outer CV folds
- Per-dataset comparison table of CENTRE vs full TargetFinder vs reduced TargetFinder vs Random Forest
- Fig. 4A/4B-style ordering showing whether CENTRE achieves equivalent or superior performance

## How to apply

Reimplement TargetFinder's full model as a scikit-learn GradientBoostingClassifier with n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2' over the 303 enhancer-promoter-window (EPW) features for LCLs, HeLa, K562, IMR-90 and NHEK, using the hg19 BENGI annotation with CENTRE hg38 feature values averaged over pairs mapping to the same hg19 pair. For the five tissue datasets lacking full genomic data, run the reduced TargetFinder model with 13 features from DNase, H3K4me3, H3K27ac, CTCF assays plus distance. Train the Random Forest baseline (n_estimators=300, max_depth=5, max_samples=0.9, class_weight='balanced_subsample') with analogous parameters. Evaluate every method-dataset combination using the 12 cross-validation groups split by chromosome so testing always occurs in different genomic regions than training, then compute AUPRC and F1-score from the held-out outer CV folds. The rationale is that TargetFinder was shown to be the best-performing prior method across all datasets, so reproducing its exact configuration is required for an apples-to-apples comparison with the pre-trained CENTRE XGBoost classifier (logistic regression objective, scale_pos_weight=5) run without further hyperparameter optimization.

## Related tools

- **TargetFinder** (Reference method whose GradientBoostingClassifier configuration (n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', 303 EPW features) is reimplemented as the main benchmark baseline) — https://github.com/shwhalen/targetfinder
- **XGBoost** (Pre-trained extreme gradient boosting classifier used by CENTRE (logistic regression objective, scale_pos_weight=5) that is evaluated under the same CV scheme for comparison) — https://github.com/slrvv/CENTRE
- **R** (Language in which the pre-trained CENTRE framework is provided as ready-to-use software and in which the hg38-to-hg19 feature averaging is performed) — https://github.com/slrvv/CENTRE

## Examples

```
estimator = GradientBoostingClassifier(n_estimators=4000, learning_rate=0.1, max_depth=5, max_features='log2', random_state=0); scores = cross_val_score(estimator, predictors_df, labels, scoring='f1', cv=cv, n_jobs=-1)
```

## Evaluation signals

- The per-dataset table reproduces the reported Fig. 4A ordering in which CENTRE outperforms TargetFinder in 11 out of 13 BENGI datasets
- On the five eQTL tissue datasets, CENTRE significantly outperforms the reduced 13-feature TargetFinder model (Fig. 4B)
- Each method-dataset AUPRC/F1 is computed only from held-out outer CV folds, with testing chromosomes disjoint from training chromosomes
- Full TargetFinder runs use exactly 303 EPW features; reduced TargetFinder runs use exactly 13 features (DNase, H3K4me3, H3K27ac, CTCF plus distance)
- CENTRE is run without further hyperparameter optimization, so its AUPRC must match the pre-trained model's behavior on each dataset

## Limitations

- F1 scores remain low on many datasets, and the choice of validation data strongly influences any performance measurement
- CENTRE's comparison is restricted to ET pairs within 500 KB, a commonly accepted threshold
- CENTRE requires only four CT-specific experiments (RNA-seq plus three histone marks); full TargetFinder integrates hundreds of cell-specific genomics datasets and is applicable to only a few richly annotated cell lines
- Full TargetFinder can only be run on LCLs, HeLa, K562, IMR-90 and NHEK; tissue datasets require the reduced model because required genomic data are missing

## Evidence

- [results] TargetFinder (Whalen et al. 2016) which was shown to be the best-performing method across all datasets: "TargetFinder (Whalen et al. 2016) which was shown to be the best-performing method across all datasets"
- [results] chromosome-based CV design: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [methods] TargetFinder hyperparameters and hg19 averaging workflow: "the generate_training.py script and the corresponding datasets provided on the Github page (https://github.com/shwhalen/targetfinder)"
- [readme] README sklearn GBM configuration: [evidence span withheld — anchor status: external_doc]
- [results] pre-trained XGBoost classifier role in CENTRE: "Then a pre-trained extreme gradient boosting classifier (Chen and Guestrin 2016) computes a probability for an annotated ET pair to interact"
