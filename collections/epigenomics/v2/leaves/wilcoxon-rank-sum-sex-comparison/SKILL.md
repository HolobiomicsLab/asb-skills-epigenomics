---
name: wilcoxon-rank-sum-sex-comparison
description: Use when comparing a continuous score (e.g., PCA PC1) between two small,
  non-normally distributed donor groups — specifically older men vs older women from
  an age-stratified PBMC cohort — without assuming normality.
when_to_use_negative:
- The goal is to compare whole genome-wide expression/accessibility distributions
  rather than a single per-donor summary score such as PC1
- Donor groups are large and clearly normally distributed, where a t-test or linear
  model with age/sex covariates is more appropriate
- The analysis targets age-associated change over the full 22–93 lifespan (trajectory/breakpoint
  analysis) rather than the post-65 sex contrast
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3170
tools:
- name: R Shiny
  role: Searchable web application hosting the cohort's PCA and sex-comparison results
    for interactive exploration of PC1 scores by age and sex
  repo: https://immune-aging.jax.org/
- name: pandas / scipy.stats
  role: Load the Source Data table, subset donors aged 65+, and run the two-sided
    Mann-Whitney U test on PC1 scores
  repo: https://github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:e7e550a8af3969d7de386980b4157c404cbfb3dee80277589352e8383db72ea7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - pandas / scipy.stats
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# wilcoxon-rank-sum-sex-comparison

## Summary

Non-parametric two-sided Wilcoxon rank-sum (Mann-Whitney U) test to compare PCA-derived PC1 scores between older men (65+) and older women in PBMC ATAC-seq/RNA-seq data, testing whether sex differences in immune aging emerge after age 65.

## When to use

Use when comparing a continuous score (e.g., PCA PC1) between two small, non-normally distributed donor groups — specifically older men vs older women from an age-stratified PBMC cohort — without assuming normality.

## When NOT to use

- The goal is to compare whole genome-wide expression/accessibility distributions rather than a single per-donor summary score such as PC1
- Donor groups are large and clearly normally distributed, where a t-test or linear model with age/sex covariates is more appropriate
- The analysis targets age-associated change over the full 22–93 lifespan (trajectory/breakpoint analysis) rather than the post-65 sex contrast

## Inputs

- Source Data table (e.g., Fig. 1b) with per-individual PC1 scores from ATAC-seq and RNA-seq PCA, plus donor age and sex
- Donor metadata from 172 healthy adults aged 22–93 (PBMC ATAC-seq, RNA-seq, flow cytometry cohort)
- Age threshold of 65 years for defining the 'older' subgroup

## Outputs

- Wilcoxon rank-sum statistic and two-sided p-value for older men vs older women PC1 scores, per assay (ATAC-seq, RNA-seq)
- Verification CSV (e.g., fig1b_pc1_verification.csv) with correlation coefficients, variance-explained values, Wilcoxon statistics, p-values, and pass/fail verdicts

## How to apply

Load the Source Data table (e.g., Fig. 1b per-individual PC1 scores from ATAC-seq and RNA-seq PCA, with donor age and sex) using pandas. First confirm PC1 captures age-driven variation by computing Pearson/Spearman correlation of PC1 vs age and checking variance explained (~15% ATAC-seq, ~18.6% RNA-seq). Then subset donors aged 65 and older and run a two-sided Wilcoxon rank-sum test (scipy.stats.mannwhitneyu) on PC1 scores of older men versus older women; the expectation from the article is that men show significantly higher PC1, consistent with greater age-related epigenomic changes in men and higher innate/pro-inflammatory activity after 65. Record the statistic, p-value, and a pass/fail verdict per assay in a CSV (e.g., fig1b_pc1_verification.csv). Use a rank-sum test rather than a t-test because donor counts per subgroup are modest and score distributions need not be normal.

## Related tools

- **R Shiny** (Searchable web application hosting the cohort's PCA and sex-comparison results for interactive exploration of PC1 scores by age and sex) — https://immune-aging.jax.org/
- **pandas / scipy.stats** (Load the Source Data table, subset donors aged 65+, and run the two-sided Mann-Whitney U test on PC1 scores) — https://github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(pc1_men_65plus, pc1_women_65plus, alternative='two-sided')
```

## Evaluation signals

- PC1 vs age Pearson/Spearman correlation is positive in both ATAC-seq and RNA-seq PCA (PC1 increases with age)
- Variance explained by PC1 matches published values of ~15% (ATAC-seq) and ~18.6% (RNA-seq)
- Wilcoxon rank-sum on PC1 of donors aged 65+ yields a significant two-sided p-value with men scoring higher than women
- Output CSV (fig1b_pc1_verification.csv) contains per-check statistics, p-values, and pass/fail verdicts for both assays
- Only donors with age >= 65 are included in the sex-contrast subset; younger donors are excluded before testing

## Limitations

- Sex differences after age 65 are not fully attributable to cell composition: CD14+ and CD16+ monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes not resolved by this test
- The bulk PBMC approach is prone to biases in reference datasets (data-quality differences, limited cell types), which can confound PC1 interpretation
- Findings derive from a single cohort of healthy adults 22–93; conservation across ethnicities/populations requires further investigation
- The test is associative — it cannot establish the hormonal or DAMP/PAMP/transposable-element mechanisms behind inflammaging differences

## Evidence

- [results] PC1 captured 15% of variation in ATAC-seq and 18.6% in RNA-seq data, associated with age groups, with older men showing higher PC1 scores than older women: [evidence span withheld — anchor status: not_found]
- [methods] Subset donors aged 65 and older, run a two-sided Wilcoxon rank-sum (Mann-Whitney U) test on PC1 scores: [evidence span withheld — anchor status: not_found]
- [abstract] genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/).: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
