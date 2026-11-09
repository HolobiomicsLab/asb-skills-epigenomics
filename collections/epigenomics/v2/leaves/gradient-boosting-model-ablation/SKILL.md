---
name: gradient-boosting-model-ablation
description: Use when you must quantify whether a feature group — e.g., the four regulatory-density
  window features — adds predictive signal to a gradient-boosted enhancer-target classifier
  by comparing full vs RD-ablated models.
when_to_use_negative:
- Input is raw H3K27ac/H3K4me1/H3K4me3 ChIP-seq BAMs rather than the processed BENGI
  feature tables — run the CRUP-EP/CRUP-PP feature-computation steps first
- ET pairs of interest extend beyond 500 KB, outside the interval covered by the All-Pairs.Natural-Ratio
  BENGI tables and CENTRE's pair search
- No BENGI interacting/not-interacting labels are available, so the chromosome-split
  outer-CV AUPRC delta cannot be computed or validated
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: XGBoost
  role: Pre-trained extreme gradient boosting classifier trained on the full and RD-ablated
    feature vectors to output ET interaction probabilities for the AUPRC comparison
- name: CENTRE
  role: R framework supplying the pre-trained model, feature computation (computeGenericFeatures/computeCellTypeFeatures)
    and BENGI pair handling used for the ablation
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-EP
  role: Outputs per-100-bp-bin enhancer probabilities over the enhancer-promoter window
    used to compute reg_dist_enh and norm_reg_dist_enh
- name: CRUP-PP
  role: Supplies promoter probabilities over the intervening window used to compute
    reg_dist_prom and norm_reg_dist_prom
- name: crupR
  role: R dependency that generates the CRUP enhancer/promoter probabilities consumed
    by CENTRE's RD feature computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:986976dc8674c3dcc96fe0fb8bdd3d041af0ebe5fe43fafb73aa8e8249a3f378
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - CRUP-EP
  - CRUP-PP
  - XGBoost
  - crupR
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gradient-boosting-model-ablation

## Summary

Ablation protocol that trains CENTRE's XGBoost classifier on All-Pairs.Natural-Ratio BENGI enhancer-target feature tables with and without the four CRUP-derived regulatory-distance (RD) window features, then compares chromosome-split cross-validated AUPRC to quantify the RD features' cell-type-specific contribution.

## When to use

Use when you must quantify whether a feature group — e.g., the four regulatory-density window features — adds predictive signal to a gradient-boosted enhancer-target classifier by comparing full vs RD-ablated models.

## When NOT to use

- Input is raw H3K27ac/H3K4me1/H3K4me3 ChIP-seq BAMs rather than the processed BENGI feature tables — run the CRUP-EP/CRUP-PP feature-computation steps first
- ET pairs of interest extend beyond 500 KB, outside the interval covered by the All-Pairs.Natural-Ratio BENGI tables and CENTRE's pair search
- No BENGI interacting/not-interacting labels are available, so the chromosome-split outer-CV AUPRC delta cannot be computed or validated

## Inputs

- Processed All-Pairs.Natural-Ratio BENGI feature tables of ET pairs within 500 KB, with BENGI ground-truth interacting/not-interacting labels and 12 chromosome-split CV groups
- CRUP-EP enhancer and CRUP-PP promoter probabilities per 100-bp genomic bin for the cell type of interest (e.g., LCL/GM12878)
- Full CENTRE feature vectors (generic features plus cell-type-specific features such as RNA-seq, Wilcoxon tests, CRUP correlation) for each ET pair

## Outputs

- rd_ablation_results.csv with per-BENGI-dataset outer-CV average precision/AUPRC for the full vs RD-ablated XGBoost models
- RD-stratified table of predicted interaction probabilities for ET pairs (pairs with probability > 0.5 labeled interacting)

## How to apply

Load the processed All-Pairs.Natural-Ratio BENGI feature tables and keep only ET pairs located within 500 KB. Compute the four regulatory-distance features (reg_dist_enh, norm_reg_dist_enh, reg_dist_prom, norm_reg_dist_prom) as counts of 100-bp bins with CRUP-EP or CRUP-PP probability > 0.5 in the enhancer-promoter window, with the normalized variants dividing by the total number of bins. Train the XGBoost classifier (logistic objective, scale_pos_weight = 5, colsample_bytree = 0.7, learning_rate = 0.1, max_depth = 5, n_estimators = 300, subsample = 0.9) twice — once on the full feature vector and once with the four RD features removed — using the BENGI 12-fold chromosome-split outer cross-validation so testing is always performed in different genomic regions than training. Record outer-CV average precision/AUPRC per BENGI dataset for both models; the full-minus-ablated delta isolates the RD contribution. As a secondary check, label ET pairs with probability > 0.5 as interacting and stratify predicted probabilities by normalized regulatory distance to confirm that low intervening regulatory activity favors specific interactions (e.g., LCL-interacting pairs show smaller RD in LCLs than in HeLa). Write the per-dataset AUPRC comparison and the RD-stratified probability summary to rd_ablation_results.csv.

## Related tools

- **XGBoost** (Pre-trained extreme gradient boosting classifier trained on the full and RD-ablated feature vectors to output ET interaction probabilities for the AUPRC comparison)
- **CENTRE** (R framework supplying the pre-trained model, feature computation (computeGenericFeatures/computeCellTypeFeatures) and BENGI pair handling used for the ablation) — https://github.com/slrvv/CENTRE
- **CRUP-EP** (Outputs per-100-bp-bin enhancer probabilities over the enhancer-promoter window used to compute reg_dist_enh and norm_reg_dist_enh)
- **CRUP-PP** (Supplies promoter probabilities over the intervening window used to compute reg_dist_prom and norm_reg_dist_prom)
- **crupR** (R dependency that generates the CRUP enhancer/promoter probabilities consumed by CENTRE's RD feature computation)

## Examples

```
library(xgboost); bst <- xgboost(as.matrix(X), label=y, objective="binary:logistic", eta=0.1, max_depth=5, nrounds=300, subsample=0.9, colsample_bytree=0.7, scale_pos_weight=5)
```

## Evaluation signals

- On GM12878 RNAPII-ChIAPET, the full-feature model's outer-CV average precision must exceed the RD-ablated model's, consistent with normalized regulatory distance being a top-ranked cell-type-specific feature (position 3 overall)
- Identical 12-fold chromosome-split folds are used for both models; the reduced vector must contain every feature except reg_dist_enh, norm_reg_dist_enh, reg_dist_prom and norm_reg_dist_prom
- rd_ablation_results.csv contains a per-BENGI-dataset AUPRC row for both the full and ablated models, with positive deltas on datasets where RD carries cell-type-specific signal
- All retained pairs are within 500 KB and RD features are counts of 100-bp bins with CRUP-EP/CRUP-PP probability > 0.5, with normalized variants bounded in [0,1]
- RD-stratified probabilities reproduce the reported direction: pairs interacting in LCLs but not HeLa have smaller intervening regulatory activity in LCLs than in HeLa

## Limitations

- The ET-pair search is limited to 500 KB; extending this interval requires future methodological improvements
- F1/AUPRC remain low on many BENGI datasets and even the choice of validation data strongly influences any performance measurement
- Genomic distance still ranks first among features, so RD-ablation deltas may be small on datasets where distance dominates
- RD annotation relies on only three histone marks (H3K27ac, H3K4me1, H3K4me3); more HMs could improve cCRE/RD annotation but require extra experiments

## Evidence

- [results] We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters: "We apply CRUP-EP and CRUP-PP to the window between the two regulatory elements and extract the fraction of regions classified as enhancers or promoters"
- [methods] We kept only ET pairs that are located within 500 KB: "We kept only ET pairs that are located within 500 KB"
- [results] the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training: "the authors also provide 12 cross-validation (CV) groups split by chromosome. This ensures that testing is always performed in different genomic regions than training"
- [results] ET pairs with higher probability than 0.5 are labeled as interacting pairs: "ET pairs with higher probability than 0.5 are labeled as interacting pairs"
- [results] Combining all the orthogonal variates ... into a single feature vector, CENTRE achieves the highest AUPRC: [evidence span withheld — anchor status: not_found]
- [results] Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features: "Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features"
- [readme] CENTRE is a machine learning framework that predicts enhancer target interactions in a cell-type-specific manner: "CENTRE is a machine learning framework that predicts enhancer target interactions in a cell-type-specific manner"
- [discussion] even the choice of validation data strongly influences any performance measurement.: "even the choice of validation data strongly influences any performance measurement."
