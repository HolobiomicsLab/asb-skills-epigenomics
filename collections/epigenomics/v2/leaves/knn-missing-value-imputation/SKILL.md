---
name: knn-missing-value-imputation
description: Use when an input gene-expression or methylation matrix (e.g., from golubEsets,
  ALL, or recount2) contains missing values (NAs) that must be filled before feature
  selection and consensus partitioning.
when_to_use_negative:
- Matrix rows have more than 25% missing samples — these rows are removed rather than
  imputed
- Input matrix contains no missing values — imputation is skipped ('if there is any')
  and preprocessing proceeds directly to outlier adjustment and variance filtering
- Input is not a numeric matrix of samples vs. features (e.g., raw reads without summarization),
  since variance- and correlation-based KNN imputation is undefined on such data
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: impute
  role: R package whose impute.knn() function fills missing matrix values by k-nearest-neighbor
    imputation within cola's adjust_matrix() preprocessing
- name: cola
  role: R/Bioconductor package providing adjust_matrix(), the wrapper that runs KNN
    imputation as part of input-matrix cleaning prior to consensus partitioning
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Runtime environment for cola and the impute package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:ed9b42112f8fa14f06286e127bdcdf257ead791c9152425fb652d8dc3156ea8f
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
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# knn-missing-value-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Imputes missing values in a numeric sample-by-feature genomic matrix using impute.knn() from the R package impute, as implemented inside cola's adjust_matrix() preprocessing step before consensus partitioning.

## When to use

Use when an input gene-expression or methylation matrix (e.g., from golubEsets, ALL, or recount2) contains missing values (NAs) that must be filled before feature selection and consensus partitioning.

## When NOT to use

- Matrix rows have more than 25% missing samples — these rows are removed rather than imputed
- Input matrix contains no missing values — imputation is skipped ('if there is any') and preprocessing proceeds directly to outlier adjustment and variance filtering
- Input is not a numeric matrix of samples vs. features (e.g., raw reads without summarization), since variance- and correlation-based KNN imputation is undefined on such data

## Inputs

- Numeric sample-by-feature matrix (e.g., gene expression from the Golub leukemia dataset in golubEsets, or CpG methylation beta values)
- Matrix with missing values (NA) in rows where ≤25% of samples are missing

## Outputs

- Complete numeric matrix with no remaining NA values
- Cleaned matrix passed onward to winsorization, variance filtering, and consensus partitioning (e.g., writable as preprocessed_matrix.csv)

## How to apply

First remove rows in which more than 25% of the samples have missing values, so that only rows with sparse missingness are imputed. Then apply impute.knn() from the R package impute to fill the remaining missing values — imputation is conditional ('if there is any'), so a matrix without NAs skips this step. In cola, this imputation is embedded in the adjust_matrix() function and executed sequentially before winsorizing values at the 5th/95th percentiles and removing zero-variance and low-variance rows. The rationale is that downstream top-value scoring (SD, MAD, CV, ATC) and row scaling require a complete numeric matrix with no NA-driven failures in variance or correlation computations.

## Related tools

- **impute** (R package whose impute.knn() function fills missing matrix values by k-nearest-neighbor imputation within cola's adjust_matrix() preprocessing)
- **cola** (R/Bioconductor package providing adjust_matrix(), the wrapper that runs KNN imputation as part of input-matrix cleaning prior to consensus partitioning) — https://github.com/jokergoo/cola_manuscript
- **R** (Runtime environment for cola and the impute package)

## Examples

```
library(cola); M_clean <- adjust_matrix(M)
```

## Evaluation signals

- No NA values remain in the imputed matrix (complete.cases() or anyNA() returns FALSE)
- Rows with >25% missing samples were removed before imputation — row counts should reflect this filter
- Imputation occurred before winsorization: values beyond the 5th/95th percentiles should be replaced with percentile values only in the post-imputation matrix
- Row variances computed on the imputed matrix are finite (no NaN from empty rows), so zero-variance and 5th-percentile variance filters run without error
- Subsequent consensus partitioning (e.g., with skmeans) completes without NA-related failures

## Limitations

- KNN imputation adds runtime and memory cost; cola's authors note runtime and memory usage 'might not be acceptable if the number of cells becomes very high, e.g., several thousands' for scRNA-Seq data
- Imputation quality depends on row-level missingness — the article caps imputation at rows with ≤25% missing samples, so sparser rows must be discarded instead
- No version number for the impute package or cola is stated in the manuscript, limiting reproducibility of imputation results

## Evidence
