---
name: genomic-subgroup-classification
description: Use when given a high-dimensional gene expression or methylation matrix
  (rows = features, columns = samples) and the goal is to assign samples to molecular
  subgroups without known labels.
when_to_use_negative:
- The input is single-cell RNA-Seq with several thousands of cells — the article notes
  cola's runtime and memory usage may be unacceptable, and downsampling may miss small
  cell clusters
- A trusted sample classification already exists and the goal is only downstream interpretation,
  not de novo subgroup discovery
- The matrix is already feature-selected and scaled with no missing values to impute,
  so adjust_matrix() preprocessing would be redundant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: 'Core R/Bioconductor package providing the general consensus partitioning
    framework: adjust_matrix() preprocessing, top-value feature selection (SD, ATC),
    consensus partitioning, 1-PAC scoring, and HTML report generation'
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: R package supplying impute.knn() for k-nearest-neighbour imputation of missing
    values in adjust_matrix()
- name: clue
  role: R package whose cl_consensus() function infers consensus subgroup labels from
    the individual bootstrap partitions
- name: bsub
  role: R package used to submit the cola analysis scripts as jobs to an LSF computing
    cluster
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:966e438e1930a7b6241ea62549c2f96350c9b1db006652199242080084bb1ae7
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
  - clue
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genomic-subgroup-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consensus-partitioning workflow for discovering molecular subgroups in high-throughput genomic matrices, implemented in the R/Bioconductor package cola, combining feature selection (top-value methods such as SD and ATC), row/column bootstrap resampling, and multiple partitioning methods (e.g., hclust, skmeans) with 1-PAC scoring to pick the best classification.

## When to use

Use when given a high-dimensional gene expression or methylation matrix (rows = features, columns = samples) and the goal is to assign samples to molecular subgroups without known labels.

## When NOT to use

- The input is single-cell RNA-Seq with several thousands of cells — the article notes cola's runtime and memory usage may be unacceptable, and downsampling may miss small cell clusters
- A trusted sample classification already exists and the goal is only downstream interpretation, not de novo subgroup discovery
- The matrix is already feature-selected and scaled with no missing values to impute, so adjust_matrix() preprocessing would be redundant

## Inputs

- High-dimensional genomic data matrix (e.g., TCGA GBM expression matrix from https://gdc.cancer.gov/about-data/publications/gbm_exp), rows = genes/features, columns = samples, possibly containing missing values
- User-specified parameters: top-value method (SD, MAD, CV, ATC), partitioning method (hclust, skmeans, ...), range of k (e.g., 2–6), number of bootstrap repetitions (e.g., 100), sampling probability p

## Outputs

- Consensus subgroup (partition) assignments for each sample at each k
- 1-PAC stability scores per top-value/partitioning/k combination
- Signature rows per subgroup (samples with silhouette scores < 0.5 excluded by default from signature analysis)
- Functional enrichment results and visualizations, optionally packaged as a comprehensive HTML report

## How to apply

First preprocess the raw matrix with cola's adjust_matrix(), which imputes missing values with impute.knn() from the R package impute, winsorizes values beyond the 5th/95th percentiles, and removes rows with zero variance or variance below the 5th percentile of all row variances. Scale rows (z-score or min-max) to remove absolute-level differences, then score rows with a top-value method — SD, MAD, CV, or the article's ATC (ability to correlate to other rows) — and select the top-scoring rows for partitioning. Run consensus partitioning (e.g., run_all_consensus_partition_methods()) over k = 2 to 6 with ~100 bootstrap repetitions, sampling a subset of rows or columns with probability p and applying a partitioning method such as hclust or the recommended spherical k-means (skmeans); infer consensus labels via cl_consensus() from the R package clue. Benchmark the article's own TCGA GBM matrix (https://gdc.cancer.gov/about-data/publications/gbm_exp) shows ATC and skmeans outperform other common choices, so prefer that combination, and evaluate stability with the 1-PAC score (higher is better) across method/k combinations. Rationale: cola formalizes consensus partitioning into a general framework with user-definable methods at each step and can generate a comprehensive HTML report of the whole analysis.

## Related tools

- **cola** (Core R/Bioconductor package providing the general consensus partitioning framework: adjust_matrix() preprocessing, top-value feature selection (SD, ATC), consensus partitioning, 1-PAC scoring, and HTML report generation) — https://github.com/jokergoo/cola_manuscript
- **impute** (R package supplying impute.knn() for k-nearest-neighbour imputation of missing values in adjust_matrix())
- **clue** (R package whose cl_consensus() function infers consensus subgroup labels from the individual bootstrap partitions)
- **bsub** (R package used to submit the cola analysis scripts as jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub

## Examples

```
library(cola); mat = adjust_matrix(mat); rl = run_all_consensus_partition_methods(mat, top_value_method = c("SD","ATC"), partition_method = c("hclust","skmeans"), k = 2:6)
```

## Evaluation signals

- Reproducing the TCGA GBM row-vs-column sampling comparison should yield mean 1-PAC differences of approximately 0.023 (SD:hclust), 0.019 (SD:skmeans), 0.037 (ATC:hclust), and 0.046 (ATC:skmeans), i.e., a small but consistent row-sampling advantage
- Mean concordance between row-sampled and column-sampled partitions should be approximately 0.939 for ATC:skmeans at k = 3, indicating the two sampling types agree highly
- 1-PAC scores for the chosen ATC + skmeans combination should be competitive with or better than other top-value/partitioning combinations on the dataset
- Preprocessing invariants: no missing values remain after impute.knn(), outlier values are capped at the 5th/95th percentiles, and all rows with zero variance (or variance below the 5th percentile of row variances) are removed before partitioning
- The analysis should run across the specified k range (e.g., 2–6) with the specified number of bootstrap repetitions (e.g., 100) and produce consensus labels for every method/k combination requested

## Limitations

- Runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling (row/column sampling) strategies may miss small cell clusters
- Weighting consensus partitions across different methods is complex and merging classifications can produce ambiguous labels
- The article text reviewed here does not state version numbers for cola or R, limiting exact version pinning

## Evidence
