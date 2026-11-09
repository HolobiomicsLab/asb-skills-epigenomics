---
name: genomic-matrix-missing-value-filtering
description: Use when a raw numeric expression/methylation matrix destined for cola
  consensus partitioning contains missing values, extreme outliers, or rows with very
  small variance that would distort row-wise scaling and feature selection.
when_to_use_negative:
- The matrix is already fully preprocessed (no NAs, rows already variance-filtered)
  — re-running winsorization and variance filtering would drop informative rows and
  clamp true extremes.
- The analysis goal is dataset-specific sample/cell QC rather than matrix preprocessing
  — e.g., removing cells with <2 million reads or non-protein-coding genes in the
  MCF10CA scRNA-Seq workflow, which uses different per-dataset filter steps.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: Provides adjust_matrix(), which implements imputation of missing values, outlier
    adjustment, and removal of very small-variance rows
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: R package supplying impute.knn() for k-nearest-neighbor imputation of remaining
    missing values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:0263330ad118a35a7794d8d00a83c31066e58d861b79dd035c91cc44d910bf82
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
  - impute
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# genomic-matrix-missing-value-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sequentially cleans a numeric genomic data matrix (genes x samples) via cola's adjust_matrix() before consensus partitioning: remove rows with >25% missing values, kNN-impute remaining NAs, winsorize at the 5th/95th percentiles, and drop zero-variance and low-variance (below the 5th variance percentile) rows.

## When to use

Use when a raw numeric expression/methylation matrix destined for cola consensus partitioning contains missing values, extreme outliers, or rows with very small variance that would distort row-wise scaling and feature selection.

## When NOT to use

- The matrix is already fully preprocessed (no NAs, rows already variance-filtered) — re-running winsorization and variance filtering would drop informative rows and clamp true extremes.
- The analysis goal is dataset-specific sample/cell QC rather than matrix preprocessing — e.g., removing cells with <2 million reads or non-protein-coding genes in the MCF10CA scRNA-Seq workflow, which uses different per-dataset filter steps.

## Inputs

- Numeric gene expression (or CpG methylation) matrix with rows as genes/features and columns as samples, may contain NA values (e.g., Golub leukemia matrix from golubEsets)

## Outputs

- Cleaned numeric matrix with no missing values, winsorized values, and rows with zero or sub-5th-percentile variance removed (e.g., preprocessed_matrix.csv)

## How to apply

Load the numeric matrix in R (e.g., Golub leukemia data from the golubEsets Bioconductor package), then apply the adjust_matrix() steps in order: (1) remove rows in which >25% of samples have missing values, (2) impute remaining missing values with impute.knn() from the R package impute, (3) winsorize by replacing values above the 95th percentile or below the 5th percentile with the corresponding percentile values, (4) remove rows with zero variance, and (5) remove rows whose variance falls below the 5th percentile of all row variances. The rationale is that imputation, outlier clamping, and variance filtering prevent high-missingness or flat rows from dominating top-value scoring (SD, MAD, CV, ATC) and row scaling in the downstream consensus partition. The output cleaned matrix (e.g., preprocessed_matrix.csv) feeds directly into feature selection and partitioning.

## Related tools

- **cola** (Provides adjust_matrix(), which implements imputation of missing values, outlier adjustment, and removal of very small-variance rows) — https://github.com/jokergoo/cola_manuscript
- **impute** (R package supplying impute.knn() for k-nearest-neighbor imputation of remaining missing values)

## Examples

```
M_adjusted = cola::adjust_matrix(mat)  # impute NAs, winsorize at 5th/95th percentiles, remove low-variance rows
```

## Evaluation signals

- No row in the output has more than 25% missing values, and no NAs remain after impute.knn() imputation.
- All output values lie within the [5th percentile, 95th percentile] bounds of the row values (winsorization invariant).
- No zero-variance rows remain and no row variance is below the 5th percentile of all row variances.
- The number of samples (columns) is unchanged by preprocessing; only rows are removed (output row count <= input row count).
- The resulting matrix matches the expected sequential order of operations (remove high-missing rows -> impute -> winsorize -> variance filter), e.g., as written to preprocessed_matrix.csv.

## Limitations

- The thresholds (25% missing, 5th/95th percentile winsorization, 5th variance-percentile cutoff) are defaults; datasets with genuinely low-variance but informative features may lose rows irreversibly.
- kNN imputation assumes a correlated feature structure; in very sparse matrices where >25% of samples are missing per row, rows are discarded rather than imputed.

## Evidence
