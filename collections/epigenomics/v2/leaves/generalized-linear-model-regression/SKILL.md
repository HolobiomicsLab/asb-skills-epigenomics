---
name: generalized-linear-model-regression
description: Use when you have per-donor flow cytometry cell population proportion
  data plus donor metadata (continuous age, sex) and need to quantify which factor
  — age, sex, or their interaction — is most strongly associated with each PBMC cell
  subset.
when_to_use_negative:
- Input is already aggregated summary statistics rather than per-donor cell proportions
  — GLM ranking of factor associations requires donor-level variation
- The question is about molecular or single-cell aging signatures (ATAC-seq/RNA-seq
  loci, sorted cells), not bulk flow cytometry composition of PBMCs
- Testing within-donor longitudinal change — this skill models cross-sectional between-donor
  variation (age-matched male and female donors), not repeated measures
edam_operation: http://edamontology.org/operation_2990
edam_topics: []
tools:
- name: R Shiny
  role: Interactive application (https://immune-aging.jax.org/) through which the
    per-donor flow cytometry GLM results are shared and searchable
  repo: github.com/UcarLab/SexDimorphismNatureCommunications
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14396-9
    title: Sexual-dimorphism in human immune system aging
  source_content_hash: sha256:617a498a4db1c90b727d38a81df11bd37f0dd404492c36fe35a9664f6d73f46a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R Shiny
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# generalized-linear-model-regression

## Summary

Fits per-cell-population generalized linear models of flow cytometry PBMC cell proportions against age (continuous or age-group factor), sex, and an age-by-sex interaction to rank which individual-level factor most strongly drives each immune cell composition change in the 172-donor Márquez et al. cohort.

## When to use

Use when you have per-donor flow cytometry cell population proportion data plus donor metadata (continuous age, sex) and need to quantify which factor — age, sex, or their interaction — is most strongly associated with each PBMC cell subset.

## When NOT to use

- Input is already aggregated summary statistics rather than per-donor cell proportions — GLM ranking of factor associations requires donor-level variation
- The question is about molecular or single-cell aging signatures (ATAC-seq/RNA-seq loci, sorted cells), not bulk flow cytometry composition of PBMCs
- Testing within-donor longitudinal change — this skill models cross-sectional between-donor variation (age-matched male and female donors), not repeated measures

## Inputs

- Per-donor flow cytometry cell population proportion table (PBMC fractions underlying Fig. 1d)
- Donor metadata: age as continuous variable (22-93 years) and age-group factors, sex
- List of immune cell population labels (e.g., CD8+ T cells, naive CD4+ T cells, B cells, CD14+/CD16+ monocytes)

## Outputs

- flow_glm_coefficients.csv — coefficient table with model coefficients, standard errors, and p-values per cell population and term (age, sex, age:sex)
- Per-population ranking of coefficient magnitudes identifying the strongest individual-level factor

## How to apply

Load the per-donor flow cytometry cell population proportion data and donor metadata (age, sex) underlying Fig. 1d of the sexual-dimorphism in human immune system aging study in R. For each immune cell population, fit a GLM with age as a continuous covariate for major populations (switching to an age-group factor for subset analyses), sex, and an age-by-sex interaction term. Extract model coefficients, standard errors, and p-values into a single coefficient table written to flow_glm_coefficients.csv. Rank coefficient magnitudes per population and check that the ranking reproduces the reported pattern: age strongest for CD8+ T cell proportions (especially naive cells), sex strongest for naive CD4+ T cells, and a significant age-by-sex effect on B cell proportions.

## Related tools

- **R Shiny** (Interactive application (https://immune-aging.jax.org/) through which the per-donor flow cytometry GLM results are shared and searchable) — github.com/UcarLab/SexDimorphismNatureCommunications

## Examples

```
fit <- glm(proportion ~ age * sex, data = pbmc_props); write.csv(coef(summary(fit)), "flow_glm_coefficients.csv")
```

## Evaluation signals

- flow_glm_coefficients.csv exists and contains columns for coefficients, standard errors, and p-values across every modeled cell population
- Ranking of coefficient magnitudes reproduces the reported validation: age strongest for CD8+ T cell proportions (especially naive cells)
- Sex emerges as the strongest term for naive CD4+ T cell proportions in the ranking
- The age-by-sex interaction term is significant for B cell proportions
- Major populations use age as a continuous covariate while subset analyses use an age-group factor, matching the paper's modeling scheme

## Limitations

- Bulk PBMC annotation approach is prone to biases in reference datasets, including differences in data quality and limited cell types available in references
- Not all sex-biased differences in older adults are attributable to cell composition: CD14+ and CD16+ monocyte numbers/frequencies did not differ between older men and women, implying cell-intrinsic changes not captured by the GLM
- Cross-sectional cohort design (172 healthy adults aged 22-93, age-matched by sex) cannot establish within-individual longitudinal trajectories

## Evidence

- [results] flow GLM design and validation: [evidence span withheld — anchor status: not_found]
- [abstract] cohort and data modalities: "we characterize peripheral blood mononuclear cells from 172 healthy adults 22–93 years of age using ATAC-seq, RNA-seq, and flow cytometry."
- [intro] R Shiny dissemination of results: "These results are shared via a searchable R Shiny application (https://immune-aging.jax.org/)."
- [readme] repository organization for flow analyses: "This is a repository that include the code used for the analyses of PBMC ATAC/RNA/flow cytometry data."
- [discussion] monocyte cell-intrinsic limitation: "we did not detect significant differences between older men and older women in CD14+ and CD16+ monocyte cell numbers/frequencies neither in this cohort"
