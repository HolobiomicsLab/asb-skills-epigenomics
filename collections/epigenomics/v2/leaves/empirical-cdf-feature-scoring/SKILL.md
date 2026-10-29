---
name: empirical-cdf-feature-scoring
description: Use when a high-dimensional genomic matrix (e.g., gene expression or
  CpG methylation) needs feature scoring before partitioning and variance-based top-value
  methods (SD, MAD, CV) are insufficient because the global correlation structure
  matters.
when_to_use_negative:
- The matrix already contains a small curated feature set and no feature selection
  step is needed before partitioning
- Features are independent or the goal is purely per-feature variance ranking where
  correlation to other rows is irrelevant (SD/MAD/CV top-value methods suffice)
- The input is single-cell RNA-Seq with several thousands of cells, where cola's runtime
  and memory usage may be unacceptable per the article's discussion
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: cola
  role: Implements the ATC top-value method that scores matrix rows via the CDF of
    their absolute correlations to other rows for feature selection before consensus
    partitioning
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Language in which the pairwise row correlations, empirical CDFs, and ATC score
    integration are computed
- name: Bioconductor
  role: Source of example input matrices such as the Golub leukemia dataset from golubEsets
- name: impute
  role: Provides impute.knn() used by adjust_matrix() to impute missing values before
    feature scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:e9ffa8948b7afb2efbea16d8d102d03183d354df8df6f424dc9118209fa33fac
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# empirical-cdf-feature-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-row feature scores from the empirical CDF of a row's absolute correlation coefficients to all other rows (the ATC method in cola), so rows strongly correlated with many other rows receive high scores and can be selected as top features for consensus partitioning.

## When to use

Use when a high-dimensional genomic matrix (e.g., gene expression or CpG methylation) needs feature scoring before partitioning and variance-based top-value methods (SD, MAD, CV) are insufficient because the global correlation structure matters.

## When NOT to use

- The matrix already contains a small curated feature set and no feature selection step is needed before partitioning
- Features are independent or the goal is purely per-feature variance ranking where correlation to other rows is irrelevant (SD/MAD/CV top-value methods suffice)
- The input is single-cell RNA-Seq with several thousands of cells, where cola's runtime and memory usage may be unacceptable per the article's discussion

## Inputs

- Preprocessed numeric matrix in R with rows as features and columns as samples (e.g., gene expression matrix from the golubEsets Bioconductor package)
- Optional: matrix already passed through cola's adjust_matrix() (missing-value imputation, outlier adjustment, low-variance row removal)

## Outputs

- Per-row ATC scores as a two-column table (feature name, ATC score), e.g., written to atc_scores.csv
- Ordered top-n feature subset for consensus partitioning

## How to apply

Load the preprocessed numeric matrix in R (rows as features, columns as samples), ideally after running cola's adjust_matrix() which imputes missing values with impute.knn(), winsorizes values beyond the 5th/95th percentiles, and removes low-variance rows. Compute the pairwise absolute correlation coefficients between every row and all other rows. For each row i, build the empirical CDF F_X(x) over its vector of absolute correlations to other rows and compute ATC_i = 1 - integral of F_X(x) dx, i.e., the area above the CDF curve, which increases when row i correlates strongly with many other rows. Order the per-row scores and select only the top n rows with the highest ATC scores as input to consensus partitioning (e.g., with the recommended spherical k-means, skmeans). Write the scores as a two-column table (feature name, ATC score) for validation and downstream use.

## Related tools

- **cola** (Implements the ATC top-value method that scores matrix rows via the CDF of their absolute correlations to other rows for feature selection before consensus partitioning) — https://github.com/jokergoo/cola_manuscript
- **R** (Language in which the pairwise row correlations, empirical CDFs, and ATC score integration are computed)
- **Bioconductor** (Source of example input matrices such as the Golub leukemia dataset from golubEsets)
- **impute** (Provides impute.knn() used by adjust_matrix() to impute missing values before feature scoring)

## Examples

```
Rscript -e 'm <- adjust_matrix(expr_matrix); atc <- ATC(m); write.csv(data.frame(feature=names(atc), score=atc), "atc_scores.csv")'
```

## Evaluation signals

- atc_scores.csv exists and contains exactly one score per input matrix row
- Every ATC value lies within [0, 1], since the score is the area above a CDF curve
- Rows with many strong absolute correlations to other rows rank higher than rows with weak correlations (monotonicity of score with correlation strength)
- Ordering the scores and taking the top n rows yields a feature subset of the expected size for consensus partitioning
- Downstream skmeans consensus partitioning on top-ATC features outperforms other top-value/cluster method combinations as reported in the cola benchmark of 435 public gene expression datasets

## Limitations

- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling of rows/columns during partitioning may miss small cell clusters
- ATC requires computing all pairwise row correlations, which scales with the number of features in high-dimensional genomic data

## Evidence
