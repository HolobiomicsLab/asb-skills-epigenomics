---
name: enhancer-target-generic-feature-computation
description: Use when building training/prediction features for enhancer-target pairs
  from BENGI-style ET pair lists and you need cell-type-independent statistics (Wilcoxon
  rank-sum P-values, Fisher combination, CRUP correlation) to complement CT-specific
  ChIP-seq/RNA-seq features.
when_to_use_negative:
- Input is already a feature table with generic features computed (use centrePrediction
  directly)
- The analysis goal is cell-type-specific prediction for a single cell line only —
  generic features are cell-type agnostic and were shown to perform worse than genomic
  distance on GM12878
- Interactions beyond 500 KB of the target gene — CENTRE restricts ET pair search
  to 500 KB
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3169
tools:
- name: CENTRE
  role: R package providing computeGenericFeatures(pairs) and the PrecomputedData.db
    of precomputed Wilcoxon rank-sum tests
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-EP
  role: Source of enhancer probabilities per 100-bp bin across 104 ENCODE cell types
    used for the cor_CRUP Pearson correlation feature
- name: CRUP-PP
  role: Source of promoter probabilities summed over the 5-bin target region for the
    cor_CRUP correlation
- name: XGBoost
  role: Classifier used to evaluate each generic feature individually with 12-fold
    chromosome-split CV, recording AUPRC per feature
- name: R
  role: Implementation language for CENTRE's feature computation and classification
    pipeline
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:5a870c241d9403934231f236cf9a036d99bd7df5e7c1e57608353f46aae5ce2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - CRUP-EP
  - CRUP-PP
  - R
  - XGBoost
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# enhancer-target-generic-feature-computation

## Summary

Computation of cell-type-agnostic (generic) features for candidate enhancer-target (ET) pairs: a Fisher's combined probability from Wilcoxon rank-sum tests across public enhancer-activity datasets, and a Pearson correlation of CRUP-EP/CRUP-PP probabilities across 104 cell types, used to complement CENTRE's cell-type-specific features.

## When to use

Use when building training/prediction features for enhancer-target pairs from BENGI-style ET pair lists and you need cell-type-independent statistics (Wilcoxon rank-sum P-values, Fisher combination, CRUP correlation) to complement CT-specific ChIP-seq/RNA-seq features.

## When NOT to use

- Input is already a feature table with generic features computed (use centrePrediction directly)
- The analysis goal is cell-type-specific prediction for a single cell line only — generic features are cell-type agnostic and were shown to perform worse than genomic distance on GM12878
- Interactions beyond 500 KB of the target gene — CENTRE restricts ET pair search to 500 KB

## Inputs

- All-Pairs.Natural-Ratio BENGI ET pair collection (lifted to hg38 and overlapped with ENCODE cCRE-ELS)
- PrecomputedData.db SQLite database of precomputed Wilcoxon rank-sum tests (CAGE-seq Andersson 2014, DNase hypersensitivity Thurman 2012, DNase-seq gene expression Sheffield 2013)
- CRUP-EP/CRUP-PP probability tables per 100-bp bin for 104 ENCODE cell types (In_house_constructed_datasets.zip)
- GENCODE gene/cCRE-ELS pair dataframe

## Outputs

- combined_tests feature: negative log Fisher's combined P-value per ET pair
- cor_CRUP feature: Pearson correlation coefficient of summed CRUP-EP/CRUP-PP probabilities across 104 cell types per ET pair
- Generic feature matrix indexed by ET pair for input to the XGBoost classifier

## How to apply

Load the processed All-Pairs.Natural-Ratio BENGI ET pairs and the precomputed CRUP-EP/CRUP-PP probability tables for 104 ENCODE cell types (In_house_constructed_datasets.zip). For each ET pair, run a Wilcoxon rank-sum test comparing the target gene's signal between cell types where the enhancer is active (top quantile 0.25 of DNase/DHS activity) versus inactive, across four public datasets: FANTOM5 CAGE enhancers (Andersson et al. 2014), DNase-hypersensitive counts (Thurman et al. 2012), DNase-matched microarray gene expression (Sheffield et al. 2013), and CRUP-EP across 66 matched ENCODE cell types. Combine the four P-values with Fisher's method and use the negative logarithm of the combined P-value as the combined_tests feature. For the cor_CRUP feature, sum CRUP-EP probabilities over the 5-bin enhancer region and CRUP-PP probabilities over the 5-bin target region, then compute the Pearson correlation across the 104 cell types. In the CENTRE R package these are computed via computeGenericFeatures(pairs), which uses the PrecomputedData.db SQLite database of precomputed Wilcoxon rank-sum tests.

## Related tools

- **CENTRE** (R package providing computeGenericFeatures(pairs) and the PrecomputedData.db of precomputed Wilcoxon rank-sum tests) — https://github.com/slrvv/CENTRE
- **CRUP-EP** (Source of enhancer probabilities per 100-bp bin across 104 ENCODE cell types used for the cor_CRUP Pearson correlation feature)
- **CRUP-PP** (Source of promoter probabilities summed over the 5-bin target region for the cor_CRUP correlation)
- **XGBoost** (Classifier used to evaluate each generic feature individually with 12-fold chromosome-split CV, recording AUPRC per feature)
- **R** (Implementation language for CENTRE's feature computation and classification pipeline) — https://github.com/slrvv/CENTRE

## Examples

```
R: features_generic <- computeGenericFeatures(pairs)
```

## Evaluation signals

- On GM12878, both generic features perform poorly and are inferior to the baseline distance method; the illustration case TTC39C-EH38E1904551 has a Pearson correlation of only 0.11
- Feature ranking check: genomic distance first, Wilcoxon tests combined (combined_tests) sixth, CRUP correlation (cor_CRUP) ninth under XGBoost 12-fold chromosome-split CV on GM12878
- Fisher's combined probability must be the negative logarithm of the combined P-value from exactly four Wilcoxon rank-sum tests
- cor_CRUP must be computed from probabilities summed over 5-bin regions (5 x 100-bp bins) and correlated across 104 cell types

## Limitations

- Correlation alone has low predictive value: the average Pearson correlation from the same datasets underperforms the Wilcoxon-based Fisher's combined probability for classifying ET interactions
- Both generic features perform poorly on GM12878 and are inferior to the genomic distance baseline
- Generic features are cell-type agnostic and cannot capture cell-type-specific enhancer activity

## Evidence

- [results] by performing a Wilcoxon rank-sum test between target transcriptional signals between cells where the enhancer is active and those where it is not active: "by performing a Wilcoxon rank-sum test between target transcriptional signals between cells where the enhancer is active and those where it is not active"
- [methods] Fisher's combined probability: "We used the negative logarithm of the combined P-value as the final feature in our classification"
- [methods] The function `computeGenericFeatures(pairs)` computes the features that are not cell type specific: "The function `computeGenericFeatures(pairs)` computes the features that are not cell type specific"
- [results] resulting in a Pearson Correlation Coefficient of only 0.11: "resulting in a Pearson Correlation Coefficient of only 0.11"
- [readme] PrecomputedData.db is a database containing precomputed Wilcoxon rank sum tests on the following data sets: [evidence span withheld — anchor status: external_doc]
