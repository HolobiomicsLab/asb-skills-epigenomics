---
name: gene-expression-matrix-processing
description: Use when a raw numeric expression matrix (rows = features/genes, columns
  = samples) contains missing values, outliers, or low-variance rows and must be prepared
  for feature selection and subgroup classification.
when_to_use_negative:
- Input is already a filtered, imputed feature table with no missing values and homogeneous
  row variance
- Sample classification is not the goal (e.g. the task is differential expression
  rather than consensus-partition subgroup discovery)
- Single-cell datasets with several thousands of cells where cola's runtime and memory
  usage become unacceptable
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: cola
  role: Provides adjust_matrix() for imputing, outlier adjustment and low-variance
    row removal, plus top-value methods (SD, MAD, CV, ATC) for row scoring
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: Supplies impute.knn() used by adjust_matrix() to fill in missing values
- name: golubEsets
  role: Bioconductor package supplying the example Golub leukemia expression matrix
    used to demonstrate preprocessing and ATC scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:1a12577c1f35e397fe845987aca9a4592bc58f7725b92d380e0af517a25c43e8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jokergoo/cola_manuscript
  repo_url: https://github.com/jokergoo/cola_manuscript
  tools:
  - R
  - cola
  - golubEsets
  - impute
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# gene-expression-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses a gene-expression (or other high-dimensional genomic) matrix in R so its rows are suitable for scoring and consensus partitioning with cola: impute missing values, adjust outliers, remove low-variance rows, then rank rows by a top-value method such as ATC.

## When to use

Use when a raw numeric expression matrix (rows = features/genes, columns = samples) contains missing values, outliers, or low-variance rows and must be prepared for feature selection and subgroup classification.

## When NOT to use

- Input is already a filtered, imputed feature table with no missing values and homogeneous row variance
- Sample classification is not the goal (e.g. the task is differential expression rather than consensus-partition subgroup discovery)
- Single-cell datasets with several thousands of cells where cola's runtime and memory usage become unacceptable

## Inputs

- Numeric gene-expression matrix (rows = features/genes, columns = samples), e.g. from the golubEsets Bioconductor package
- Optionally a methylation or single-cell RNA-Seq matrix (e.g. recount2 or HSMM data)

## Outputs

- Preprocessed matrix with imputed, winsorized and low-variance-filtered rows
- Two-column per-row score table (feature name, top-value/ATC score), e.g. atc_scores.csv
- Ordered list of top n features selected for consensus partitioning

## How to apply

Load the numeric matrix with rows as features and columns as samples (e.g. the Golub leukemia dataset from the golubEsets Bioconductor package). Pass it through cola's adjust_matrix(), which removes rows in which >25% of samples have missing values, imputes remaining missing values with impute.knn() from the R package impute, winsorizes values beyond the 5th/95th percentiles, and removes rows with zero variance or variance below the 5th percentile of all row variances. For feature scoring, cola's top-value methods (SD, MAD, CV, ATC) assign a score to each row and the top n rows with the highest scores are selected for consensus partitioning; the ATC score for row i is 1 - ∫₀¹ F_X(x) dx over the empirical CDF of its absolute correlations to all other rows, yielding scores in [0, 1] that increase with global correlation strength. Assemble per-row scores into a two-column table (feature name, score) such as atc_scores.csv for downstream validation.

## Related tools

- **cola** (Provides adjust_matrix() for imputing, outlier adjustment and low-variance row removal, plus top-value methods (SD, MAD, CV, ATC) for row scoring) — https://github.com/jokergoo/cola_manuscript
- **impute** (Supplies impute.knn() used by adjust_matrix() to fill in missing values)
- **golubEsets** (Bioconductor package supplying the example Golub leukemia expression matrix used to demonstrate preprocessing and ATC scoring)

## Examples

```
library(cola); m = adjust_matrix(exprs(Golub_Merge)); atc = ATC(m); write.csv(data.frame(feature = names(atc), score = atc), "atc_scores.csv", row.names = FALSE)
```

## Evaluation signals

- atc_scores.csv exists and contains exactly one score per input row
- Every ATC value lies within [0, 1]
- Rows with >25% missing samples, zero variance, or variance below the 5th percentile of all row variances are absent from the preprocessed matrix
- Remaining missing values are imputed by impute.knn() and no NA values persist in the matrix
- Top-value scores are ordered and only the top n rows with the highest scores are passed forward

## Limitations

- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies may miss small cell clusters
- Feature selection precedes partitioning, so features excluded by the top-value cutoff cannot influence subgroup discovery

## Evidence
