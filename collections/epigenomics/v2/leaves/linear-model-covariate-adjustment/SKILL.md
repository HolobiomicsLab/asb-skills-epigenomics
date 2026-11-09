---
name: linear-model-covariate-adjustment
description: Use when evaluating how one prediction parameter (e.g. minimal TPX length
  8/10/12/16) affects accuracy (AUC) across multiple conditions, and confounders like
  peak filtering method or masking covariate must be controlled.
when_to_use_negative:
- When the design varies only one factor with no confounding covariates — a simple
  paired comparison suffices and the nested-model machinery is unnecessary.
- When the outcome is not a continuous performance metric such as AUC but raw peak
  counts or sequence-level predictions — the lm formula and nesting structure would
  need to be redefined.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: pROC
  role: R package (v1.17.0.1) used to compute the ROC curves and AUC values that feed
    the linear models
- name: R lm / anova
  role: Base R functions used to fit the full and nested linear models and obtain
    Panoval for the minimal length term
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:8a5be51fa5516f10f85872bcec66ca4d60516915db850a940013a90577b67532
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - R lm / anova
  - Triplexator
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# linear-model-covariate-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit a full linear model on AUC values that includes the parameter of interest (minimal TPX length) alongside covariates (peak filtering method, ssRNA, n peaks), then compare it to a nested model without that term using R's anova to isolate the parameter's contribution while controlling for confounders.

## When to use

Use when evaluating how one prediction parameter (e.g. minimal TPX length 8/10/12/16) affects accuracy (AUC) across multiple conditions, and confounders like peak filtering method or masking covariate must be controlled.

## When NOT to use

- When the design varies only one factor with no confounding covariates — a simple paired comparison suffices and the nested-model machinery is unnecessary.
- When the outcome is not a continuous performance metric such as AUC but raw peak counts or sequence-level predictions — the lm formula and nesting structure would need to be redefined.

## Inputs

- Table of AUC values per parameter combination (peak filtering method x ssRNA x minimal length), from pROC ROC analysis
- Covariate columns: peak filtering method (IDR conservative, IDR optimal, Overlap conservative filtered at SPP q<0.05, Top1000, author-provided), ssRNA masking extent, number of peaks, minimal TPX length (8/10/12/16)

## Outputs

- Panoval: p-value from ANOVA comparison of full vs nested lm models
- Relevance probability of the parameter from a 10000-permutation randomisation test
- Model coefficients quantifying the effect of minimal length on AUC

## How to apply

Compute ROC curves and AUC values for each parameter combination (e.g. with the pROC R package v1.17.0.1) before modelling. Fit the full model with R's lm using the formula AUC ~ peak filtering method + ssRNA + n peaks + minimal length, and a nested model omitting the minimal length term, so ssRNA masking is carried as a covariate. Compare full vs nested models with R's anova function to obtain Panoval for the parameter of interest. To strengthen the inference, run a randomisation test with 10000 permutations of the AUC-versus-parameter association, refitting full and nested models on each permuted dataset while preserving the correlation structure, and count how many permutations yield Panova_random <= Panoval to estimate the relevance probability of the parameter.

## Related tools

- **pROC** (R package (v1.17.0.1) used to compute the ROC curves and AUC values that feed the linear models)
- **R lm / anova** (Base R functions used to fit the full and nested linear models and obtain Panoval for the minimal length term)

## Examples

```
full <- lm(AUC ~ method + ssRNA + n_peaks + min_length, data=aucs); nested <- lm(AUC ~ method + ssRNA + n_peaks, data=aucs); anova(nested, full)
```

## Evaluation signals

- The reproduced ANOVA comparison of full vs nested models yields Panoval < 2.2e-16 for the minimal length term
- The permutation-based probability of the minimal length parameter is estimated below 1e-4 (0 of 10000 permutations give Panova_random <= Panoval)
- Shorter minimal lengths (8 and 10) show the best AUC performance, with the default 16 the worst (reducing average AUC by ~0.1) and 12 intermediate
- Nested model differs from the full model only by the minimal length term; ssRNA is retained in both models as covariate

## Limitations

- Only 2 out of 7 lncRNAs had a defined set of IDR conservative peaks, so some peak filtering levels have unbalanced representation
- A minimal TPX length of 6 was tested in a subsample of the parameter space and gave no further improvement
- The permutation test must preserve the correlation structure of the data when refitting models, otherwise the relevance probability is biased

## Evidence

- [methods] Compute ROC curves and AUC values for each parameter combination with the pROC R package (v1.17.0.1).: [evidence span withheld — anchor status: not_found]
- [methods] Fit a full linear model with the R lm function (formula: AUC ~ peak filtering method + ssRNA + n peaks + minimal length): [evidence span withheld — anchor status: not_found]
- [methods] Run a randomisation test with 10000 permutations of the AUC-versus-parameter association: [evidence span withheld — anchor status: not_found]
- [results] Minimal TPX length significantly affects prediction accuracy (p-value<1e-04, permutation test): [evidence span withheld — anchor status: not_found]
