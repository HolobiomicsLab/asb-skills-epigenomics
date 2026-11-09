---
name: cross-dataset-method-comparison
description: Use when a choice among feature-selection (top-value) or partitioning
  methods for consensus clustering must be justified by large-scale empirical comparison
  across many datasets rather than a single example.
when_to_use_negative:
- You need to partition a single dataset and just pick a method, with no cross-dataset
  benchmarking required — use cola's per-dataset comparison of method results instead
- The data are scRNA-Seq with several thousands of cells — the article notes cola's
  runtime and memory usage 'might not be acceptable' at that scale
- The input is already a partition or a feature table rather than a raw gene expression
  matrix — the preprocessing and partitioning steps here assume row-level expression
  matrices
tools:
- name: cola
  role: R/Bioconductor package providing the consensus partitioning framework, adjust_matrix()
    preprocessing, ATC top-value method, and 1-PAC stability statistic
  repo: https://github.com/jokergoo/cola_manuscript
- name: bsub
  role: Submits the 24-combination cola analyses as jobs to an LSF computing cluster
    and monitors their completion
  repo: https://github.com/jokergoo/bsub
- name: impute
  role: Supplies impute.knn() used by adjust_matrix() to impute missing values before
    benchmarking
- name: skmeans
  role: Spherical k-means partitioning method whose combinations are benchmarked against
    hclust and other partitioning methods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:8348bcc03fe56d7c928a5cefa662e9c222cc60fe9af10b16c33a870e08cc8390
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
  - skmeans
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-dataset-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks all 24 combinations of top-value methods (SD, MAD, CV, ATC) and partitioning methods across 435 public gene expression datasets (206 GDS microarray + 223 recount2 RNA-Seq), scoring each combination by the mean 1-PAC score at the best k to identify which method combinations (notably ATC and skmeans) yield the most stable partitions.

## When to use

Use when a choice among feature-selection (top-value) or partitioning methods for consensus clustering must be justified by large-scale empirical comparison across many datasets rather than a single example.

## When NOT to use

- You need to partition a single dataset and just pick a method, with no cross-dataset benchmarking required — use cola's per-dataset comparison of method results instead
- The data are scRNA-Seq with several thousands of cells — the article notes cola's runtime and memory usage 'might not be acceptable' at that scale
- The input is already a partition or a feature table rather than a raw gene expression matrix — the preprocessing and partitioning steps here assume row-level expression matrices

## Inputs

- 206 GDS microarray expression matrices downloaded via GEOquery (GDS1761 excluded)
- 223 recount2 RNA-Seq expression matrices downloaded from the recount2 website
- R/Bioconductor environment with cola, impute, skmeans, and bsub installed

## Outputs

- Per-dataset, per-method-combination 1-PAC scores at the best k (k = 2–6)
- Aggregated benchmark table (e.g. benchmark_1pac_summary.csv) with mean 1-PAC per method combination and proportion of datasets with 1-PAC > 0.9
- HTML cola reports for the 435 datasets (available at https://jokergoo.github.io/cola_collection/)

## How to apply

Preprocess every expression matrix with cola's adjust_matrix(), which imputes missing values (impute.knn from the R package impute), winsorizes outliers at the 5th/95th percentiles, and removes low-variance rows. Run cola's consensus partitioning for each dataset across all 24 combinations of top-value methods (SD, MAD, CV, ATC) and partitioning methods, testing k = 2 to 6, submitting jobs to an LSF cluster via the bsub package. For each dataset and combination, extract the 1-PAC score at the best k, then aggregate per combination to compute the mean 1-PAC score and the proportion of datasets with stable partitions (1-PAC > 0.9). Compare combinations containing ATC and/or skmeans against hclust-containing combinations; the expected outcome is that ATC and/or skmeans combinations attain higher 1-PAC scores and a higher proportion of stable partitions, while hclust combinations yield fewer stable partitions. Write the aggregated per-comparison table (e.g. benchmark_1pac_summary.csv) as the deliverable.

## Related tools

- **cola** (R/Bioconductor package providing the consensus partitioning framework, adjust_matrix() preprocessing, ATC top-value method, and 1-PAC stability statistic) — https://github.com/jokergoo/cola_manuscript
- **bsub** (Submits the 24-combination cola analyses as jobs to an LSF computing cluster and monitors their completion) — https://github.com/jokergoo/bsub
- **impute** (Supplies impute.knn() used by adjust_matrix() to impute missing values before benchmarking)
- **skmeans** (Spherical k-means partitioning method whose combinations are benchmarked against hclust and other partitioning methods)

## Examples

```
mat = adjust_matrix(mat)  # impute missing values, adjust outliers, remove low-variance rows before consensus partitioning
```

## Evaluation signals

- All 24 top-value × partitioning method combinations are represented in the summary table, and 435 datasets total (206 GDS + 223 recount2) are covered
- Combinations containing ATC and/or skmeans show higher mean 1-PAC scores at the best k than hclust-containing combinations
- Combinations containing ATC and/or skmeans show a higher proportion of datasets with stable partitions (1-PAC > 0.9)
- k values tested span 2 to 6 for every dataset × combination cell
- GDS1761 is absent from the aggregated results (removed due to a GEOquery error)

## Limitations

- GDS1761 was removed from the analysis due to an error of GEOquery, so the GDS benchmark covers 206 rather than 207 datasets
- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies in consensus partitioning may fail to detect small cell clusters

## Evidence
