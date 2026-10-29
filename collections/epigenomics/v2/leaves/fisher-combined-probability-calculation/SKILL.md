---
name: fisher-combined-probability-calculation
description: Use when building enhancer-target interaction features from multi-cell-type
  activity data and you need a dataset-independent score contrasting gene expression
  between enhancer-active and enhancer-inactive cell types.
when_to_use_negative:
- The analysis goal is cell-type-specific prediction — this feature is deliberately
  cell-type agnostic and must be complemented by CT-specific features (RNA-seq, H3K27ac/H3K4me1/H3K4me3
  ChIP-seq)
- 'Expecting correlation-like behavior: the source work shows the Fisher-combined
  rank-sum statistic outperforms the average Pearson correlation (cor_CRUP, e.g. TTC39C-EH38E1904551
  r = 0.11) but both are inferior to the genomic-distance baseline on GM12878'
- Enhancer-target pairs extend beyond the 500 KB window used by CENTRE, since the
  precomputed tests and BENGI-derived pairs are restricted to that range
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CENTRE
  role: R package whose computeGenericFeatures() computes the Fisher's combined probability
    feature and packages the precomputed Wilcoxon rank-sum tests in PrecomputedData.db
  repo: https://github.com/slrvv/CENTRE
- name: metapod
  role: R dependency used within CENTRE's requirements for combining P-values
- name: XGBoost
  role: Pre-trained gradient boosting classifier used to evaluate the predictive value
    of the combined_tests feature under 12-fold chromosome-split cross-validation
- name: R
  role: Language in which the skill is implemented and distributed as the CENTRE Bioconductor
    package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:54a82ccfc6223e424243aa43aa512fb55a10c4e1ee84af30e7d1d923f7738532
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slrvv/CENTRE
  tools:
  - CENTRE
  - R
  - XGBoost
  - metapod
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fisher-combined-probability-calculation

## Summary

A cell-type-agnostic feature-construction skill from CENTRE: per enhancer-target pair, run Wilcoxon rank-sum tests of target gene signal across cell types with active vs inactive enhancers on four public activity datasets, combine the four P-values with Fisher's method, and use the negative logarithm of the combined P-value as a classification feature.

## When to use

Use when building enhancer-target interaction features from multi-cell-type activity data and you need a dataset-independent score contrasting gene expression between enhancer-active and enhancer-inactive cell types.

## When NOT to use

- The analysis goal is cell-type-specific prediction — this feature is deliberately cell-type agnostic and must be complemented by CT-specific features (RNA-seq, H3K27ac/H3K4me1/H3K4me3 ChIP-seq)
- Expecting correlation-like behavior: the source work shows the Fisher-combined rank-sum statistic outperforms the average Pearson correlation (cor_CRUP, e.g. TTC39C-EH38E1904551 r = 0.11) but both are inferior to the genomic-distance baseline on GM12878
- Enhancer-target pairs extend beyond the 500 KB window used by CENTRE, since the precomputed tests and BENGI-derived pairs are restricted to that range

## Inputs

- All-Pairs.Natural-Ratio BENGI ET pairs (lifted to hg38, overlapping ENCODE cCRE-ELS, within 500 KB)
- FANTOM5 CAGE enhancer activity across cell types (Andersson et al. 2014)
- DNase hypersensitivity counts across 112 cell types (Thurman et al. 2012) with top quantile 0.25 as active
- DNase-matched microarray gene expression (Sheffield et al. 2013)
- CRUP-EP enhancer probabilities across 66 matched ENCODE cell types
- Precomputed Wilcoxon rank-sum test tables (e.g. PrecomputedData.db from CENTREprecomputed/In_house_constructed_datasets.zip)

## Outputs

- combined_tests feature: negative log of Fisher's combined P-value per enhancer-target pair
- Per-dataset Wilcoxon rank-sum P-values for active vs inactive enhancer cell types
- Generic (cell-type-agnostic) feature matrix column consumed by the XGBoost classifier

## How to apply

For each ET pair, define enhancer-active versus enhancer-inactive cell types per dataset: for the FANTOM5 CAGE enhancer set, DNase-hypersensitive counts across the 112 cell types of Thurman et al. 2012 (top quantile 0.25 defined as high activity), DNase-matched microarray gene expression, and CRUP-EP probabilities across 66 matched ENCODE cell types. In each dataset, perform a Wilcoxon rank-sum test comparing the target gene's signal in cell types where the enhancer is active versus inactive. Summarize the four resulting P-values into a single Fisher's combined probability and take the negative logarithm to obtain the 'combined_tests' feature used by CENTRE's generic feature set. Evaluate the feature's predictive value independently (e.g. with XGBoost under 12-fold chromosome-split CV on the All-Pairs.Natural-Ratio BENGI GM12878 dataset) rather than assuming it beats simpler statistics: in the source work this feature ranked sixth, above the Pearson-correlation feature (ninth) but below the genomic-distance baseline (first).

## Related tools

- **CENTRE** (R package whose computeGenericFeatures() computes the Fisher's combined probability feature and packages the precomputed Wilcoxon rank-sum tests in PrecomputedData.db) — https://github.com/slrvv/CENTRE
- **metapod** (R dependency used within CENTRE's requirements for combining P-values)
- **XGBoost** (Pre-trained gradient boosting classifier used to evaluate the predictive value of the combined_tests feature under 12-fold chromosome-split cross-validation)
- **R** (Language in which the skill is implemented and distributed as the CENTRE Bioconductor package)

## Examples

```
features_generic <- computeGenericFeatures(pairs)  # combined_tests = -log(Fisher's combined P of 4 Wilcoxon rank-sum tests)
```

## Evaluation signals

- Each of the four datasets yields a Wilcoxon rank-sum P-value per ET pair; missing any one of CAGE, DNase counts, DNase-matched expression, or CRUP-EP invalidates the Fisher combination
- The final feature value equals -log(P) of the Fisher-combined probability and is non-negative
- Feature-importance ranking on GM12878 should place genomic distance first, the Wilcoxon combined feature sixth, and the CRUP correlation ninth — deviations indicate a pipeline error
- Both generic features should perform worse than the genomic-distance baseline AUPRC under the 12-fold chromosome-split CV on All-Pairs.Natural-Ratio BENGI GM12878
- When recomputing activity calls for DNase data, cell types in the top quantile (0.25) are labeled active; verify grouping sizes match this cutoff

## Limitations

- Both generic features (Fisher's combined rank-sum probability and Pearson correlation) perform poorly on distinguishing true and false ET interactions on GM12878, being inferior to the baseline distance method
- Performance depends strongly on the choice of validation data; F1 scores remain low on many datasets
- The feature set and precomputed tests are limited to ET pairs within 500 KB of each other

## Evidence

- [results] Wilcoxon active/inactive contrast: "by performing a Wilcoxon rank-sum test between target transcriptional signals between cells where the enhancer is active and those where it is not active"
- [methods] Negative log Fisher combination: "We used the negative logarithm of the combined P-value as the final feature in our classification"
- [methods] DNase activity quantile cutoff: "selected the top quantile (0.25) as the ones with the higher enhancer activity"
- [readme] Precomputed tests database: [evidence span withheld — anchor status: external_doc]
- [results] Generic feature ranking finding: "Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features"
