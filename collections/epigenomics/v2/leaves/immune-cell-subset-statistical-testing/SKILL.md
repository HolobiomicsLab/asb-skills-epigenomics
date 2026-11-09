---
name: immune-cell-subset-statistical-testing
description: Use when you have per-donor cell population proportions (flow cytometry)
  plus donor metadata (continuous age, sex) and need to test which factor—age, sex,
  or their interaction—is most strongly associated with each cell subset.
when_to_use_negative:
- The input is already a fitted model object or a coefficient table rather than per-donor
  proportion data with donor metadata.
- The question is about bulk transcriptomic or ATAC-seq signal changes rather than
  cytometry-derived cell composition proportions.
- The cohort lacks both male and female donors across a sufficient age range, making
  the age-by-sex interaction term unestimable.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R Shiny
  role: Language/platform used for the analysis scripts and for sharing results via
    the searchable immune-aging Shiny application; flow cytometry GLM code is provided
    in the repository's FlowData folder
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:bb878386d98eb30edccc2cfcedd2bdc236dcc7c9126414dd80169d09045b6b7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# immune-cell-subset-statistical-testing

## Summary

Fits per-population generalized linear models (GLMs) of flow-cytometry-derived PBMC cell proportions against age, sex, and an age-by-sex interaction term, then ranks coefficient magnitudes to identify which donor factor most strongly shapes each immune cell subset. It operationalizes the Márquez et al. (Nature Communications, 2020) finding that age dominates CD8+/naive T cell variation while sex dominates naive CD4+ T cell variation.

## When to use

Use when you have per-donor cell population proportions (flow cytometry) plus donor metadata (continuous age, sex) and need to test which factor—age, sex, or their interaction—is most strongly associated with each cell subset.

## When NOT to use

- The input is already a fitted model object or a coefficient table rather than per-donor proportion data with donor metadata.
- The question is about bulk transcriptomic or ATAC-seq signal changes rather than cytometry-derived cell composition proportions.
- The cohort lacks both male and female donors across a sufficient age range, making the age-by-sex interaction term unestimable.

## Inputs

- Per-donor flow cytometry cell population proportion table (PBMC subsets, underlying Fig. 1d)
- Donor metadata: age (continuous years, cohort 22–93), sex

## Outputs

- flow_glm_coefficients.csv: per-population table of GLM coefficients, standard errors, and p-values for age, sex, and age-by-sex terms
- Ranked list of strongest associated factor (age, sex, or interaction) per immune cell population

## How to apply

Load the per-donor flow cytometry cell population proportion data together with donor metadata (age, sex) underlying Fig. 1d into R. For each immune cell population, fit a GLM with age as a continuous covariate for major populations (and as an age-group factor for subset-level analyses), sex, and an age-by-sex interaction term. Extract model coefficients, standard errors, and p-values into a single coefficient table written to flow_glm_coefficients.csv. Rank coefficient magnitudes per population to determine which term is strongest for each subset; the model design rationale is that a continuous age term captures lifespan trends over the 22–93 year cohort, while the interaction term captures sex-dimorphic aging effects. Code for the flow cytometry analyses lives in the FlowData folder of the UcarLab/SexDimorphismNatureCommunications repository.

## Related tools

- **R Shiny** (Language/platform used for the analysis scripts and for sharing results via the searchable immune-aging Shiny application; flow cytometry GLM code is provided in the repository's FlowData folder) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
fit <- glm(CD8_naive_prop ~ age + sex + age:sex, data = flow_props, family = quasibinomial); coefs <- summary(fit)$coefficients
```

## Evaluation signals

- The coefficient table flow_glm_coefficients.csv contains coefficient, standard error, and p-value columns for age, sex, and age-by-sex terms for every modeled cell population.
- Ranking coefficient magnitudes per population reproduces the reported pattern: age is strongest for CD8+ T cell proportions (especially naive cells), sex is strongest for naive CD4+ T cells, and the age-by-sex interaction is significant for B cell proportions.
- Age is treated as a continuous covariate for major populations and as an age-group factor for subset analyses, matching the specified model specification.
- Each model uses one population at a time as the response, so coefficient rows match the number of flow cytometry populations analyzed.

## Limitations

- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references.
- Not all sex-biased differences in older adults are attributable to changes in cell composition; monocyte cell numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes that a proportion-based GLM cannot capture.
- Findings are derived from a single cohort of 172 healthy adults aged 22–93; conservation of sex-specific aging signatures across ethnicities/populations requires further investigation.

## Evidence

- [abstract] cohort and platforms: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] sharing platform: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [readme] repo code location: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
- [readme] flow analysis code folder: "1. FlowData: Code used for flow cytometry data analyses"
