---
name: consensus-partitioning-method-benchmarking
description: Use when you must decide which top-value and partitioning method combination
  produces the most stable consensus partitions, e.g. comparing ATC/skmeans versus
  hclust-based combinations over many expression datasets.
when_to_use_negative:
- Input is a single small dataset where running all 24 method combinations across
  a cluster is unnecessary overhead rather than a benchmark
- scRNA-Seq datasets with several thousands of cells, where cola's runtime and memory
  usage may be unacceptable and downsampling may miss small cell clusters
- The goal is functional enrichment or signature analysis of an already-chosen partition
  rather than comparing partitioning methods
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: cola
  role: R/Bioconductor package performing consensus partitioning across method combinations,
    providing adjust_matrix() preprocessing and 1-PAC stability scoring
  repo: https://github.com/jokergoo/cola_manuscript
- name: bsub
  role: Submits the per-dataset, per-combination cola jobs to an LSF computing cluster
    from R
  repo: https://github.com/jokergoo/bsub
- name: impute
  role: Provides impute.knn() used during adjust_matrix() preprocessing to impute
    missing values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:dcf31e7735ce48057b27c43c6c16eec332aad106367a8adf3e12f1a91661ab34
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# consensus-partitioning-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks all combinations of top-value feature-selection methods (SD, MAD, CV, ATC) and partitioning methods (including skmeans and hclust) across hundreds of gene expression datasets, scoring partition stability with 1-PAC at the best k to show ATC/skmeans combinations yield more stable partitions.

## When to use

Use when you must decide which top-value and partitioning method combination produces the most stable consensus partitions, e.g. comparing ATC/skmeans versus hclust-based combinations over many expression datasets.

## When NOT to use

- Input is a single small dataset where running all 24 method combinations across a cluster is unnecessary overhead rather than a benchmark
- scRNA-Seq datasets with several thousands of cells, where cola's runtime and memory usage may be unacceptable and downsampling may miss small cell clusters
- The goal is functional enrichment or signature analysis of an already-chosen partition rather than comparing partitioning methods

## Inputs

- Gene expression matrices (e.g. GDS microarray matrices and recount2 RNA-Seq count matrices)
- List of 24 top-value method × partitioning method combinations (SD, MAD, CV, ATC × partitioning methods including skmeans and hclust)
- Range of k values to test (k = 2 to 6)

## Outputs

- Per-dataset, per-combination 1-PAC scores at the best k
- Aggregated benchmark summary table (e.g. benchmark_1pac_summary.csv) with mean 1-PAC and proportion of stable partitions (1-PAC > 0.9) per method combination
- cola HTML reports per dataset

## How to apply

Load each expression matrix (e.g. the 206 GDS microarray and 223 recount2 RNA-Seq matrices) and preprocess with cola's adjust_matrix(), which imputes missing values with impute.knn(), winsorizes outliers at the 5th/95th percentiles, and removes low-variance rows. For each dataset, run cola's consensus partitioning across all 24 combinations of top-value methods (SD, MAD, CV, ATC) and partitioning methods, testing k = 2 to 6, submitting jobs to an LSF cluster with the bsub package. For each combination and dataset, extract the 1-PAC score at the best k, then aggregate per combination to compute the mean 1-PAC and the proportion of datasets with stable partitions (1-PAC > 0.9). Compare combinations containing ATC and/or skmeans against hclust-containing combinations; the rationale is that ATC exploits the global correlation structure for feature selection and skmeans suits the scaled matrix geometry, while hclust yields low 1-PAC scores. Write the aggregated comparison table (e.g. benchmark_1pac_summary.csv) as the deliverable.

## Related tools

- **cola** (R/Bioconductor package performing consensus partitioning across method combinations, providing adjust_matrix() preprocessing and 1-PAC stability scoring) — https://github.com/jokergoo/cola_manuscript
- **bsub** (Submits the per-dataset, per-combination cola jobs to an LSF computing cluster from R) — https://github.com/jokergoo/bsub
- **impute** (Provides impute.knn() used during adjust_matrix() preprocessing to impute missing values)

## Examples

```
library(cola); library(bsub); mat = adjust_matrix(mat); rl = run_all_consensus_partition_methods(mat, k = 2:6);
```

## Evaluation signals

- Combinations containing ATC and/or skmeans attain higher 1-PAC scores at the best k than other combinations
- Combinations containing ATC and/or skmeans show a higher proportion of datasets with stable partitions (1-PAC > 0.9)
- hclust-containing combinations generate low 1-PAC scores across the 435-dataset benchmark
- Every dataset was preprocessed with adjust_matrix() (missing values imputed, outliers adjusted, low-variance rows removed) before partitioning
- Summary table contains one row per method combination with mean 1-PAC and stability proportion over all 206 GDS and 223 recount2 datasets

## Limitations

- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies may miss small cell clusters
- One dataset (GDS1761) was removed from the analysis due to an error of GEOquery

## Evidence

- [readme] bsub job submission: [evidence span withheld — anchor status: external_doc]
