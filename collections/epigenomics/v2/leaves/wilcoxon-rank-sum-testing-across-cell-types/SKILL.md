---
name: wilcoxon-rank-sum-testing-across-cell-types
description: Use when building cell-type-agnostic features for enhancer-target interaction
  classification and you have enhancer activity and target gene expression measured
  across many cell types (e.g. 104 ENCODE CTs, 112 DHS tissues).
when_to_use_negative:
- You need cell-type-specific prediction for a single cell line — these are generic,
  cell-type-agnostic features; use computeCellTypeFeatures (RNA-seq, H3K27ac/H3K4me1/H3K4me3
  ChIP-seq) instead.
- The enhancer of interest has no activity measurements across enough cell types to
  form active/inactive groups for the rank-sum test.
- You only have a correlation between enhancer activity and gene expression available
  and expect it to classify ET interactions — the paper shows correlation alone (e.g.
  TTC39C-EH38E1904551 Pearson r = 0.11) has low predictive value.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3173
tools:
- name: CENTRE
  role: Framework whose computeGenericFeatures() step performs the Wilcoxon rank-sum
    tests and Fisher combination; precomputed tests are stored in PrecomputedData.db
  repo: https://github.com/slrvv/CENTRE
- name: CRUP-EP
  role: Provides enhancer probability per 100-bp bin across cell types, defining active/inactive
    groups and the enhancer-side signal for the cor_CRUP contrast
- name: CRUP-PP
  role: Provides promoter probability summed over the 5-bin target region for the
    Pearson correlation feature
- name: XGBoost
  role: Pre-trained gradient boosting classifier used to evaluate each generic feature
    individually via AUPRC under 12-fold chromosome-split CV
- name: R
  role: Language in which CENTRE and its feature computation functions are delivered
    (Bioconductor-style package, metapod for P-value combination)
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:fbabe5f338858cfb974d26e9741779678ec0661b02fc5132035a424bd01e5dcd
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# wilcoxon-rank-sum-testing-across-cell-types

## Summary

A feature-engineering procedure from CENTRE that tests, per enhancer-target pair, whether target gene signal differs between cell types where the enhancer is active versus inactive, using Wilcoxon rank-sum tests on multiple public activity datasets combined by Fisher's method as a cell-type-agnostic predictor.

## When to use

Use when building cell-type-agnostic features for enhancer-target interaction classification and you have enhancer activity and target gene expression measured across many cell types (e.g. 104 ENCODE CTs, 112 DHS tissues).

## When NOT to use

- You need cell-type-specific prediction for a single cell line — these are generic, cell-type-agnostic features; use computeCellTypeFeatures (RNA-seq, H3K27ac/H3K4me1/H3K4me3 ChIP-seq) instead.
- The enhancer of interest has no activity measurements across enough cell types to form active/inactive groups for the rank-sum test.
- You only have a correlation between enhancer activity and gene expression available and expect it to classify ET interactions — the paper shows correlation alone (e.g. TTC39C-EH38E1904551 Pearson r = 0.11) has low predictive value.

## Inputs

- Processed All-Pairs.Natural-Ratio BENGI ET pairs (liftover to hg38, overlapped with ENCODE cCRE-ELS)
- Precomputed CRUP-EP/CRUP-PP probability tables across 104 ENCODE cell types (In_house_constructed_datasets.zip / PrecomputedData.db)
- FANTOM5 CAGE enhancer activity (Andersson et al. 2014)
- DNase-hypersensitivity counts across 112 cell types (Thurman et al. 2012) with top quantile 0.25 defining high enhancer activity
- DNase-matched microarray gene expression (Sheffield et al. 2013)
- Target gene expression (TPM) per cell type

## Outputs

- combined_tests feature per ET pair (negative log Fisher's combined P-value)
- cor_CRUP feature per ET pair (Pearson correlation of summed CRUP-EP vs CRUP-PP probabilities across 104 cell types)
- Per-feature AUPRC from XGBoost 12-fold chromosome-split cross-validation on GM12878

## How to apply

For each candidate ET pair, split the panel of cell types into active and inactive groups using enhancer activity thresholds (e.g. top quantile 0.25 of enhancer DHS counts, FANTOM5 CAGE enhancers, CRUP-EP across 66 matched ENCODE cell types), then run a Wilcoxon rank-sum test comparing the target gene signal between the two groups for each dataset. Combine the four resulting P-values with Fisher's method and take the negative logarithm to yield the combined_tests feature. Optionally compute the contrasting cor_CRUP feature by summing CRUP-EP probabilities over the 5-bin enhancer region and CRUP-PP probabilities over the 5-bin target region and correlating across the 104 cell types with Pearson. The rationale is that a rank-based two-group comparison carries more predictive value than correlation alone; evaluate each feature individually with XGBoost under 12-fold chromosome-split cross-validation on BENGI (All-Pairs.Natural-Ratio, GM12878) and expect it to rank below genomic distance.

## Related tools

- **CENTRE** (Framework whose computeGenericFeatures() step performs the Wilcoxon rank-sum tests and Fisher combination; precomputed tests are stored in PrecomputedData.db) — https://github.com/slrvv/CENTRE
- **CRUP-EP** (Provides enhancer probability per 100-bp bin across cell types, defining active/inactive groups and the enhancer-side signal for the cor_CRUP contrast)
- **CRUP-PP** (Provides promoter probability summed over the 5-bin target region for the Pearson correlation feature)
- **XGBoost** (Pre-trained gradient boosting classifier used to evaluate each generic feature individually via AUPRC under 12-fold chromosome-split CV)
- **R** (Language in which CENTRE and its feature computation functions are delivered (Bioconductor-style package, metapod for P-value combination)) — https://github.com/slrvv/CENTRE

## Examples

```
R> pairs <- createPairs(...); generic <- computeGenericFeatures(pairs)  # computes combined_tests (Fisher-combined Wilcoxon P-values) and cor_CRUP
```

## Evaluation signals

- Feature ranking on GM12878 should place genomic distance first, Wilcoxon tests combined (combined_tests) sixth, and CRUP correlation (cor_CRUP) ninth.
- Both generic features should perform worse than the baseline distance method on GM12878 RNAPII-ChIAPET (inferior AUPRC).
- Negative log of Fisher's combined P-value must be finite and larger for pairs where target expression separates cleanly between active/inactive enhancer cell-type groups.
- Sanity-check the contrast on the TTC39C-EH38E1904551 pair: Pearson correlation across tissues is only 0.11, illustrating low correlation signal despite a true interaction.
- All ET pairs used must lie within the 500 KB distance threshold.

## Limitations

- Both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method — they are complements, not primary predictors.
- Correlation alone has low predictive value for enhancer-target interactions (Pearson r of only 0.11 for TTC39C-EH38E1904551).
- Requires precomputed Wilcoxon tests over many datasets (PrecomputedData.db) covering only the specific public datasets (CAGE-seq Andersson 2014, DHS Thurman 2012, DNase-matched expression, CRUP-EP); new activity data types require recomputation.
- Interaction search is restricted to ET pairs within 500 KB.

## Evidence

- [results] by performing a Wilcoxon rank-sum test between target transcriptional signals between cells where the enhancer is active and those where it is not active: "by performing a Wilcoxon rank-sum test between target transcriptional signals between cells where the enhancer is active and those where it is not active"
- [results] we summarize these four P-values into a single Fisher's combined probability: "we summarize these four P-values into a single Fisher's combined probability"
- [methods] We used the negative logarithm of the combined P-value as the final feature in our classification: "We used the negative logarithm of the combined P-value as the final feature in our classification"
- [results] both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method: "both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method"
- [methods] selected the top quantile (0.25) as the ones with the higher enhancer activity: "selected the top quantile (0.25) as the ones with the higher enhancer activity"
- [readme] PrecomputedData.db is a database containing precomputed Wilcoxon rank sum tests: [evidence span withheld — anchor status: external_doc]
