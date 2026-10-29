---
name: pac-clustering-stability-evaluation
description: Use when comparing stability of consensus subgroup classifications on
  a gene expression matrix — e.g., judging whether row sampling outperforms column
  sampling, or whether ATC/skmeans yields more consistent partitions than alternatives.
when_to_use_negative:
- Input is raw unprocessed expression data — missing values, outliers, and low-variance
  rows must be handled by adjust_matrix() before partitioning, so raw stability comparisons
  are not meaningful
- ScRNA-Seq datasets with several thousands of cells, where cola runtime and memory
  usage may be unacceptable
- The goal is detecting very small clusters — downsampling (row/column sampling) may
  miss small cell clusters, so PAC-based stability on sampled partitions can be misleading
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cola
  role: Provides consensus partitioning framework, adjust_matrix() preprocessing,
    top-value methods (SD, ATC), partitioning methods (hclust, skmeans), and consensus
    statistics used to compute PAC/1-PAC
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: impute.knn() from the R package impute fills missing values in the expression
    matrix before partitioning
- name: clue
  role: cl_consensus() infers consensus subgroup labels from the individual sampled
    partitions
- name: bsub
  role: Optionally submits the cola consensus partitioning jobs to an LSF computing
    cluster
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:47b1e061ac11eee08a166561153976d2af6d1cc5dfc4e966c372029ce930c468
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pac-clustering-stability-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate stability of consensus partitions from the cola framework by computing the 1-PAC score (proportion of ambiguous clustering) across top-value methods (SD, ATC), partitioning methods (hclust, skmeans), sampling schemes (rows vs columns) and k values, then comparing mean 1-PAC differences and partition concordance.

## When to use

Use when comparing stability of consensus subgroup classifications on a gene expression matrix — e.g., judging whether row sampling outperforms column sampling, or whether ATC/skmeans yields more consistent partitions than alternatives.

## When NOT to use

- Input is raw unprocessed expression data — missing values, outliers, and low-variance rows must be handled by adjust_matrix() before partitioning, so raw stability comparisons are not meaningful
- ScRNA-Seq datasets with several thousands of cells, where cola runtime and memory usage may be unacceptable
- The goal is detecting very small clusters — downsampling (row/column sampling) may miss small cell clusters, so PAC-based stability on sampled partitions can be misleading

## Inputs

- Preprocessed gene expression matrix (e.g., TCGA GBM from https://gdc.cancer.gov/about-data/publications/gbm_exp) after cola adjust_matrix() treatment
- Vector of k values (e.g., k = 2 to 6)
- Top-value method choices (SD, ATC) and partitioning method choices (hclust, skmeans)
- Number of bootstrap repetitions (e.g., 100) and sampling probability p for row/column subsampling

## Outputs

- Per-combination mean 1-PAC scores averaged over repetitions
- Mean 1-PAC difference between row sampling and column sampling per method combination
- Mean concordance between row-sampled and column-sampled partitions (e.g., ATC:skmeans k=3)
- Comparison table written to gbm_row_column_sampling_comparison.csv

## How to apply

Load the expression matrix (e.g., TCGA GBM from https://gdc.cancer.gov/about-data/publications/gbm_exp) and preprocess with cola's adjust_matrix(), which imputes missing values via impute.knn(), winsorizes at the 5th/95th percentiles, and removes low/zero-variance rows. Run consensus partitioning (run_all_consensus_partition_methods or consensus_partition loops) for SD and ATC top-value methods with hclust and skmeans partitioning, k from 2 to 6, with 100 bootstrap repetitions where a subset of rows or columns is randomly sampled with probability p, generating individual partitions whose consensus labels are inferred via cl_consensus() from the R package clue. For each top-value/partitioning/sampling/k combination compute the 1-PAC score (1 minus the proportion of ambiguous clustering in the 40% middle segment of the consensus matrix) and average over repetitions. Compute mean 1-PAC differences between row and column sampling (expected ~0.023 for SD:hclust, ~0.019 for SD:skmeans, ~0.037 for ATC:hclust, ~0.046 for ATC:skmeans) and the mean concordance between row-sampled and column-sampled partitions (expected ~0.939 for ATC:skmeans at k=3). A combination with higher mean 1-PAC and high cross-sampling concordance is judged more stable; cola recommends skmeans with the ATC feature-selection method.

## Related tools

- **cola** (Provides consensus partitioning framework, adjust_matrix() preprocessing, top-value methods (SD, ATC), partitioning methods (hclust, skmeans), and consensus statistics used to compute PAC/1-PAC) — https://github.com/jokergoo/cola_manuscript
- **impute** (impute.knn() from the R package impute fills missing values in the expression matrix before partitioning)
- **clue** (cl_consensus() infers consensus subgroup labels from the individual sampled partitions)
- **bsub** (Optionally submits the cola consensus partitioning jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub

## Examples

```
library(cola); mat = adjust_matrix(gbm_exp); rl = run_all_consensus_partition_methods(mat, top_value_method = c("SD","ATC"), partition_method = c("hclust","skmeans"), k = 2:6)
```

## Evaluation signals

- Mean 1-PAC differences (row minus column sampling) should approximate 0.023 for SD:hclust, 0.019 for SD:skmeans, 0.037 for ATC:hclust, and 0.046 for ATC:skmeans — a small but consistent row-sampling advantage
- Mean concordance between row- and column-sampled partitions for ATC:skmeans at k=3 should be approximately 0.939, indicating high agreement between sampling types
- Row sampling should yield higher 1-PAC than column sampling across all four method combinations
- Consensus labels must be reproducibly inferred from the individual partitions via cl_consensus() and labels adjusted consistently across partitions before concordance computation
- Output CSV (gbm_row_column_sampling_comparison.csv) should contain one row per top-value/partitioning method combination with mean 1-PAC differences and concordance values

## Limitations

- Row/column subsampling may fail to detect small clusters, biasing stability estimates
- Runtime and memory usage may be unacceptable for very high-sample-count data (e.g., scRNA-Seq with several thousands of cells)
- No changelog available for the manuscript scripts, limiting reproducibility of versioned results; no version number stated for the cola package or R

## Evidence
