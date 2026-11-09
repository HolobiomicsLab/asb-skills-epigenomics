---
name: wilcoxon-rank-sum-hypothesis-testing
description: Use when comparing a continuous measurement (e.g. cell-type proportions)
  between two independent groups — such as men vs women, or young vs older donors
  with a 65-year cutoff — and no normality assumption is warranted.
when_to_use_negative:
- Data are paired or repeated-measures (same donor before/after) — Wilcoxon rank-sum
  is for independent samples; use a signed-rank or paired test instead
- The comparison involves more than two groups or requires effect-size/interaction
  modeling (e.g. sex × age), rather than a two-sample proportion test
- Input is single-cell count matrices or sequencing data rather than precomputed cell-type
  proportion tables
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: pandas
  role: Load the Fig. 1d Source Data spreadsheet and subset donors by sex and age
    category (65-year cutoff) before testing
- name: scipy
  role: scipy.stats.mannwhitneyu performs the two-sided Wilcoxon rank-sum tests for
    each reported comparison
- name: R Shiny
  role: Interactive exploration of the underlying results via the searchable immune-aging
    Shiny app
  repo: https://immune-aging.jax.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:020c303b4611d9ff04ae6659851ef42bfed3869e672be0f157645aa53f1d568b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# wilcoxon-rank-sum-hypothesis-testing

## Summary

Non-parametric two-sided Wilcoxon rank-sum (Mann–Whitney U) testing to verify reported cell-composition differences, e.g. the male-specific CD19+ B cell decline after age 65 and CD8+ T cell declines with age, from flow-cytometry proportions in the sexual-dimorphism immune aging study.

## When to use

Use when comparing a continuous measurement (e.g. cell-type proportions) between two independent groups — such as men vs women, or young vs older donors with a 65-year cutoff — and no normality assumption is warranted.

## When NOT to use

- Data are paired or repeated-measures (same donor before/after) — Wilcoxon rank-sum is for independent samples; use a signed-rank or paired test instead
- The comparison involves more than two groups or requires effect-size/interaction modeling (e.g. sex × age), rather than a two-sample proportion test
- Input is single-cell count matrices or sequencing data rather than precomputed cell-type proportion tables

## Inputs

- Source Data spreadsheet for Fig. 1d (flow cytometry cell-type proportions per donor, with sex and age metadata; 67 women, 62 men)
- Group definitions: sex (male/female) and age category (young vs older, 65-year cutoff)

## Outputs

- wilcoxon_fig1d_verification.csv — table of recomputed two-sided Wilcoxon p-values per comparison
- Per-comparison p-values matched against reported values (8.2e-07, 0.0069, 1.6e-05, 0.0032, 0.00057)

## How to apply

Load the Source Data spreadsheet for Fig. 1d containing flow cytometry cell-type proportions for 67 women and 62 men into pandas. Subset by sex and the age categories used in the figure (young vs older donors, with age 65 as the cutoff for the B-cell comparison). Run two-sided Wilcoxon rank-sum tests with scipy.stats.mannwhitneyu for each reported comparison: CD8+ T cell decline with age in men and in women, the male-specific CD19+ B cell decline after 65, the older men-vs-women B cell difference, and the young-cohort naive CD4+ sex difference (~13.2% vs ~7.8%). Assemble recomputed p-values into a table (wilcoxon_fig1d_verification.csv) and compare against the reported values (p=8.2e-07 men / p=0.0069 women for CD8+ T decline; p=1.6e-05 male B cell decline; p=0.0032 older men-vs-women B cells; p=0.00057 naive CD4+ young sex difference). Rank-sum tests are appropriate because cell-type proportions are non-normal, bounded distributions and group sizes are modest (62–67 donors).

## Related tools

- **pandas** (Load the Fig. 1d Source Data spreadsheet and subset donors by sex and age category (65-year cutoff) before testing)
- **scipy** (scipy.stats.mannwhitneyu performs the two-sided Wilcoxon rank-sum tests for each reported comparison)
- **R Shiny** (Interactive exploration of the underlying results via the searchable immune-aging Shiny app) — https://immune-aging.jax.org/

## Examples

```
from scipy.stats import mannwhitneyu; p = mannwhitneyu(b_men_older, b_women_older, alternative='two-sided')
```

## Evaluation signals

- Recomputed p-value for the male-specific CD19+ B cell decline after 65 reproduces the reported p = 1.6e-05
- Recomputed p-value for older men-vs-women B cell proportions matches p = 0.0032
- CD8+ T cell decline with age reproduces p = 8.2e-07 (men) and p = 0.0069 (women)
- Young-cohort naive CD4+ sex difference reproduces p = 0.00057 with reported means ~13.2% vs ~7.8%
- Output CSV has one row per reported comparison with matching column names and p-value magnitudes within rounding of published values

## Limitations

- Results depend on using the exact age cutoff (65) and donor subsets (67 women, 62 men) as in the figure; deviating from these changes p-values
- Rank-sum tests detect group-level composition differences but cannot establish cell-intrinsic changes — the paper notes monocyte cell numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes requiring further study
- Bulk PBMC-level analysis is prone to reference/annotation biases and is limited to available cell types; single-cell or sorted-cell resolution would be needed for functional interpretation
- The test is non-parametric and assumption-light but does not by itself adjust for multiple testing across the several reported comparisons

## Evidence

- [abstract] characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [abstract] genomic differences between sexes increase after age 65: "genomic differences between sexes increase after age 65, with men having higher innate and pro-inflammatory activity and lower adaptive activity."
- [readme] FlowData: Code used for flow cytometry data analyses: "1. FlowData: Code used for flow cytometry data analyses"
- [other] workflow: load Source Data and run Wilcoxon rank-sum tests: [evidence span withheld — anchor status: not_found]
- [abstract] male-specific B-cell specific loci decline: "These changes are greater in magnitude in men and accompanied by a male-specific decline in B-cell specific loci."
