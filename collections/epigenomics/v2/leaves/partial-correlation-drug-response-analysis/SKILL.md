---
name: partial-correlation-drug-response-analysis
description: Use when estimating gene-expression vs drug-response associations from
  pharmacogenomic data (CCLE-style expression, copy number, mutation, and pharmacology
  profiles) where CNV effects and confounding mutations (kinase/tumor-suppressor genes)
  must be adjusted before correlation.
when_to_use_negative:
- No mutation or copy-number data are available for the cohort, so confounders cannot
  be identified or CNV effects regressed out — plain correlation would be mislabeled
  as confounder-adjusted.
- The goal is downstream transcriptional-regulator selection (uniP/multiP/GSEA) rather
  than gene-level drug-response association estimation — use the later RePhine steps
  instead.
- Reliable target/ChIP-seq information is absent — the article notes RePhine overall
  'is restricted to the cases where reliable target information is available' (relevant
  when extending this step to TR identification).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: RePhine
  role: R package implementing the full pipeline; Step 1 performs CNV adjustment,
    adaptive-lasso confounder selection, and partial-correlation computation
  repo: https://github.com/coexps/RePhine
- name: R (ppcor)
  role: Computes the partial correlation of each gene's adjusted expression with drug
    response controlling for selected mutation confounders
  repo: https://github.com/coexps/RePhine
- name: R (glmnet)
  role: Fits the adaptive lasso model of mutation features against drug response to
    nominate confounding mutated genes
  repo: https://github.com/coexps/RePhine
- name: Python
  role: Supporting scripts (e.g., RP score calculation); RePhine distribution provides
    modified Python scripts and R code
  repo: https://github.com/coexps/RePhine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.09.008
    title: '<i>RePhine</i>: An Integrative Method for Identification of Drug Response-Related
      Transcriptional Regulators'
  source_content_hash: sha256:ab5a1247048aebd9babe2c947ca8e518c9cc5ff32074bf9dea7eb05000f9c718
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/coexps/RePhine
  tools:
  - Python
  - R
  - R (glmnet)
  - R (ppcor)
  - RePhine
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# partial-correlation-drug-response-analysis

## Summary

RePhine's Step 1 module that estimates noise- and confounder-robust gene–drug associations: expression is first adjusted for copy number by linear-regression residuals, drug-response mutation confounders are selected with an adaptive lasso plus likelihood ratio test, and per-gene partial correlations (computed with the R ppcor package) quantify CNV-adjusted expression vs drug response (e.g., PLX4720 in CCLE).

## When to use

Use when estimating gene-expression vs drug-response associations from pharmacogenomic data (CCLE-style expression, copy number, mutation, and pharmacology profiles) where CNV effects and confounding mutations (kinase/tumor-suppressor genes) must be adjusted before correlation.

## When NOT to use

- No mutation or copy-number data are available for the cohort, so confounders cannot be identified or CNV effects regressed out — plain correlation would be mislabeled as confounder-adjusted.
- The goal is downstream transcriptional-regulator selection (uniP/multiP/GSEA) rather than gene-level drug-response association estimation — use the later RePhine steps instead.
- Reliable target/ChIP-seq information is absent — the article notes RePhine overall 'is restricted to the cases where reliable target information is available' (relevant when extending this step to TR identification).

## Inputs

- CCLE DepMap gene-expression matrix
- CCLE copy-number segments
- CCLE mutation calls (mutation status per cell line)
- Drug pharmacology / pharmacological drug-sensitivity profile (e.g., PLX4720 response)
- Cancer-type information per cell line
- TCGA differentially expressed gene sets (for cancer-relevance down-weighting via limma)

## Outputs

- Per-gene partial-correlation coefficient table (e.g., PLX4720_partial_correlations.csv)
- Selected mutation-confounder gene list (e.g., PLX4720_confounder_genes.txt)
- CNV-adjusted (residual) expression matrix per gene

## How to apply

Load CCLE DepMap expression, copy-number segments, mutation calls, and the drug pharmacology (drug sensitivity) file into R. For each gene, regress expression on its copy number with a linear model and retain the residuals, removing CNV-driven expression variation. Fit an adaptive lasso (R glmnet) of mutation features on drug response (e.g., PLX4720 sensitivity) and apply likelihood-ratio tests to select significant drug-response mutation confounders. Using the R ppcor package, compute the partial correlation of each gene's CNV-adjusted expression with drug response while controlling for the selected mutation confounders, iterating gene by gene. Down-weight cancer-type-specific/non-cancer-related genes by scoring them against TCGA differentially expressed gene sets identified with limma. Write the per-gene partial-correlation coefficient table (e.g., PLX4720_partial_correlations.csv) and the confounder gene list (e.g., PLX4720_confounder_genes.txt); these coefficients feed downstream RePhine steps (uniP/multiP TR selection and modified GSEA).

## Related tools

- **RePhine** (R package implementing the full pipeline; Step 1 performs CNV adjustment, adaptive-lasso confounder selection, and partial-correlation computation) — https://github.com/coexps/RePhine
- **R (ppcor)** (Computes the partial correlation of each gene's adjusted expression with drug response controlling for selected mutation confounders) — https://github.com/coexps/RePhine
- **R (glmnet)** (Fits the adaptive lasso model of mutation features against drug response to nominate confounding mutated genes) — https://github.com/coexps/RePhine
- **Python** (Supporting scripts (e.g., RP score calculation); RePhine distribution provides modified Python scripts and R code) — https://github.com/coexps/RePhine

## Examples

```
library(ppcor); cor <- pcor.test(expr_adj[,g], drug_response, confounder_mut, method="pearson")$estimate
```

## Evaluation signals

- Regressing raw expression on copy number yields residual expression with no significant expression–CNV association remaining per gene.
- Adaptive lasso plus likelihood-ratio test on mutation features against drug response returns a non-empty, statistically supported confounder set (e.g., BRAF-pathway mutations for PLX4720).
- Computed partial correlations (ppcor) differ from naive Pearson correlations, confirming confounder adjustment was applied, and fall in [-1, 1].
- Outputs match the expected schema: per-gene coefficient table (PLX4720_partial_correlations.csv) and confounder list (PLX4720_confounder_genes.txt) are written to disk.
- On simulated data with noise/confounders, the pipeline outperforms Pearson correlation, logistic regression, and GSEA, as reported for RePhine against three commonly used methods.

## Limitations

- Correlations detected through targets may not be causal; upstream regulators or kinases influencing drug response may remain unmodeled.
- Detection of acquired resistance is limited due to lack of post-treatment data in CCLE (e.g., secondary resistance to erlotinib via acquired STAT3 activation could not be detected).
- Only confounders captured in the mutation data and adaptive-lasso/likelihood-ratio-test selection are adjusted; other potential confounding factors may persist.

## Evidence

- [readme] Confounder identification via adaptive lasso and partial correlation with ppcor: "Identify mutated genes related to drug response based on the CCLE mutation data and pharmacology profiles through an adaptive lasso model and likelihood ratio test."
- [readme] Partial correlation computation controlling confounding factors: "Compute the partial correlation between a specific gene and a drug by controlling the confounding factors."
- [readme] CNV adjustment as prerequisite for partial correlation: "Estimate the adjusted expression for each gene by removing the effects of copy number through linear regression."
- [intro] Motivation: confounding mutations challenge association measurement: "the effects of copy number variations (CNVs), data noises, and confounding mutations of some kinase genes or tumor suppressor genes, as well as how to accurately infer targets, pose challenges in"
- [discussion] Design rationale: adjust confounders not under TR regulation: "an accurate measurement of correlation patterns by adjusting all potential CFs that are not under the impact of TR regulation"
