---
name: sex-dimorphism-aging-group-comparison
description: Use when flow-cytometry or compositional immune-cell data must be split
  by sex and age groups (e.g., 65-year cutoff) and non-parametric group differences
  tested against reported statistics.
when_to_use_negative:
- The input is single-cell or sorted-cell data — the paper notes future work is needed
  to describe these sex differences at single-cell resolution and in sorted cells,
  so this bulk PBMC-composition skill does not apply
- The goal is testing monocyte (CD14+/CD16+) differences between older men and women
  — the article reports no significant differences were detected, so verification
  of such comparisons will not reproduce a reported finding
- The analysis requires parametric modeling or covariate adjustment rather than non-parametric
  two-group rank-sum testing
edam_operation: http://edamontology.org/operation_3492
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_2228
tools:
- name: pandas
  role: Load the Fig. 1d Source Data spreadsheet and subset donor records by sex and
    age category
- name: scipy
  role: Run two-sided Wilcoxon rank-sum tests (scipy.stats.mannwhitneyu) for each
    sex/age group comparison
- name: R Shiny
  role: Interactive application (https://immune-aging.jax.org/) sharing the searchable
    immune-aging results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:6bd7e6c149eacc101f3c9c4d277283d15f095f8b570a6586ab032470a77e448e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3492
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_2228
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sex-dimorphism-aging-group-comparison

## Summary

Reproduce sex-stratified aging group comparisons on flow-cytometry PBMC cell-type proportions (67 women, 62 men) using two-sided Wilcoxon rank-sum tests, verifying reported p-values such as the male-specific CD19+ B cell decline after age 65 (p=1.6e-05).

## When to use

Use when flow-cytometry or compositional immune-cell data must be split by sex and age groups (e.g., 65-year cutoff) and non-parametric group differences tested against reported statistics.

## When NOT to use

- The input is single-cell or sorted-cell data — the paper notes future work is needed to describe these sex differences at single-cell resolution and in sorted cells, so this bulk PBMC-composition skill does not apply
- The goal is testing monocyte (CD14+/CD16+) differences between older men and women — the article reports no significant differences were detected, so verification of such comparisons will not reproduce a reported finding
- The analysis requires parametric modeling or covariate adjustment rather than non-parametric two-group rank-sum testing

## Inputs

- Source Data spreadsheet for Fig. 1d: flow-cytometry cell-type proportions per donor (67 women, 62 men), with sex and age metadata
- Age cutoff (65 years) and sex/group labels defining the comparison subsets

## Outputs

- wilcoxon_fig1d_verification.csv: table of recomputed two-sided Wilcoxon rank-sum p-values per comparison
- Recomputed p-value table matched against reported values (8.2e-07, 0.0069, 1.6e-05, 0.0032, 0.00057)

## How to apply

Load the Source Data spreadsheet for Fig. 1d containing flow-cytometry cell-type proportions for 67 women and 62 men with pandas. Subset by sex and by the age categories used in the figure, applying the age-65 cutoff for the B-cell comparison. Run two-sided Wilcoxon rank-sum tests via scipy.stats.mannwhitneyu for each reported comparison: CD8+ T cell decline with age in men (p=8.2e-07) and women (p=0.0069), the male-specific CD19+ B cell decline after 65 (p=1.6e-05), the older men-vs-women B cell difference (p=0.0032), and the young-cohort naive CD4+ sex difference (~13.2% vs ~7.8%, p=0.00057). Assemble recomputed p-values into a table and compare against reported values, saving the verification output as wilcoxon_fig1d_verification.csv. The paper's code for flow-cytometry analyses lives in the FlowData folder of the UcarLab repository.

## Related tools

- **pandas** (Load the Fig. 1d Source Data spreadsheet and subset donor records by sex and age category)
- **scipy** (Run two-sided Wilcoxon rank-sum tests (scipy.stats.mannwhitneyu) for each sex/age group comparison)
- **R Shiny** (Interactive application (https://immune-aging.jax.org/) sharing the searchable immune-aging results)

## Examples

```
from scipy.stats import mannwhitneyu; p = mannwhitneyu(bcell_men_older, bcell_women_older, alternative='two-sided')
```

## Evaluation signals

- Recomputed p-value for the male-specific CD19+ B cell decline after age 65 matches the reported 1.6e-05 (Wilcoxon)
- Recomputed p-value for the older men-vs-women B cell proportion difference matches the reported 0.0032
- Recomputed CD8+ T cell age-decline p-values match reported values (p=8.2e-07 men, p=0.0069 women)
- Young-cohort naive CD4+ sex difference reproduces group means ~13.2% vs ~7.8% with p=0.00057
- Verification table is saved as wilcoxon_fig1d_verification.csv with one row per reported comparison

## Limitations

- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Not all sex-biased differences in older adults are attributable to cell composition; monocyte cell numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes that this compositional test cannot resolve
- Sex-specific aging signatures may not be conserved across ethnicities/populations, which requires further investigation

## Evidence

- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry.: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [abstract] genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity.: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [intro] we profiled PBMCs of healthy adults by carefully matching the ages of male and female donors: "we profiled PBMCs of healthy adults by carefully matching the ages of male and female donors"
- [discussion] we did not detect significant differences between older men and older women in CD14+ and CD16+ monocyte cell numbers/frequencies neither in this cohort nor in a separate bigger cohort12: "we did not detect significant differences between older men and older women in CD14+ and CD16+ monocyte cell numbers/frequencies neither in this cohort nor in a separate bigger cohort12"
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
