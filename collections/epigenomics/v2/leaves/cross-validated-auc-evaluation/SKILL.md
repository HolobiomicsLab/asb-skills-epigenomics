---
name: cross-validated-auc-evaluation
description: Use when you have labeled genomic region sets (e.g. HOT vs COLD) with
  per-region sequence features such as 2-4-mer frequencies, GC content, and CpG O/E
  ratio, and need an honest accuracy estimate via cross-validated AUC plus feature-importance
  rankings.
when_to_use_negative:
- Input is already a trained model or a precomputed feature-importance table rather
  than labeled HOT/COLD regions with sequence-derived features.
- The goal is functional or biological characterization of HOT regions (e.g. R-loop
  or G-quadruplex enrichment, expression stability of associated genes) rather than
  sequence-based HOT/COLD discrimination.
- Class labels are unavailable or the region sets are not HOT/COLD pairs, so no supervised
  discrimination can be trained.
tools:
- name: glmnet
  role: fits the species-specific elastic net classifiers with 10-fold cross-validation
    to tune the regularization parameter and yields coefficient-based feature importances
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:4d58f8f165c45c37201a13676167299f9206cd98241f7b06f0c8cfdb7bb9a3db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - glmnet
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cross-validated-auc-evaluation

## Summary

Trains species-specific elastic net classifiers with glmnet to discriminate HOT from COLD regions and validates them via 10-fold cross-validated AUC, reporting which sequence features (CpG/GC-rich k-mers, CpG O/E ratio) drive the discrimination.

## When to use

Use when you have labeled genomic region sets (e.g. HOT vs COLD) with per-region sequence features such as 2-4-mer frequencies, GC content, and CpG O/E ratio, and need an honest accuracy estimate via cross-validated AUC plus feature-importance rankings.

## When NOT to use

- Input is already a trained model or a precomputed feature-importance table rather than labeled HOT/COLD regions with sequence-derived features.
- The goal is functional or biological characterization of HOT regions (e.g. R-loop or G-quadruplex enrichment, expression stability of associated genes) rather than sequence-based HOT/COLD discrimination.
- Class labels are unavailable or the region sets are not HOT/COLD pairs, so no supervised discrimination can be trained.

## Inputs

- HOT and COLD region sets (BED/interval objects) from upstream peak clustering (e.g. task_001 output)
- corresponding genomic sequences for each region (FASTA or equivalent)
- per-region feature matrix: k-mer frequencies for k = 2, 3, 4, GC content, GC skew, CpG O/E ratio
- binary class labels (HOT vs COLD)

## Outputs

- four fitted elastic net classifiers (one per species)
- feature-importance table of top-ranked model coefficients
- cross-validated AUC values per classifier (expected 0.82-0.94)
- optimal regularization parameter (lambda) per model from 10-fold CV

## How to apply

Assemble a feature matrix from per-region k-mer frequency vectors (k = 2, 3, 4), GC content, GC skew, and CpG observed/expected ratio, with HOT/COLD class labels. Fit an elastic net logistic model with glmnet for each species, using 10-fold cross-validation to tune the regularization parameter (cv.glmnet). Extract the top-ranked coefficients into a feature-importance table and record the cross-validated AUC per classifier. Interpret the model only if AUCs are consistent with the reported 0.82-0.94 range and CpG/GC-rich k-mers plus the CpG O/E ratio rank among the top features; the paper's rationale is that these discriminatory features may explain the artificial ChIP-seq enrichment (hyper-ChIPability) of HOT regions.

## Related tools

- **glmnet** (fits the species-specific elastic net classifiers with 10-fold cross-validation to tune the regularization parameter and yields coefficient-based feature importances) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
cv.fit <- cv.glmnet(x = feature_matrix, y = hot_cold_labels, family = "binomial", type.measure = "auc", nfolds = 10); auc <- max(cv.fit$cvm)
```

## Evaluation signals

- Each cross-validated AUC falls within the reported 0.82-0.94 range.
- CpG/GC-rich k-mers rank among the top features of each fitted model.
- CpG observed/expected ratio ranks among the top features across all species.
- The regularization parameter lambda is selected by 10-fold cross-validation, not fixed arbitrarily.
- Feature matrix dimensions match: one row per HOT/COLD region, columns covering k = 2-4 k-mer frequencies plus GC content, GC skew, and CpG O/E ratio.

## Limitations

- The paper argues HOT regions are likely ChIP-seq artifacts and resemble 'hyper-ChIPable' regions, so the classifiers may partly detect sequence properties driving artifactual ChIP-seq enrichment rather than true TF binding.
- HOT regions lack canonical motifs for the TFs thought to be bound there, so motif-based predictors are not a substitute for the k-mer/CpG feature space used here.
- No cross-species comparison of HOT region sequence features existed to date, so the generalization of species-specific models across species is not established by this analysis.

## Evidence

- [results] Species-specific machine-learning models discriminate HOT from COLD regions with high accuracy (cross-validation AUC 0.82-0.94), and CpG/GC-rich k-mers together with CpG observed/expected ratio rank as the most important predictive features across all species.: [evidence span withheld — anchor status: not_found]
- [abstract] The paper's abstract reports the discriminatory sequence characteristics that motivate the feature set.: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [intro] The paper's intro states the gap the cross-species modeling addresses.: "To date, there has been no cross-species comparison of HOT regions in terms of sequence features."
- [readme] The README points to the analysis scripts used for the manuscript.: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
