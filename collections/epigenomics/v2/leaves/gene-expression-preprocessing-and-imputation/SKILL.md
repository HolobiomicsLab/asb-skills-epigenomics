---
name: gene-expression-preprocessing-and-imputation
description: Use when a raw gene expression matrix (e.g. from golubEsets Bioconductor
  data) contains missing values, outlier or sub-1 expression values, or low-variance
  rows that would break consensus partitioning.
when_to_use_negative:
- Input is already a preprocessed feature table with no missing values and stable
  variance (adjust_matrix() would needlessly re-filter rows)
- Single-cell datasets with many thousands of cells, where cola's runtime and memory
  usage may be unacceptable
- Datasets where >25% of values are missing per row, so imputation by k-nearest neighbors
  would be unreliable rather than a cleanup step
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: cola
  role: Provides adjust_matrix() to impute missing values, adjust outliers and remove
    rows with very small variance before consensus partitioning
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: Supplies impute.knn() for k-nearest-neighbor imputation of missing expression
    values
- name: golubEsets
  role: Bioconductor package providing the Golub leukemia expression dataset used
    as input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:f397b69279d34553859092fba4393e5aa192e074d2a6754211c3368e43528e94
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
  - NMF
  - R
  - cola
  - golubEsets
  - impute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-expression-preprocessing-and-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses gene expression matrices for consensus partitioning with cola: quantile-normalize, set invalid low values to NA, impute missing values with impute.knn(), and clean the matrix with cola's adjust_matrix() so that downstream top-value selection and partitioning operate on a reliable numeric matrix.

## When to use

Use when a raw gene expression matrix (e.g. from golubEsets Bioconductor data) contains missing values, outlier or sub-1 expression values, or low-variance rows that would break consensus partitioning.

## When NOT to use

- Input is already a preprocessed feature table with no missing values and stable variance (adjust_matrix() would needlessly re-filter rows)
- Single-cell datasets with many thousands of cells, where cola's runtime and memory usage may be unacceptable
- Datasets where >25% of values are missing per row, so imputation by k-nearest neighbors would be unreliable rather than a cleanup step

## Inputs

- Raw gene expression matrix (e.g. Golub leukemia data from the golubEsets Bioconductor package)
- R session with Bioconductor packages: cola, impute, golubEsets

## Outputs

- Cleaned, imputed, row-scaled numeric expression matrix suitable for cola's run_all_consensus_partitions()
- Preprocessed matrix stored (e.g. saved with the full cola result object as golub_cola_results.rds)

## How to apply

Load the expression data (e.g. the Golub leukemia dataset from the golubEsets Bioconductor package) and quantile-normalize it. Set expression values below 1 to NA, then impute missing values with impute.knn() from the R package impute; as filter heuristics, rows in which >25% of samples have missing values can be removed, and values beyond the 5th/95th percentiles are winsorized. Pass the matrix through cola's adjust_matrix(), which imputes missing values, adjusts outliers, and removes rows with very small variance (zero-variance rows and rows with variance below the 5th percentile of all row variances). Before applying partitioning methods, scale matrix rows (e.g. z-score or min-max) to remove differences of absolute levels between rows. Rationale: consensus partitioning repeatedly samples rows and applies partitioning methods such as skmeans, so NaNs, outliers, or flat rows distort top-value scores (SD, CV, MAD, ATC) and the resulting consensus partitions.

## Related tools

- **cola** (Provides adjust_matrix() to impute missing values, adjust outliers and remove rows with very small variance before consensus partitioning) — https://github.com/jokergoo/cola_manuscript
- **impute** (Supplies impute.knn() for k-nearest-neighbor imputation of missing expression values)
- **golubEsets** (Bioconductor package providing the Golub leukemia expression dataset used as input)

## Examples

```
library(cola); library(impute); m[exprs(golub.eset) < 1] <- NA; m <- impute.knn(exprs(golub.eset))$data; m <- adjust_matrix(m)
```

## Evaluation signals

- The preprocessed matrix contains no NA/NaN values after impute.knn() and adjust_matrix()
- Rows with zero variance and rows with variance below the 5th percentile of all row variances have been removed
- Expression values originally < 1 were set to NA before imputation rather than carried through
- Downstream check: cola run on the cleaned matrix reproduces reported stability scores (k=2 ATC:kmeans 1-PAC = 1, ATC:skmeans 1-PAC = 1; k=3 ATC:skmeans 1-PAC = 0.96) within a small numeric tolerance
- The ATC:skmeans k=2 signature set contains exactly 1810 genes

## Limitations

- cola's runtime and memory usage may be unacceptable for single-cell datasets with several thousands of cells
- Downsampling strategies may miss small cell clusters (e.g. small cell populations in scRNA-Seq)
- No version number for cola or R is stated, limiting reproducibility of versioned results

## Evidence
