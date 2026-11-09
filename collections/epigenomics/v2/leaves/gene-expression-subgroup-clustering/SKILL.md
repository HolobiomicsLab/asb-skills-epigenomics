---
name: gene-expression-subgroup-clustering
description: Use when a high-dimensional gene-expression matrix (e.g. TCGA GBM from
  gdc.cancer.gov/about-data/publications/gbm_exp) must be stratified into stable molecular
  subgroups and the effect of resampling count on consensus stability must be quantified.
when_to_use_negative:
- Single-cell RNA-Seq datasets with several thousands of cells, where cola runtime
  and memory usage may be unacceptable and downsampling may miss small cell clusters
- The goal is feature selection rather than sample stratification — ATC/SD/MAD/CV
  top-value benchmarking is a different skill
- The input is already a consensus partition or signature table rather than a raw
  expression matrix requiring adjust_matrix() preprocessing
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: cola
  role: Consensus partitioning framework; adjust_matrix() for preprocessing, MAD top-value
    method, hclust partitioning, 1-PAC stability metric
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: impute.knn() to impute missing values during matrix adjustment
- name: bsub
  role: Submit cola analysis jobs to an LSF computing cluster
  repo: https://github.com/jokergoo/bsub
- name: R
  role: Implementation and scripting language for the cola workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:0dd5d7b3b82df90cf8f09b16a37fc05980c1b23d873dc96ef8b2e786c62a098e
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-expression-subgroup-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A cola-based consensus partitioning protocol for gene-expression subgroup discovery, benchmarking how the number of random row samplings (25, 50, 100, 200) affects partition stability (1-PAC SD) and concordance on the TCGA GBM dataset with MAD:hclust at k=2.

## When to use

Use when a high-dimensional gene-expression matrix (e.g. TCGA GBM from gdc.cancer.gov/about-data/publications/gbm_exp) must be stratified into stable molecular subgroups and the effect of resampling count on consensus stability must be quantified.

## When NOT to use

- Single-cell RNA-Seq datasets with several thousands of cells, where cola runtime and memory usage may be unacceptable and downsampling may miss small cell clusters
- The goal is feature selection rather than sample stratification — ATC/SD/MAD/CV top-value benchmarking is a different skill
- The input is already a consensus partition or signature table rather than a raw expression matrix requiring adjust_matrix() preprocessing

## Inputs

- Preprocessed TCGA GBM gene expression matrix (https://gdc.cancer.gov/about-data/publications/gbm_exp)
- Row-sampling sizes: 25, 50, 100, 200
- Number of repetitions per setting: 100
- Top-value method: MAD; partitioning method: hclust; k = 2

## Outputs

- Average SD of 1-PAC stability metric per sampling size (expected: 0.0077, 0.0063, 0.0054, 0.0050 for 25/50/100/200)
- Mean concordance between 25-row and 200-row consensus partitions (0.973 vs 0.991)
- stability_summary.csv with per-setting stability and concordance statistics

## How to apply

Load the TCGA GBM expression matrix and preprocess it with cola's adjust_matrix(), which imputes missing values (impute.knn from the impute package), winsorizes outliers at the 5th/95th percentiles, and removes low-variance rows (zero variance or below the 5th percentile of row variances). For each row-sampling size (25, 50, 100, 200 rows), run cola consensus partitioning with 100 repetitions using the MAD top-value method with hclust at k=2, selecting top rows by MAD scores. Compute the average SD of the 1-PAC stability metric across repetitions per sampling size and compare consensus partitions between the 25-row and 200-row settings via mean concordance. Expect only marginal stability gains with more samplings (avg SD of 1-PAC: 0.0077, 0.0063, 0.0054, 0.0050) and near-complete agreement (0.973 vs 0.991) for a stable partition. Assemble per-setting stability and concordance statistics into stability_summary.csv.

## Related tools

- **cola** (Consensus partitioning framework; adjust_matrix() for preprocessing, MAD top-value method, hclust partitioning, 1-PAC stability metric) — https://github.com/jokergoo/cola_manuscript
- **impute** (impute.knn() to impute missing values during matrix adjustment)
- **bsub** (Submit cola analysis jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub
- **R** (Implementation and scripting language for the cola workflow)

## Examples

```
library(cola); mat = adjust_matrix(gbm_mat); rl = run_all_consensus_partition_methods(mat, top_value_method = "MAD", partition_method = "hclust", k = 2, mc.cores = 4)
```

## Evaluation signals

- Average SD of 1-PAC across 100 repetitions shows a monotonically decreasing trend with sampling size: 0.0077 (25), 0.0063 (50), 0.0054 (100), 0.0050 (200)
- Mean concordance between consensus partitions from 25-row and 200-row samplings is ≥ 0.973 when the partition is stable (0.973 vs 0.991 for MAD:hclust, k=2)
- stability_summary.csv exists and contains per-setting stability and concordance statistics for all four sampling sizes
- Preprocessing succeeded: missing values imputed via impute.knn(), outliers winsorized at the 5th/95th percentiles, zero-variance and <5th-percentile-variance rows removed
- The partition is judged stable (low 1-PAC) before interpreting concordance values

## Limitations

- Increasing the number of random row samplings only marginally improves partition stability — diminishing returns beyond 25 samplings
- cola runtime and memory usage may be unacceptable for datasets with several thousands of cells (e.g. scRNA-Seq)
- Downsampling strategies may fail to detect small clusters
- No version number stated for the cola package or R in the section, limiting reproducibility of versioned results

## Evidence
