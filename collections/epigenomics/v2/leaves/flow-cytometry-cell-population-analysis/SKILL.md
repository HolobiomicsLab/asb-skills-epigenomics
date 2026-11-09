---
name: flow-cytometry-cell-population-analysis
description: Use when per-donor flow-cytometry population proportion data are paired
  with donor age and sex metadata and the goal is to test which cell-type proportions
  change with age and differ between men and women (e.g., around an age-65 cutoff).
when_to_use_negative:
- Input is raw FCS flow-cytometry events rather than summarized per-donor cell-type
  proportions — this skill starts from gated population proportions
- The question is epigenomic (ATAC-seq) or transcriptomic (RNA-seq) aging signatures
  rather than flow-cytometry composition differences
- Donor metadata lacks sex or age, since both Wilcoxon stratification and the age/sex/interaction
  GLM terms require them
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_2269
tools:
- name: pandas
  role: Load and subset the Fig. 1d Source Data spreadsheet by sex and age category
    before testing
- name: scipy
  role: Run two-sided Wilcoxon rank-sum tests via scipy.stats.mannwhitneyu for each
    reported comparison
- name: R Shiny
  role: R environment used to fit per-population GLMs with age, sex, and age-by-sex
    interaction terms and expose results via the immune-aging Shiny app
  repo: https://immune-aging.jax.org/
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:2b718c49a63a04cebf0600ed2a478d0a8fb09b1f65f8c8f342be757732de3f43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# flow-cytometry-cell-population-analysis

## Summary

Statistical analysis of PBMC flow-cytometry cell-type proportions (CD8+ T, naive CD4+ T, CD19+ B cells, monocytes) across 172 healthy adults aged 22–93 to detect age- and sex-dependent composition changes, including Wilcoxon rank-sum comparisons and GLMs with age-by-sex interaction terms.

## When to use

Use when per-donor flow-cytometry population proportion data are paired with donor age and sex metadata and the goal is to test which cell-type proportions change with age and differ between men and women (e.g., around an age-65 cutoff).

## When NOT to use

- Input is raw FCS flow-cytometry events rather than summarized per-donor cell-type proportions — this skill starts from gated population proportions
- The question is epigenomic (ATAC-seq) or transcriptomic (RNA-seq) aging signatures rather than flow-cytometry composition differences
- Donor metadata lacks sex or age, since both Wilcoxon stratification and the age/sex/interaction GLM terms require them

## Inputs

- Source Data spreadsheet for Fig. 1d: per-donor flow-cytometry cell-type proportions for 67 women and 62 men
- Donor metadata: age (continuous and categorical, age-65 cutoff) and sex
- Per-donor flow cytometry population proportion data underlying Fig. 1d (loaded via R for GLM)

## Outputs

- wilcoxon_fig1d_verification.csv: recomputed p-values compared against reported values
- flow_glm_coefficients.csv: per-population GLM coefficients, standard errors, and p-values
- Ranked per-population association table (age vs sex vs age-by-sex interaction)

## How to apply

Load the Source Data spreadsheet underlying Fig. 1d (cell-type proportions for 67 women and 62 men) with pandas, subset by sex and the figure's age categories, and run two-sided Wilcoxon rank-sum tests (scipy.stats.mannwhitneyu) for each reported comparison: CD8+ T cell decline with age in men (p=8.2e-07) and women (p=0.0069), the male-specific CD19+ B cell decline after age 65 (p=1.6e-05), the older men-vs-women B cell difference (p=0.0032), and the young-cohort naive CD4+ sex difference (~13.2% vs ~7.8%, p=0.00057). Complement this by fitting, in R, a per-population GLM with age as a continuous covariate for major populations (age-group factor for subsets), sex, and an age-by-sex interaction term; extract coefficients, standard errors, and p-values and rank coefficient magnitudes per population. The rationale is that age is expected to be most strongly associated with CD8+/naive T cell proportions, sex with naive CD4+ T cell proportions, and the age-by-sex interaction with B cell proportions. Write results as verification and coefficient CSV tables (e.g., wilcoxon_fig1d_verification.csv, flow_glm_coefficients.csv).

## Related tools

- **pandas** (Load and subset the Fig. 1d Source Data spreadsheet by sex and age category before testing)
- **scipy** (Run two-sided Wilcoxon rank-sum tests via scipy.stats.mannwhitneyu for each reported comparison)
- **R Shiny** (R environment used to fit per-population GLMs with age, sex, and age-by-sex interaction terms and expose results via the immune-aging Shiny app) — https://immune-aging.jax.org/

## Examples

```
from scipy.stats import mannwhitneyu; u, p = mannwhitneyu(b_men_older, b_women_older, alternative='two-sided')
```

## Evaluation signals

- Recomputed Wilcoxon p-values match reported values: p=8.2e-07 (men CD8+ decline), p=0.0069 (women CD8+ decline), p=1.6e-05 (male B cell decline after 65), p=0.0032 (older men-vs-women B cells), p=0.00057 (young naive CD4+ sex difference)
- Young-cohort naive CD4+ T cell proportions reproduce the ~13.2% (women) vs ~7.8% (men) sex difference
- GLM coefficient ranking shows age strongest for CD8+/naive T cells, sex strongest for naive CD4+ T cells, and a significant age-by-sex effect on B cell proportions
- flow_glm_coefficients.csv contains coefficients, standard errors, and p-values for every immune cell population modeled
- Verification table saved as wilcoxon_fig1d_verification.csv with matching comparison labels

## Limitations

- Monocyte cell numbers/frequencies did not differ between older men and older women in this cohort or a separate bigger cohort, implying cell-intrinsic changes not captured by composition analysis
- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Sex differences in immune aging have not been described at single-cell resolution or in sorted cells, and functional implications remain unestablished
- Conservation of sex-specific aging signatures across ethnicities/populations requires further investigation

## Evidence

- [results] Wilcoxon p = 1.6e-05 male-specific B cell decline after 65: [evidence span withheld — anchor status: not_found]
- [results] GLM confirmed age most strongly associated with CD8+ T cell proportions: [evidence span withheld — anchor status: not_found]
- [abstract] 172 healthy adults 22-93 characterized by flow cytometry: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] Shared searchable R Shiny application: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [readme] Repository includes FlowData code for flow cytometry analyses: "1. FlowData: Code used for flow cytometry data analyses"
