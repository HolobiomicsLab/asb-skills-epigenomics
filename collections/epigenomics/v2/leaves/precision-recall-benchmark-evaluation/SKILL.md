---
name: precision-recall-benchmark-evaluation
description: Use when benchmarking a cell-type-specific enhancer-target prediction
  model against BENGI ground truth, or when quantifying the contribution of a feature
  group (e.g. regulatory distance) via AUPRC ablation.
when_to_use_negative:
- Input is not a labeled pair-level benchmark (e.g. unannotated cCRE-ELS enhancer-gene
  pairs without BENGI ground truth) — AUPRC evaluation requires known interacting/non-interacting
  labels
- Evaluating predictions in a cell type lacking interaction ground truth — the article
  notes that the choice of validation data strongly influences performance measurements
- Feature table has already been reduced to final predictions without per-pair probabilities,
  preventing probability-thresholded (p > 0.5) classification and AUPRC computation
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: CENTRE
  role: Machine learning framework whose XGBoost classifier produces ET-pair interaction
    probabilities evaluated by AUPRC under chromosome-split CV
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Extreme gradient boosting classifier (logistic objective) whose predicted
    probabilities are scored with average precision / AUPRC
  repo: https://github.com/slrvv/CENTRE
- name: TargetFinder
  role: Best-performing prior method used as a benchmark comparison across all BENGI
    datasets
  repo: https://github.com/shwhalen/targetfinder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:68ad7283d4499aaf4bef236dcf70d0b98de0c695e80df774946e43031108f6bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - TargetFinder
  - XGBoost
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# precision-recall-benchmark-evaluation

## Summary

Evaluates enhancer-target interaction classifiers using chromosome-split cross-validation and average precision (AUPRC) on the All-Pairs.Natural-Ratio BENGI ground-truth datasets, including ablation comparisons (e.g. with/without regulatory-distance features) to quantify feature contributions.

## When to use

Use when benchmarking a cell-type-specific enhancer-target prediction model against BENGI ground truth, or when quantifying the contribution of a feature group (e.g. regulatory distance) via AUPRC ablation.

## When NOT to use

- Input is not a labeled pair-level benchmark (e.g. unannotated cCRE-ELS enhancer-gene pairs without BENGI ground truth) — AUPRC evaluation requires known interacting/non-interacting labels
- Evaluating predictions in a cell type lacking interaction ground truth — the article notes that the choice of validation data strongly influences performance measurements
- Feature table has already been reduced to final predictions without per-pair probabilities, preventing probability-thresholded (p > 0.5) classification and AUPRC computation

## Inputs

- All-Pairs.Natural-Ratio BENGI feature tables (labeled interacting / not interacting ET pairs, hg38)
- Feature vectors including CT-specific features (RNA-seq, normalized regulatory distance enhancers/promoters) and generic features (genomic distance, Fisher's combined probability, CRUP correlation)
- BENGI 12 cross-validation groups split by chromosome

## Outputs

- Per-dataset outer-CV average precision / AUPRC values for full and ablated models (e.g. rd_ablation_results.csv)
- RD-stratified predicted interaction probability summary (interacting when probability > 0.5)

## How to apply

Load the processed All-Pairs.Natural-Ratio BENGI feature tables, keeping only ET pairs within 500 KB. Use the chromosome-split 12-fold outer cross-validation scheme provided with the BENGI datasets so that testing always occurs in different genomic regions than training. Train an XGBoost classifier (logistic objective, scale_pos_weight = 5, colsample_bytree = 0.7, learning_rate = 0.1, max_depth = 5, n_estimators = 300, subsample = 0.9) on the full feature vector and on a reduced vector with the ablated features removed. Record outer-CV average precision / AUPRC for both models per dataset, and label pairs with predicted probability > 0.5 as interacting when tabulating predictions (e.g. stratified by normalized regulatory distance). Write per-dataset AUPRC comparisons to a results table such as rd_ablation_results.csv.

## Related tools

- **CENTRE** (Machine learning framework whose XGBoost classifier produces ET-pair interaction probabilities evaluated by AUPRC under chromosome-split CV) — https://github.com/slrvv/CENTRE
- **XGBoost** (Extreme gradient boosting classifier (logistic objective) whose predicted probabilities are scored with average precision / AUPRC) — https://github.com/slrvv/CENTRE
- **TargetFinder** (Best-performing prior method used as a benchmark comparison across all BENGI datasets) — https://github.com/shwhalen/targetfinder

## Examples

```
scores <- rd_ablation(xgb_params = list(objective='binary:logistic', scale_pos_weight=5, max_depth=5, n_estimators=300), cv_groups=bengi_12fold, out='rd_ablation_results.csv')
```

## Evaluation signals

- The full-feature model must outperform the ablated model (e.g. higher average precision with vs. without the four RD features) on the GM12878 RNAPII-ChIAPET evaluation, consistent with normalized regulatory distance being a top-ranked cell-type-specific feature
- Cross-validation must be chromosome-split: testing performed in different genomic regions than training using the 12 CV groups from BENGI
- Feature importance ranking should place genomic distance first, with positions 2-5 occupied by CT-specific features (e.g. RNA-seq at position 2, Normalized Regulatory distance enhancers at position 3)
- ET pairs with predicted probability higher than 0.5 should be labeled as interacting, matching the paper's classification convention
- Reported AUPRC for the combined-feature CENTRE model should be the highest across feature-source combinations on GM12878 RNAPII-ChIAPET

## Limitations

- F1 scores remain low on many datasets; performance depends strongly on the choice of validation data
- Search is restricted to ET pairs within 500 KB, so benchmark results do not cover longer-range interactions
- BENGI ET pairs required liftover from hg19 to hg38 before overlapping with the ENCODE Registry of cCREs v2

## Evidence

- [results] Evaluate using 12-fold cross-validation split by chromosome on BENGI datasets with F1-score: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [results] Probability threshold for labeling interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
- [results] AUPRC maximized by combined feature vector: [evidence span withheld — anchor status: not_found]
- [results] Feature ranking placing genomic distance first: "Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features, while positions 2 (RNA-seq), 3 (Normalized Regulatory distance enhancers),"
- [discussion] Performance depends on validation data choice: "even the choice of validation data strongly influences any performance measurement."
- [readme] TargetFinder cross-validation and AUPRC-style scoring in README: [evidence span withheld — anchor status: external_doc]
