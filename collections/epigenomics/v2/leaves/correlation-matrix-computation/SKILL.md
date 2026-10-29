---
name: correlation-matrix-computation
description: Use when a preprocessed high-dimensional matrix (rows = features such
  as genes or CpG probes, columns = samples) needs per-row feature scores based on
  global correlation structure before consensus partitioning in cola.
when_to_use_negative:
- The matrix has already been feature-selected or the goal is per-row variance scoring
  (SD, MAD, CV) rather than global-correlation-based scoring
- The input is a single-row or very small matrix where a distribution of correlations
  to 'other rows' is not meaningful
- The dataset is a scRNA-Seq matrix with several thousands of cells where cola's runtime
  and memory usage may be unacceptable per the article's discussion
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: cola
  role: Implements the ATC top-value method that scores rows via their correlations
    to all other rows for feature selection prior to consensus partitioning
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Language in which the row-wise absolute correlation matrix and per-row empirical-CDF
    integrals are computed
- name: impute
  role: Provides impute.knn() used to fill missing values in the input matrix before
    correlation computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:e8096f0de8e63bd94d803a05a4d66018a434c69c1f02babe4aa4af0c76658694
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
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# correlation-matrix-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes pairwise absolute correlations between all rows of a numeric matrix as the computational core of cola's ATC (ability to correlate to other rows) top-value method, so rows that correlate strongly with many other rows receive high feature-selection scores.

## When to use

Use when a preprocessed high-dimensional matrix (rows = features such as genes or CpG probes, columns = samples) needs per-row feature scores based on global correlation structure before consensus partitioning in cola.

## When NOT to use

- The matrix has already been feature-selected or the goal is per-row variance scoring (SD, MAD, CV) rather than global-correlation-based scoring
- The input is a single-row or very small matrix where a distribution of correlations to 'other rows' is not meaningful
- The dataset is a scRNA-Seq matrix with several thousands of cells where cola's runtime and memory usage may be unacceptable per the article's discussion

## Inputs

- Preprocessed numeric matrix with rows as features and columns as samples (e.g., gene expression matrix from the golubEsets Bioconductor package)
- Optionally, an adjust_matrix()-cleaned matrix with imputed missing values, winsorized outliers (5th/95th percentiles), and low-variance rows removed

## Outputs

- Per-row ATC score vector within [0, 1]
- Two-column table (feature name, ATC score) written to atc_scores.csv
- Ordered top-n feature list for consensus partitioning

## How to apply

Start from a preprocessed numeric matrix (e.g., a gene expression matrix from golubEsets), optionally passed through cola's adjust_matrix() which imputes missing values with impute.knn() from the R package impute, winsorizes values at the 5th/95th percentiles, and removes rows with variance below the 5th percentile. Compute the pairwise absolute correlation coefficient between every row and all other rows. For each row i, build the empirical CDF F_X(x) over its vector of absolute correlations to other rows and score it as ATC_i = 1 - integral of F_X(x) dx from 0 to 1, i.e., the area above the CDF curve. Assemble the per-row scores into a two-column table (feature name, ATC score), order them, and select only the top n rows with the highest scores for consensus partitioning.

## Related tools

- **cola** (Implements the ATC top-value method that scores rows via their correlations to all other rows for feature selection prior to consensus partitioning) — https://github.com/jokergoo/cola_manuscript
- **R** (Language in which the row-wise absolute correlation matrix and per-row empirical-CDF integrals are computed)
- **impute** (Provides impute.knn() used to fill missing values in the input matrix before correlation computation)

## Examples

```
Rscript -e 'library(cola); s = ATC(mat); write.csv(data.frame(row = names(s), score = s), "atc_scores.csv", row.names = FALSE)'
```

## Evaluation signals

- atc_scores.csv exists and contains exactly one score per input matrix row
- Every ATC value lies within [0, 1], consistent with the area-above-CDF definition
- Rows known to correlate strongly with many other rows receive higher ATC scores than weakly correlated rows
- The pairwise absolute correlation matrix is symmetric with unit diagonal before per-row CDF construction
- Selecting the top n rows by ATC score yields a subset suitable for downstream consensus partitioning without empty selections

## Limitations

- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells, limiting correlation-matrix computation at scale
- The ATC score depends on the whole correlation distribution, so heavy preprocessing artifacts (missing values, outliers, low-variance rows) distort scores if adjust_matrix() steps are skipped
- No version number for cola or R is stated in the manuscript, so exact numeric reproducibility of scores may vary across package versions

## Evidence
