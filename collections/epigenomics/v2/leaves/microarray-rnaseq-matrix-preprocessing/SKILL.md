---
name: microarray-rnaseq-matrix-preprocessing
description: Reference for cleaning raw gene-expression matrices (missing values,
  outliers, low-variance rows) before consensus partitioning or benchmarking clustering
  methods.
when_to_use_negative:
- Input is raw sequencing reads (e.g., FASTQ) or single-cell count data with several
  thousands of cells — preprocessing assumes an already-quantified matrix, and cola's
  runtime/memory may be unacceptable for very large scRNA-Seq data
- Input is methylation data requiring probe-level filtering (sex-chromosome probes,
  SNP-overlapping probes) that adjust_matrix() does not perform
- The matrix is already feature-selected or normalized for a method whose own preprocessing
  steps should be preserved instead
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: cola
  role: Provides adjust_matrix() to impute missing values, adjust outliers and remove
    rows with very small variance from the matrix
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: impute.knn() imputes remaining missing values in the expression matrix after
    rows with >25% missing samples are removed
- name: bsub
  role: Submits the per-dataset preprocessing and cola jobs to an LSF computing cluster
    without leaving R
  repo: https://github.com/jokergoo/bsub
- name: R / Bioconductor
  role: Language and package ecosystem in which the preprocessing functions are run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:b6507dbd9aae9db77fb318c6e1a7c6c7a61406ce6cd3b36fecd3bebf6b1844b7
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
  - R / Bioconductor
  - bsub
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# microarray-rnaseq-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess gene-expression matrices (GDS microarray, recount2 RNA-Seq) with cola's adjust_matrix() — imputing missing values, winsorizing outliers, and removing low-variance rows — so they are suitable for consensus partitioning and benchmarking of top-value/partitioning method combinations.

## When to use

Reference for cleaning raw gene-expression matrices (missing values, outliers, low-variance rows) before consensus partitioning or benchmarking clustering methods.

## When NOT to use

- Input is raw sequencing reads (e.g., FASTQ) or single-cell count data with several thousands of cells — preprocessing assumes an already-quantified matrix, and cola's runtime/memory may be unacceptable for very large scRNA-Seq data
- Input is methylation data requiring probe-level filtering (sex-chromosome probes, SNP-overlapping probes) that adjust_matrix() does not perform
- The matrix is already feature-selected or normalized for a method whose own preprocessing steps should be preserved instead

## Inputs

- Gene-expression matrix for each dataset (206 GDS microarray matrices)
- recount2 RNA-Seq matrices (223 datasets downloaded from https://jhubiostatistics.shinyapps.io/recount/)
- Raw R matrix/data.frame of rows = genes/features, columns = samples

## Outputs

- Cleaned numeric matrix per dataset ready for cola consensus partitioning (top n features selectable via SD, MAD, CV, or ATC scores)
- Aggregated comparison of 1-PAC scores per method combination (benchmark_1pac_summary.csv) when used in the benchmarking workflow

## How to apply

Load each matrix (e.g., 206 GDS microarray matrices or 223 recount2 RNA-Seq matrices) into R and pass it through cola's adjust_matrix(), which implements the article's preprocessing chain: remove rows in which >25% of samples have missing values, impute remaining missing values with impute.knn() from the impute package, replace values above the 95th percentile or below the 5th percentile with the corresponding percentile values (winsorization), and drop rows with zero variance or variance below the 5th percentile of all row variances. For recount2 data, additionally remove genes with <2 reads in >50% of samples before adjust_matrix(). This standardization removes absolute-level differences and non-informative rows so that downstream top-value methods (SD, MAD, CV, ATC) and partitioning methods (e.g., skmeans, hclust) operate on comparable features. Large-scale runs across the 435 datasets can be submitted to an LSF cluster with the bsub package.

## Related tools

- **cola** (Provides adjust_matrix() to impute missing values, adjust outliers and remove rows with very small variance from the matrix) — https://github.com/jokergoo/cola_manuscript
- **impute** (impute.knn() imputes remaining missing values in the expression matrix after rows with >25% missing samples are removed)
- **bsub** (Submits the per-dataset preprocessing and cola jobs to an LSF computing cluster without leaving R) — https://github.com/jokergoo/bsub
- **R / Bioconductor** (Language and package ecosystem in which the preprocessing functions are run)

## Examples

```
library(cola); mat = adjust_matrix(mat)  # impute missing values, winsorize outliers, remove low-variance rows
```

## Evaluation signals

- No rows remain with >25% missing sample values; all remaining missing values are imputed by impute.knn()
- No row values exceed the 95th percentile or fall below the 5th percentile (winsorization applied)
- No rows with zero variance, and no rows with variance below the 5th percentile of all row variances, remain in the output matrix
- For recount2 inputs, genes with <2 reads in >50% of samples are absent from the cleaned matrix
- Downstream benchmark reproduces the expected pattern: combinations with ATC and/or skmeans yield higher 1-PAC scores and more partitions with 1-PAC > 0.9 than hclust combinations

## Limitations

- Rows are entirely discarded when >25% of samples have missing values, which may remove informative features in sparse datasets
- The 5th-percentile variance cutoff and 5th/95th-percentile winsorization are fixed defaults; datasets with unusual variance distributions may lose valid features
-  cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells, and downsampling strategies may miss small cell clusters

## Evidence
