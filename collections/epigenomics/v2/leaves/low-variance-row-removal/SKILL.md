---
name: low-variance-row-removal
description: Use when a numeric feature-by-sample matrix (gene expression, methylation)
  feeds into consensus partitioning and rows with near-constant values across samples
  would distort row scoring, scaling, or clustering.
when_to_use_negative:
- Rows are already filtered by an external quality-control step and the matrix contains
  only informative, variable features
- You need to keep low-variance rows for a dedicated test of those features rather
  than for clustering input
- Data is single-cell with thousands of cells where cola's runtime/memory may be unacceptable
  — consider its stated limitations first
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: Provides the adjust_matrix() function that executes the sequential preprocessing
    including low-variance row removal
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation language for the adjustment/filtering pipeline
- name: Bioconductor
  role: Distribution channel for cola and for input datasets such as golubEsets
- name: impute
  role: Supplies impute.knn() used upstream to fill missing values before variance
    filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:6d7eecc4743456c2f257085e053c356fe6007acc82f97802bbf1dcdd940af9db
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
  - Bioconductor
  - R
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# low-variance-row-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A matrix-cleaning step from cola's adjust_matrix() that removes rows (e.g., genes or CpG probes) with zero variance or variance below the 5th percentile of all row variances, so that downstream consensus partitioning is not dominated by non-informative features.

## When to use

Use when a numeric feature-by-sample matrix (gene expression, methylation) feeds into consensus partitioning and rows with near-constant values across samples would distort row scoring, scaling, or clustering.

## When NOT to use

- Rows are already filtered by an external quality-control step and the matrix contains only informative, variable features
- You need to keep low-variance rows for a dedicated test of those features rather than for clustering input
- Data is single-cell with thousands of cells where cola's runtime/memory may be unacceptable — consider its stated limitations first

## Inputs

- Numeric feature-by-sample matrix (e.g., gene expression from golubEsets Bioconductor package, CpG methylation probes)
- Matrix possibly containing missing values and outliers

## Outputs

- Cleaned numeric matrix with rows removed that have zero variance or variance below the 5th percentile of all row variances
- Matrix ready for top-value feature selection and row scaling prior to consensus partitioning

## How to apply

As part of cola's sequential adjust_matrix() preprocessing, first remove rows in which >25% of samples have missing values, impute remaining missing values with impute.knn() from the R package impute, then winsorize by replacing values above the 95th percentile or below the 5th percentile with the corresponding percentile values. Only after outlier adjustment, remove rows with exactly zero variance, then remove rows whose variance falls below the 5th percentile of all row variances. The ordering matters: outlier adjustment and imputation precede variance filtering so variances are computed on cleaned values. The output cleaned matrix then feeds top-value feature selection (SD, MAD, CV, ATC) and row scaling before partitioning.

## Related tools

- **cola** (Provides the adjust_matrix() function that executes the sequential preprocessing including low-variance row removal) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation language for the adjustment/filtering pipeline)
- **Bioconductor** (Distribution channel for cola and for input datasets such as golubEsets)
- **impute** (Supplies impute.knn() used upstream to fill missing values before variance filtering)

## Examples

```
M_clean = cola::adjust_matrix(M)
```

## Evaluation signals

- No row in the output matrix has exactly zero variance across samples
- The minimum remaining row variance is greater than or equal to the 5th percentile of the row variances computed after imputation and winsorization
- Rows removed total roughly the zero-variance rows plus 5% of remaining rows (modulo ties at the threshold)
- Order-of-operations check: variance filtering is computed on the winsorized/imputed matrix, not the raw one
- Downstream top-value scoring (SD/MAD/CV/ATC) and clustering run without errors on the cleaned matrix

## Limitations

- The 5th-percentile variance threshold is a fixed default; it may remove genuinely low-but-meaningful variation rows
- Filtering is applied before feature selection, so strongly predictive low-variance features could be lost from the partitioning input
- The full cola pipeline (of which this is one step) may have unacceptable runtime and memory usage for scRNA-Seq datasets with several thousands of cells

## Evidence
