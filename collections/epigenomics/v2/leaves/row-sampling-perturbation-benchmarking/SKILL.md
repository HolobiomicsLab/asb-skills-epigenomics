---
name: row-sampling-perturbation-benchmarking
description: Use when you need to justify or tune the number of random samplings (repetitions)
  in cola consensus partitioning, or to show that a low sampling count already yields
  a stable partition.
when_to_use_negative:
- Input is single-cell data with several thousands of cells — cola runtime and memory
  usage may be unacceptable, and downsampling may miss small cell clusters
- The goal is feature selection method comparison (e.g., ATC vs SD/MAD/CV) rather
  than sampling-size sensitivity of partition stability
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: cola
  role: Runs the consensus partitioning with configurable row sampling (p), top-value
    method (MAD), partitioning method (hclust), and provides adjust_matrix() for preprocessing
    and 1-PAC stability metrics
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: Provides impute.knn() used inside adjust_matrix() to impute missing values
    in the expression matrix
- name: bsub
  role: Submits the repeated cola consensus partitioning jobs to an LSF computing
    cluster
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:83a4ca593f885a8c1c4e712a755ef81b4a5e1c82cbd4551e9f879d68984be33e
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# row-sampling-perturbation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that varies the number of random row samplings (e.g., 25, 50, 100, 200) in cola consensus partitioning to quantify how sampling effort affects partition stability (SD of 1-PAC) and concordance between settings, using the TCGA GBM expression dataset as the exemplar input.

## When to use

Use when you need to justify or tune the number of random samplings (repetitions) in cola consensus partitioning, or to show that a low sampling count already yields a stable partition.

## When NOT to use

- Input is single-cell data with several thousands of cells — cola runtime and memory usage may be unacceptable, and downsampling may miss small cell clusters
- The goal is feature selection method comparison (e.g., ATC vs SD/MAD/CV) rather than sampling-size sensitivity of partition stability

## Inputs

- Preprocessed TCGA GBM gene expression matrix from https://gdc.cancer.gov/about-data/publications/gbm_exp
- Row-sampling sizes to benchmark (25, 50, 100, 200)
- Number of repetitions per setting (100)
- Partitioning configuration: MAD top-value method, hclust partitioning, k=2

## Outputs

- stability_summary.csv with per-setting average SD of 1-PAC and concordance statistics
- Per-setting consensus partitions for the compared sampling sizes

## How to apply

Preprocess the gene expression matrix with cola's adjust_matrix(), which imputes missing values (via impute.knn() from the impute package), adjusts outliers, and removes low-variance rows. Then, for each row-sampling size (25, 50, 100, 200), run cola consensus partitioning with 100 repetitions per setting using the MAD top-value method with hierarchical clustering (hclust) at k=2 on the TCGA GBM matrix (https://gdc.cancer.gov/about-data/publications/gbm_exp). For each sampling size, compute the average standard deviation of the 1-PAC stability metric across the 100 repetitions; the expected decreasing trend is 0.0077, 0.0063, 0.0054, 0.0050. Also compare the consensus partitions from 25-row versus 200-row samplings and compute mean concordance; stable partitions nearly agree (0.973 vs 0.991 for MAD:hclust, k=2). Assemble per-setting stability and concordance statistics into stability_summary.csv.

## Related tools

- **cola** (Runs the consensus partitioning with configurable row sampling (p), top-value method (MAD), partitioning method (hclust), and provides adjust_matrix() for preprocessing and 1-PAC stability metrics) — https://github.com/jokergoo/cola_manuscript
- **impute** (Provides impute.knn() used inside adjust_matrix() to impute missing values in the expression matrix)
- **bsub** (Submits the repeated cola consensus partitioning jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub

## Examples

```
mat = adjust_matrix(mat); rh = consensus_partition(mat, top_value_method = "MAD", partition_method = "hclust", k = 2)
```

## Evaluation signals

- Average SD of 1-PAC across 100 repetitions decreases monotonically with sampling size, approximating 0.0077, 0.0063, 0.0054, 0.0050 for 25/50/100/200 samplings
- Mean concordance between consensus partitions from 25-row and 200-row samplings is near-complete (0.973 vs 0.991 for MAD:hclust, k=2)
- stability_summary.csv exists and contains per-setting stability and concordance statistics for all four sampling sizes
- Input matrix was preprocessed with adjust_matrix() (missing values imputed, outliers adjusted, low-variance rows removed)
- Each setting used exactly 100 repetitions with the same top-value method (MAD) and partitioning method (hclust) at k=2

## Limitations

- Increasing the number of random row samplings only marginally improves partition stability, so extra compute may not be justified
- For very large datasets (e.g., thousands of single cells), cola runtime and memory usage may be unacceptable
- Downsampling strategies may miss small clusters/cell clusters with small sizes

## Evidence
