---
name: permutation-based-anova-significance-testing
description: Use when you need to establish whether a tuning parameter (e.g. minimal
  TPX length 8/10/12/16) significantly influences AUC performance, by comparing a
  full linear model against a nested model without that term via ANOVA and a permutation
  test.
when_to_use_negative:
- You have no AUC/performance table and only raw triplex prediction output (e.g. 3plex
  *_tpx.stability.gz files) — compute ROC curves first with pROC before modelling
- The question is comparing whole prediction tools (e.g. 3plex vs TriplexFPP agreement)
  rather than testing significance of a single parameter within one tool's scoring
  model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: 3plex
  role: Generates the TPX predictions across minimal-length settings (8/10/12/16)
    whose AUC values feed the ANOVA/permutation models; also scoring baseline via
    Triplexator potential score
  repo: https://github.com/molinerisLab/3plex
- name: pROC
  role: R package (v1.17.0.1) computing the ROC curves and AUC values that are the
    response variable of the linear model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:cde953d16470f775087c658509249e57909b40f53a97d6fee162238afcbc677d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - Triplexator
  - bedtools
  - pROC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# permutation-based-anova-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical framework from the 3plex paper to test whether a scoring parameter (e.g. minimal TPX length 8/10/12/16) significantly affects prediction accuracy (AUC), combining an R lm-based ANOVA comparison of a full vs nested model with a 10000-permutation randomisation test to estimate parameter relevance probability.

## When to use

Use when you need to establish whether a tuning parameter (e.g. minimal TPX length 8/10/12/16) significantly influences AUC performance, by comparing a full linear model against a nested model without that term via ANOVA and a permutation test.

## When NOT to use

- You have no AUC/performance table and only raw triplex prediction output (e.g. 3plex *_tpx.stability.gz files) — compute ROC curves first with pROC before modelling
- The question is comparing whole prediction tools (e.g. 3plex vs TriplexFPP agreement) rather than testing significance of a single parameter within one tool's scoring model

## Inputs

- AUC values per parameter combination (peak filtering method x minimal TPX length 8/10/12/16), computed with pROC v1.17.0.1 from ROC curves of TPX predictions
- Design table with covariates: peak filtering method, ssRNA, n peaks, minimal length
- Optional: bedtools-shuffled negative regions matched to positive peak length distributions (for generating the AUC values)

## Outputs

- Panoval: p-value from R anova() comparison of full vs nested linear model
- Relevance probability of the minimal length parameter: fraction of 10000 permutations with Panoval_random <= Panoval
- Estimated effect of minimal length on AUC (e.g. default 16 reducing average AUC by 0.1 vs lengths 8/10)

## How to apply

Fit a full linear model in R with lm using the formula AUC ~ peak filtering method + ssRNA + n peaks + minimal length, plus a nested model without the minimal length term (controlling for ssRNA as covariate), and compare them with the R anova function to obtain Panoval. Then run a randomisation test with 10000 permutations of the AUC-versus-parameter association: on each permuted dataset, refit both full and nested models while preserving the correlation structure, and count how many permutations yield Panoval_random <= Panoval to estimate the relevance probability of the parameter. AUC inputs come from ROC curves computed with the pROC R package (v1.17.0.1) for each combination of peak filtering method (IDR conservative, IDR optimal, Overlap conservative filtered at SPP q<0.05, Top1000, author-provided peaks) and 3plex/Triplexator minimal TPX length, with other parameters fixed at the Matveishina et al. suggested values (minimal length=10 baseline, error rate=20%, guanine rate=40%). Accept the analysis when the ANOVA comparison yields Panoval < 2.2e-16, the permutation-based probability of the minimal length parameter is below 1e-4, and the shorter lengths (8 and 10) show the best AUC performance.

## Related tools

- **3plex** (Generates the TPX predictions across minimal-length settings (8/10/12/16) whose AUC values feed the ANOVA/permutation models; also scoring baseline via Triplexator potential score) — https://github.com/molinerisLab/3plex
- **pROC** (R package (v1.17.0.1) computing the ROC curves and AUC values that are the response variable of the linear model)

## Examples

```
Rscript -e 'full<-lm(AUC~method+ssRNA+npeaks+minlen,d); nested<-lm(AUC~method+ssRNA+npeaks,d); print(anova(full,nested))'
```

## Evaluation signals

- Reproduced ANOVA comparison of full vs nested models yields Panoval < 2.2e-16 for the minimal length term
- Permutation-based relevance probability of the minimal length parameter is estimated below 1e-4 (i.e. ~0 of 10000 permutations with Panoval_random <= Panoval)
- Shorter minimal lengths (8 and 10) show the best AUC performance, 12 gives intermediate performance, and the default 16 is the worst with about 0.1 average AUC reduction
- Permuted refits preserve the correlation structure of the original AUC-versus-parameter association
- Sub-sensitivity check: testing minimal length 6 on a parameter-space subsample yields no further improvement over length 8

## Limitations

- The permutation p-value has a 1/10000 resolution floor, so relevance probabilities below 1e-4 can only be reported as 'below 1e-4'
- Minimal TPX length 6 was only evaluated on a subsample of the parameter space, so conclusions about very short triplexes are limited
- The linear model controls for peak filtering method, ssRNA, and n peaks; AUC estimates depend on the chosen peak filtering method (IDR conservative was best, +0.09 average AUC) and negative-region shuffling strategy

## Evidence

- [methods] Fit a full linear model with the R lm function (formula: AUC ~ peak filtering method + ssRNA + n peaks + minimal length): [evidence span withheld — anchor status: not_found]
- [methods] Permutation test with 10000 permutations: [evidence span withheld — anchor status: not_found]
- [readme] 3plex minimal length parameter: [evidence span withheld — anchor status: external_doc]
