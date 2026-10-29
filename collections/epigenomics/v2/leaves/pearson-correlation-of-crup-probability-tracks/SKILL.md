---
name: pearson-correlation-of-crup-probability-tracks
description: Reference for computing or benchmarking the cor_CRUP generic feature
  (Pearson correlation between summed CRUP-EP and CRUP-PP probability tracks across
  cell types) when building or ablation-testing ET-interaction feature vectors.
when_to_use_negative:
- As the primary or sole predictor of enhancer-target interactions — the article shows
  correlation alone has low predictive value and is inferior to the genomic-distance
  baseline on GM12878
- For a cell type not covered by the 104 precomputed ENCODE ChIP-seq profiles (CRUP-EP/CRUP-PP
  probability tracks would need to be generated first via crupR)
- For interactions beyond the 500 KB window, which is outside the CENTRE ET-pair search
  space
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CRUP-EP
  role: Provides the per-100-bp-bin enhancer probabilities that are summed over the
    5-bin enhancer region before correlation
- name: CRUP-PP
  role: Provides the per-100-bp-bin promoter probabilities that are summed over the
    5-bin target region before correlation
- name: CENTRE
  role: R/Bioconductor framework whose computeGenericFeatures() step computes the
    cor_CRUP feature for ET pairs
  repo: https://github.com/slrvv/CENTRE
- name: XGBoost
  role: Pre-trained gradient boosting classifier used to evaluate the standalone predictive
    value of the feature via AUPRC
- name: R
  role: Implementation language of the CENTRE package and its feature computation
    functions
  repo: https://github.com/slrvv/CENTRE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad687
    title: 'CENTRE: a gradient boosting algorithm for Cell-type-specific ENhancer-Target
      pREdiction'
  source_content_hash: sha256:6fb7313fa19fc51b90c7acf0376a3bfad7eea8fce9de2c3c33ffa2ff81828a65
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Pearson correlation of CRUP probability tracks

## Summary

A cell-type-agnostic generic feature for enhancer-target (ET) pair classification in which CRUP-EP enhancer probabilities (summed over the 5-bin enhancer region) and CRUP-PP promoter probabilities (summed over the 5-bin target region) are correlated with Pearson's coefficient across the 104 ENCODE cell types. In CENTRE this feature (cor_CRUP) was evaluated as a standalone predictor and shown to have low predictive value, ranking ninth and below the genomic-distance baseline.

## When to use

Reference for computing or benchmarking the cor_CRUP generic feature (Pearson correlation between summed CRUP-EP and CRUP-PP probability tracks across cell types) when building or ablation-testing ET-interaction feature vectors.

## When NOT to use

- As the primary or sole predictor of enhancer-target interactions — the article shows correlation alone has low predictive value and is inferior to the genomic-distance baseline on GM12878
- For a cell type not covered by the 104 precomputed ENCODE ChIP-seq profiles (CRUP-EP/CRUP-PP probability tracks would need to be generated first via crupR)
- For interactions beyond the 500 KB window, which is outside the CENTRE ET-pair search space

## Inputs

- Precomputed CRUP-EP enhancer probability tables for 104 ENCODE cell types (In_house_constructed_datasets.zip)
- Precomputed CRUP-PP promoter probability tables for 104 ENCODE cell types
- All-Pairs.Natural-Ratio BENGI ET pairs (lifted to hg38 and overlapped with ENCODE cCRE-ELS), e.g. GM12878 RNAPII-ChIAPET
- 5-bin enhancer and target region definitions at 100-bp resolution from CRUP output

## Outputs

- Scalar cor_CRUP feature (Pearson correlation coefficient) per ET pair
- AUPRC of the feature under 12-fold chromosome-split XGBoost cross-validation for benchmarking

## How to apply

For each ET pair, sum the CRUP-EP enhancer probabilities over the 5-bin (100-bp each) enhancer region and the CRUP-PP promoter probabilities over the 5-bin target region, for each of the 104 ENCODE cell types with precomputed probabilities (In_house_constructed_datasets.zip). Compute the Pearson correlation coefficient between the 104 enhancer-side sums and the 104 target-side sums to obtain a single scalar, cor_CRUP. Evaluate the feature in isolation with the XGBoost classifier under 12-fold chromosome-split cross-validation on the All-Pairs.Natural-Ratio BENGI GM12878 dataset, recording AUPRC against the genomic-distance baseline. Expect it to underperform: the feature ranks ninth (after distance at position 1 and CT-specific features at positions 2-5), and illustrative pairs such as TTC39C-EH38E1904551 yield a Pearson correlation of only 0.11 despite being true interactions. Use the Wilcoxon rank-sum/Fisher combined feature (which ranks sixth) as the comparison generic feature when interpreting the result.

## Related tools

- **CRUP-EP** (Provides the per-100-bp-bin enhancer probabilities that are summed over the 5-bin enhancer region before correlation)
- **CRUP-PP** (Provides the per-100-bp-bin promoter probabilities that are summed over the 5-bin target region before correlation)
- **CENTRE** (R/Bioconductor framework whose computeGenericFeatures() step computes the cor_CRUP feature for ET pairs) — https://github.com/slrvv/CENTRE
- **XGBoost** (Pre-trained gradient boosting classifier used to evaluate the standalone predictive value of the feature via AUPRC)
- **R** (Implementation language of the CENTRE package and its feature computation functions) — https://github.com/slrvv/CENTRE

## Examples

```
pairs <- computeGenericFeatures(pairs)  # CENTRE R package: computes cor_CRUP and other cell-type-agnostic features
```

## Evaluation signals

- The correlation is computed over exactly 104 cell-type values for each ET pair (one summed CRUP-EP value and one summed CRUP-PP value per cell type)
- Standalone XGBoost AUPRC under the 12-fold chromosome-split CV on GM12878 should be below the genomic-distance baseline and below the Wilcoxon/Fisher combined_tests feature
- Feature-importance ranking should place CRUP correlation at position 9, with distance at 1 and CT-specific features (RNA-seq, normalized regulatory distance) at positions 2-5
- Sanity check against the article's example: true pair TTC39C-EH38E1904551 yields a Pearson correlation of only 0.11
- Each probability track sum must cover exactly 5 consecutive 100-bp CRUP bins for the enhancer and the target respectively

## Limitations

- Both generic features (Fisher's combined probability and Pearson correlation) perform poorly on distinguishing true and false ET interactions on GM12878, being inferior to the baseline distance method
- Low correlation does not imply no interaction — the TTC39C-EH38E1904551 pair interacts in GM12878 despite a correlation of only 0.11
- Coverage is limited to the 104 ENCODE cell types with precomputed CRUP-EP/CRUP-PP probabilities; note CRUP-EP RNA-seq comparisons in the paper also used 66 matched ENCODE cell types
- Applicable only to ET pairs within the 500 KB distance threshold used by CENTRE

## Evidence

- [methods] CRUP-EP outputs the enhancer probability for every 100-bp genomic bin: "we applied the CRUP-EP algorithm. CRUP-EP outputs the enhancer probability for every 100-bp genomic bin"
- [results] Generic features perform poorly on GM12878 relative to distance baseline: "both generic features perform poorly on distinguishing true and false ET interactions on the GM12878 cell line, being inferior to the baseline distance method"
- [results] Feature ranking places distance first, CRUP correlation ninth: "Genomic distance (position 1), Wilcoxon tests combined (position 6), and CRUP correlation (position 9) are generic features, while positions 2 (RNA-seq), 3 (Normalized Regulatory distance enhancers),"
- [results] TTC39C-EH38E1904551 example correlation of 0.11: "resulting in a Pearson Correlation Coefficient of only 0.11"
- [readme] CENTRE computes generic (non-cell-type-specific) features via computeGenericFeatures(pairs): "The function `computeGenericFeatures(pairs)` computes the features that are not cell type specific"
