---
name: cnv-expression-linear-regression-adjustment
description: Use when gene-expression matrices will be correlated with drug response
  or other phenotypes, and CNV-driven expression variation must be removed first to
  avoid spurious associations.
when_to_use_negative:
- No copy-number data are available alongside the expression data, so the CNV regression
  cannot be fit
- The goal is CNV discovery or copy-number calling itself, not expression adjustment
- Reliable target information or ChIP-seq data are absent — the article notes RePhine
  is restricted to cases where reliable target information is available and that lack
  of ChIP-seq data would mislead TR identification
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: RePhine
  role: R package whose Step 1 performs the CNV-vs-expression linear regression adjustment
    on pharmacogenomic data
  repo: https://github.com/coexps/RePhine
- name: R
  role: Implementation environment; linear models fit per gene, with RePhine R code
    (drug_code_R.R) and prerequisite R packages
  repo: https://github.com/coexps/RePhine
- name: ppcor
  role: R package used in the downstream step to compute partial correlations of CNV-adjusted
    expression with drug response controlling for mutation confounders
- name: glmnet
  role: R package used downstream to fit the adaptive lasso model that selects drug-response
    mutation confounders
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:78e5323ebd0d5615c35bc6c506730f99d76e2d43c4431d84978612656919a361
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - RePhine
  - glmnet
  - ppcor
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cnv-expression-linear-regression-adjustment

## Summary

Adjusts per-gene mRNA expression for copy-number-driven variation by regressing expression on copy number in a linear model and taking residuals, as done in RePhine Step 1 before computing gene-vs-drug-response partial correlations on CCLE data.

## When to use

Use when gene-expression matrices will be correlated with drug response or other phenotypes, and CNV-driven expression variation must be removed first to avoid spurious associations.

## When NOT to use

- No copy-number data are available alongside the expression data, so the CNV regression cannot be fit
- The goal is CNV discovery or copy-number calling itself, not expression adjustment
- Reliable target information or ChIP-seq data are absent — the article notes RePhine is restricted to cases where reliable target information is available and that lack of ChIP-seq data would mislead TR identification

## Inputs

- CCLE DepMap gene-expression matrix
- CCLE copy-number segment data (per-gene copy number)
- Optional: CCLE mutation calls and PLX4720 drug pharmacology (drug sensitivity) profiles for downstream confounder identification

## Outputs

- CNV-adjusted (residual) expression matrix, one residual per gene per cell line
- Per-gene partial-correlation coefficient table (e.g., PLX4720_partial_correlations.csv) produced downstream

## How to apply

Load the CCLE DepMap expression matrix and copy-number segments and, for each gene, fit a linear model regressing expression on its copy number, keeping only the residuals as the CNV-adjusted expression (RePhine Step 1). Rationale: as the article notes, CNVs, data noises, and confounding mutations pose challenges for associating transcriptional regulators with drug response, so CNV effects must be removed before downstream correlation analysis. Implement this in R (RePhine is an R package; prerequisites include glmnet, MASS, lmtest, ppcor, psych, ROCR). Use the residualized expression, not raw expression, in the subsequent partial-correlation step, where RePhine further controls for mutation confounders identified by an adaptive lasso model with likelihood ratio testing (ppcor, glmnet). Decision point: only CNV adjustment happens here — mutation-confounder adjustment is a later RePhine step, so do not conflate the two.

## Related tools

- **RePhine** (R package whose Step 1 performs the CNV-vs-expression linear regression adjustment on pharmacogenomic data) — https://github.com/coexps/RePhine
- **R** (Implementation environment; linear models fit per gene, with RePhine R code (drug_code_R.R) and prerequisite R packages) — https://github.com/coexps/RePhine
- **ppcor** (R package used in the downstream step to compute partial correlations of CNV-adjusted expression with drug response controlling for mutation confounders)
- **glmnet** (R package used downstream to fit the adaptive lasso model that selects drug-response mutation confounders)

## Examples

```
Rscript -e 'fit <- lm(expr ~ cnv, data=gene_df); adjusted_expr <- residuals(fit)'  # per gene, as in RePhine's drug_code_R.R
```

## Evaluation signals

- Residualized expression shows no (or markedly reduced) linear association with copy number per gene — the regression residuals should be uncorrelated with the CNV predictor
- Per-gene residuals are produced for every gene in the expression matrix with no missing-gene failures
- Downstream partial correlations computed on residualized expression (ppcor) differ from raw Pearson correlations, indicating the adjustment took effect
- RePhine outperformed Pearson correlation analysis, logistic regression model, and gene set enrichment analysis on simulated data with noises/confounders — a benchmark comparison for the full adjusted pipeline
- Outputs such as PLX4720_partial_correlations.csv and PLX4720_confounder_genes.txt are written to disk as specified in the workflow

## Limitations

- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE
- Pre-treatment associations (e.g., FOS) need further evaluation when post-treatment data become available
- CNV adjustment removes only copy-number-driven variation; confounding mutations of kinase genes or tumor suppressor genes are handled separately in later RePhine steps

## Evidence

- [readme] readme step 1: "Estimate the adjusted expression for each gene by removing the effects of copy number through linear regression."
- [readme] RePhine prerequisites: [evidence span withheld — anchor status: external_doc]
- [intro] challenges motivating adjustment: "the effects of copy number variations (CNVs), data noises, and confounding mutations of some kinase genes or tumor suppressor genes, as well as how to accurately infer targets, pose challenges in"
- [discussion] confounder adjustment rationale: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
- [abstract] performance benchmark: "RePhine demonstrated an improved performance in comparison with three commonly used methods (including Pearson correlation analysis, logistic regression model, and gene set enrichment analysis)"
