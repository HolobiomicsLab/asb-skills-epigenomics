---
name: atc-score-reconstruction
description: Use when a high-dimensional matrix (e.g., gene expression or CpG methylation)
  needs a feature-selection score for subgroup/consensus partitioning, especially
  when features should be ranked by their global correlation to other rows rather
  than by variance.
when_to_use_negative:
- The matrix is already a small, curated feature table where no further feature ranking
  is needed before clustering
- The goal is a per-row dispersion-based ranking (SD, MAD, CV) and the global correlation
  structure is irrelevant to the analysis
- The dataset is single-cell RNA-Seq with several thousands of cells, where cola's
  runtime and memory usage may be unacceptable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: cola
  role: Provides the ATC top-value method within its general consensus-partitioning
    framework; ATC scores order rows for feature selection before partitioning
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: impute.knn() imputes missing values in the input matrix prior to ATC score
    computation
- name: bsub
  role: Used to submit the ATC/cola analysis scripts as jobs to an LSF computing cluster
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:d7b818759c07beeca8251c1674dc140458f6b60cdc1562f57db5badea7971cf1
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
  - bsub
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# atc-score-reconstruction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct the ATC (ability to correlate to other rows) top-value scoring method from the cola R/Bioconductor framework, which assigns each matrix row a score in [0,1] based on the global correlation structure of that row to all other rows, so that the highest-scored rows can be selected for consensus partitioning.

## When to use

Use when a high-dimensional matrix (e.g., gene expression or CpG methylation) needs a feature-selection score for subgroup/consensus partitioning, especially when features should be ranked by their global correlation to other rows rather than by variance.

## When NOT to use

- The matrix is already a small, curated feature table where no further feature ranking is needed before clustering
- The goal is a per-row dispersion-based ranking (SD, MAD, CV) and the global correlation structure is irrelevant to the analysis
- The dataset is single-cell RNA-Seq with several thousands of cells, where cola's runtime and memory usage may be unacceptable

## Inputs

- Numeric matrix with rows as features and columns as samples (e.g., gene expression matrix from the golubEsets Bioconductor package)
- Optionally, a matrix preprocessed with cola's adjust_matrix() (imputed, winsorized, low-variance rows removed)

## Outputs

- Two-column table (feature name, ATC score) written to atc_scores.csv, with one score per input row and every ATC value in [0, 1]
- Ordered row scores for selecting the top n features for consensus partitioning

## How to apply

Load the preprocessed numeric matrix (rows = features, columns = samples; use cola's adjust_matrix() to impute missing values via impute.knn(), winsorize outliers at the 5th/95th percentiles, and drop low-variance rows). Compute the pairwise absolute correlation coefficients between every row and all other rows. For each row i, take X as the vector of absolute correlations to other rows, construct its empirical CDF F_X(x), and compute ATC_i = 1 - integral_0^1 F_X(x) dx, i.e., the area above the CDF curve, so rows correlating strongly with many other rows receive high scores. Assemble the per-row scores into a two-column table (feature name, ATC score), order scores, and pass the top n rows to the downstream consensus partitioning (for which cola recommends spherical k-means, skmeans).

## Related tools

- **cola** (Provides the ATC top-value method within its general consensus-partitioning framework; ATC scores order rows for feature selection before partitioning) — https://github.com/jokergoo/cola_manuscript
- **impute** (impute.knn() imputes missing values in the input matrix prior to ATC score computation)
- **bsub** (Used to submit the ATC/cola analysis scripts as jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub

## Examples

```
atc_scores = cola::ATC(mat); write.csv(data.frame(feature = rownames(mat), ATC = atc_scores), "atc_scores.csv", row.names = FALSE)
```

## Evaluation signals

- atc_scores.csv exists and contains exactly one score per input row (row count matches the input matrix)
- Every ATC value lies within [0, 1], since ATC_i = 1 - integral_0^1 F_X(x) dx is an area above a CDF
- Rows with stronger correlations to many other rows rank higher: the score equals the area above the empirical CDF of the row's absolute correlations, so monotonically correlated rows should have near-1 scores
- Selecting the top n rows by ATC score yields a subset suitable for consensus partitioning, e.g., with the recommended skmeans method, and downstream subgroup classification performs better than with other commonly used top-value methods in benchmarks
- Applying adjust_matrix() first means no rows with >25% missing values, no zero-variance rows, and no rows with variance below the 5th percentile remain before scoring

## Limitations

- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies may miss small cell clusters when the ATC-selected features are used in the full consensus partitioning workflow
- Scores depend on the correlation measure and on prior preprocessing (imputation, winsorization at the 5th/95th percentiles, removal of low-variance rows), so changes in these steps change the ranking

## Evidence

- [abstract] cola provides a new method named ATC (ability to correlate to other rows) to extract features and recommends spherical k-means clustering (skmeans) for subgroup classification.: "cola provides a new method named ATC (ability to correlate to other rows) to extract features and recommends spherical k-means clustering (skmeans) for subgroup classification."
- [intro] Top-value methods are used to assign scores to the rows of a matrix. Later the scores are ordered and only the top n rows with the highest scores are selected for consensus partitioning: "Top-value methods are used to assign scores to the rows of a matrix. Later the scores are ordered and only the top n rows with the highest scores are selected for consensus partitioning"
- [intro] cola implements a simple yet powerful method (the ATC method) for selecting features based on the global correlation structure: "cola implements a simple yet powerful method (the ATC method) for selecting features based on the global correlation structure"
- [readme] This repository contains scripts for the manuscript '**cola: an R/Bioconductor Package for Consensus Partitioning through a General Framework**'.: "This repository contains scripts for the manuscript '**cola: an R/Bioconductor Package for Consensus Partitioning through a General Framework**'."
