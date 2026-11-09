---
name: percentile-based-outlier-winsorization
description: Use when a numeric genomic data matrix (e.g. gene expression rows x samples)
  contains extreme outlier values that could distort row-wise statistics or downstream
  partitioning, as part of cola's adjust_matrix() cleaning pipeline.
when_to_use_negative:
- The input has already passed through adjust_matrix() or is otherwise outlier-adjusted
  and low-variance rows have been removed
- Downstream analysis legitimately requires untruncated extreme values (e.g. detecting
  genuine outlier samples rather than stabilizing feature statistics)
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cola
  role: R/Bioconductor package whose adjust_matrix() function implements the winsorization
    of values beyond the 5th/95th percentiles within the sequential matrix-cleaning
    procedure
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Language in which adjust_matrix() and the surrounding preprocessing pipeline
    are executed
- name: impute
  role: R package providing impute.knn(), the imputation step that must precede winsorization
    in the adjust_matrix() sequence
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:24adcd7bf81d2bef808ecd6e45cac87fc345ac1028193c6a3bc088c854419641
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
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Percentile-based outlier winsorization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A matrix-cleaning step from cola's adjust_matrix() preprocessing in which row values above the 95th percentile or below the 5th percentile are replaced by the corresponding percentile values, dampening extreme outliers before consensus partitioning.

## When to use

Use when a numeric genomic data matrix (e.g. gene expression rows x samples) contains extreme outlier values that could distort row-wise statistics or downstream partitioning, as part of cola's adjust_matrix() cleaning pipeline.

## When NOT to use

- The input has already passed through adjust_matrix() or is otherwise outlier-adjusted and low-variance rows have been removed
- Downstream analysis legitimately requires untruncated extreme values (e.g. detecting genuine outlier samples rather than stabilizing feature statistics)

## Inputs

- Numeric genomic data matrix (rows = genes/features, columns = samples), e.g. a gene expression matrix loaded from golubEsets
- Matrix already cleaned of rows with >25% missing values and imputed with impute.knn()

## Outputs

- Winsorized numeric matrix with values above the 95th percentile or below the 5th percentile replaced by the corresponding percentile values
- Fully cleaned matrix (preprocessed_matrix.csv) after subsequent zero-variance and low-variance row removal

## How to apply

Within cola's adjust_matrix() sequential preprocessing, after removing rows with >25% missing samples and imputing remaining missing values with impute.knn() from the R package impute, winsorize values by replacing those larger than the 95th percentile or smaller than the 5th percentile with the corresponding percentile values. Winsorization is performed before variance-based filtering, which subsequently removes rows with zero variance and rows whose variance falls below the 5th percentile of all row variances. The rationale is that extreme values can dominate row-variance and correlation-based feature statistics (e.g. the ATC top-value method), so capping them keeps feature selection and consensus partitioning robust. The cleaned matrix can then be written out (e.g. to preprocessed_matrix.csv) and used as input for top-value scoring and row scaling prior to partitioning.

## Related tools

- **cola** (R/Bioconductor package whose adjust_matrix() function implements the winsorization of values beyond the 5th/95th percentiles within the sequential matrix-cleaning procedure) — https://github.com/jokergoo/cola_manuscript
- **R** (Language in which adjust_matrix() and the surrounding preprocessing pipeline are executed)
- **impute** (R package providing impute.knn(), the imputation step that must precede winsorization in the adjust_matrix() sequence)

## Examples

```
library(cola); m = adjust_matrix(m)
```

## Evaluation signals

- No entry in the winsorized matrix exceeds the original 95th percentile or falls below the original 5th percentile (values are capped at the percentile values)
- Winsorization occurs only after rows with >25% missing samples are removed and impute.knn() imputation is applied, preserving the documented order of adjust_matrix() steps
- Rows with zero variance are absent from the output, and remaining row variances are >= the 5th percentile of all row variances
- Row and column dimensions correspond to the same features/samples as the input minus the filtered rows; the cleaned matrix is written to preprocessed_matrix.csv

## Limitations

- Winsorization is embedded in cola's adjust_matrix() preprocessing sequence, so applying it out of order (e.g. before imputation) deviates from the documented procedure
- Percentile capping is a fixed 5th/95th percentile rule; datasets requiring different truncation thresholds are outside the described default behavior

## Evidence
