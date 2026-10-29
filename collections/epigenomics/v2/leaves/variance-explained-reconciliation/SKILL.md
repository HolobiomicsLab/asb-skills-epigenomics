---
name: variance-explained-reconciliation
description: 'Use when reproducing or auditing a published PCA claim: per-individual
  PC1 scores, donor age and sex are available, and the variance explained by PC1 and
  group differences (e.g., older men vs older women) must be verified.'
when_to_use_negative:
- Input is raw sequencing reads or per-peak/accession matrices rather than per-individual
  PC1 scores from an existing PCA — PCA must be recomputed first.
- The published claim does not involve variance explained or PC score group comparisons
  (e.g., differential peak calling or flow cytometry gating questions).
- Donor age and sex metadata are absent, since the age correlation and 65+ male-vs-female
  Wilcoxon test cannot be performed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R Shiny
  role: Searchable web application (https://immune-aging.jax.org/) sharing the aging
    results that the PC1/variance-explained reconciliation checks against
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
- name: pandas
  role: Loads the Fig. 1b Source Data table with per-individual PC1 scores, donor
    age, and sex
- name: scipy.stats
  role: Performs the two-sided Wilcoxon rank-sum (Mann-Whitney U) test comparing PC1
    scores of older men vs older women
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:c9e269aef2640db78a08a102525af513b5d8c5f2dfedb01ba50723ee83bb5aac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - pandas
  - scipy.stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# variance-explained-reconciliation

## Summary

Reconciles the fraction of variance captured by PC1 in PCA of PBMC ATAC-seq and RNA-seq datasets against published values (~15% ATAC-seq, ~18.6% RNA-seq in Márquez et al. 2020) and validates the age/sex structure of PC1 scores statistically. It ensures reported dimensionality and downstream PC1-based claims are reproducible from the Source Data tables.

## When to use

Use when reproducing or auditing a published PCA claim: per-individual PC1 scores, donor age and sex are available, and the variance explained by PC1 and group differences (e.g., older men vs older women) must be verified.

## When NOT to use

- Input is raw sequencing reads or per-peak/accession matrices rather than per-individual PC1 scores from an existing PCA — PCA must be recomputed first.
- The published claim does not involve variance explained or PC score group comparisons (e.g., differential peak calling or flow cytometry gating questions).
- Donor age and sex metadata are absent, since the age correlation and 65+ male-vs-female Wilcoxon test cannot be performed.

## Inputs

- Source Data table (CSV/Excel) with per-individual PC1 scores from ATAC-seq and RNA-seq PCA
- Per-donor metadata: age (22-93 years) and sex for 172 healthy adults
- Published reference values: PC1 variance explained ~15% (ATAC-seq), ~18.6% (RNA-seq)

## Outputs

- Verification table (fig1b_pc1_verification.csv) with correlation coefficients, variance-explained values, Wilcoxon statistics and p-values, and pass/fail verdicts
- Summary of age-PC1 correlation and older-subject sex-difference test results per assay

## How to apply

Load the figure Source Data table (e.g., Fig. 1b) containing per-individual PC1 scores from the ATAC-seq and RNA-seq PCA together with donor age and sex, using pandas. Compute Pearson and Spearman correlations between PC1 score and age separately for each assay to confirm PC1 increases with age in both ATAC-seq and RNA-seq. Compare the observed proportion of variance explained by PC1 against the published values of ~15% (ATAC-seq) and ~18.6% (RNA-seq). Subset donors aged 65 and older and run a two-sided Wilcoxon rank-sum (Mann-Whitney U) test with scipy.stats on PC1 scores of older men versus older women, expecting men to show significantly higher PC1. Record correlation coefficients, variance-explained values, Wilcoxon statistics, p-values, and pass/fail verdicts for each check in a verification CSV (e.g., fig1b_pc1_verification.csv).

## Related tools

- **R Shiny** (Searchable web application (https://immune-aging.jax.org/) sharing the aging results that the PC1/variance-explained reconciliation checks against) — github.com/UcarLab/SexDimorphismNatureCommunications
- **pandas** (Loads the Fig. 1b Source Data table with per-individual PC1 scores, donor age, and sex)
- **scipy.stats** (Performs the two-sided Wilcoxon rank-sum (Mann-Whitney U) test comparing PC1 scores of older men vs older women)

## Examples

```
from scipy.stats import mannwhitneyu; u,p = mannwhitneyu(df[(df.age>=65)&(df.sex=='M')].PC1, df[(df.age>=65)&(df.sex=='F')].PC1, alternative='two-sided')
```

## Evaluation signals

- Observed PC1 variance explained matches published values: ~15% for ATAC-seq and ~18.6% for RNA-seq.
- Pearson/Spearman correlation between PC1 score and age is positive and significant in both ATAC-seq and RNA-seq PCA.
- Two-sided Mann-Whitney U test on PC1 scores of donors aged 65+ yields significantly higher scores in older men than older women.
- Verification CSV contains one row per check with correlation coefficient, variance-explained value, Wilcoxon statistic, p-value, and pass/fail verdict.
- Number of donors loaded matches the published cohort of 172 healthy adults aged 22-93.

## Limitations

- Bulk PBMC profiling means the PC1 aging signal aggregates cell-composition and cell-intrinsic effects; the article notes the annotation approach is 'prone to biases in references, e.g., differences in data quality and limitation to use cell types available in references.'
- Not all sex-biased differences in older adults are attributable to cell composition: CD14+ and CD16+ monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes the PC1 reconciliation cannot resolve.
- Findings are from 172 healthy adults aged 22-93; conservation of the sex-specific aging signatures across ethnicities/populations 'requires further investigation.'

## Evidence

- [abstract] Cohort scope for PC1 analyses: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [abstract] Aging signature underlying PC1 direction: "These data reveal a shared epigenomic signature of aging including declining naïve T cell and increasing monocyte and cytotoxic cell functions."
- [abstract] Post-65 sex divergence tested by Wilcoxon step: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [readme] Repository provenance for the analysis code: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
- [readme] Figure-level code location for reproduction: "Figures: Code used for figures related to genomic data analyses."
