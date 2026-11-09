---
name: coefficient-table-interpretation
description: Use when you have per-donor flow cytometry cell proportion data plus
  donor metadata (age, sex) and need to determine which individual-level factor is
  most strongly associated with each immune cell population's composition.
when_to_use_negative:
- Data are bulk genomic (ATAC-seq/RNA-seq) loci rather than per-donor flow cytometry
  proportions — use differential/trajectory/breakpoint analyses instead
- The analysis goal is differential expression or enrichment rather than attributing
  cell-composition variation to individual-level covariates
- Donor metadata lacks both age and sex, so the age × sex interaction design cannot
  be fitted
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R Shiny
  role: R is used to fit the GLMs and the resulting results are shared via the searchable
    R Shiny application; the repo contains the flow cytometry analysis code under
    the FlowData folder
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:0052a1f8547cc77451375f40ccd03c951a5be013b7f563805859dc39d1dffd08
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# coefficient-table-interpretation

## Summary

Fits per-cell-population generalized linear models of PBMC flow cytometry proportions against age, sex, and age-by-sex interaction, then extracts coefficients, standard errors, and p-values into a single table (flow_glm_coefficients.csv) that is ranked to attribute composition variation to age vs. sex.

## When to use

Use when you have per-donor flow cytometry cell proportion data plus donor metadata (age, sex) and need to determine which individual-level factor is most strongly associated with each immune cell population's composition.

## When NOT to use

- Data are bulk genomic (ATAC-seq/RNA-seq) loci rather than per-donor flow cytometry proportions — use differential/trajectory/breakpoint analyses instead
- The analysis goal is differential expression or enrichment rather than attributing cell-composition variation to individual-level covariates
- Donor metadata lacks both age and sex, so the age × sex interaction design cannot be fitted

## Inputs

- Per-donor flow cytometry cell population proportion data (as underlying Fig. 1d)
- Donor metadata: age (continuous and/or age-group factor) and sex for 172 healthy adults aged 22–93

## Outputs

- flow_glm_coefficients.csv: coefficient table with model coefficients, standard errors, and p-values per immune cell population and predictor (age, sex, age-by-sex interaction)
- Per-population ranking of coefficient magnitudes identifying the strongest associated factor

## How to apply

Load the per-donor flow cytometry cell population proportion data underlying Fig. 1d together with donor metadata (age, sex) in R. For each immune cell population, fit a GLM with age as a continuous covariate for major populations (and as an age-group factor for subset analyses), sex, and an age-by-sex interaction term. Extract model coefficients, standard errors, and p-values into a single coefficient table written to flow_glm_coefficients.csv. Rank coefficient magnitudes per population to attribute each population's variation to age, sex, or the interaction. The rationale is to separate age-driven from sex-driven remodeling of immune composition in the 172 healthy adults aged 22–93 characterized by ATAC-seq, RNA-seq, and flow cytometry.

## Related tools

- **R Shiny** (R is used to fit the GLMs and the resulting results are shared via the searchable R Shiny application; the repo contains the flow cytometry analysis code under the FlowData folder) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
fit <- glm(proportion ~ age * sex, data = flow_props); write.csv(summary(fit)$coefficients, "flow_glm_coefficients.csv")
```

## Evaluation signals

- Coefficient table reproduces the reported associations: age strongest for CD8+ and naive T cell proportions
- Coefficient table shows sex strongest for naive CD4+ T cell proportions
- Coefficient table shows a significant age-by-sex effect on B cell proportions
- flow_glm_coefficients.csv exists and contains coefficients, standard errors, and p-values for every modeled immune cell population
- Coefficient-magnitude rankings per population are consistent with the paper's conclusion that age drives CD8+/naive T cell variation and sex drives naive CD4+ T cell variation

## Limitations

- Age is treated as a continuous covariate only for major populations; subset analyses require an age-group factor, so coefficient tables are not directly comparable across population levels
- The bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Not all sex-biased differences in older adults are attributable to cell composition; e.g., CD14+ and CD16+ monocyte frequencies did not differ between older men and women, implying cell-intrinsic changes not captured by proportion GLMs
- The GitHub README gives no reproducible entry-point script or parameter file; code is only organized by folder (FlowData, Scripts, Figures, Shiny)

## Evidence

- [results] GLM analysis of PBMC compositions confirmed that age is most strongly associated with CD8+ T cell proportions, especially for naive cells, whereas sex is most strongly associated with naive CD4+ T cells.: [evidence span withheld — anchor status: not_found]
- [abstract] we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry.: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/).: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [readme] This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data.: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
