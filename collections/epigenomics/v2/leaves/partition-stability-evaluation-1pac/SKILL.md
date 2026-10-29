---
name: partition-stability-evaluation-1pac
description: Use when consensus clustering has been run with cola (or an equivalent
  framework) and you must quantify which top-value/partitioning method combination
  and k yields the most stable sample partitions.
when_to_use_negative:
- Input is single-cell RNA-Seq with several thousands of cells — the paper notes cola's
  runtime and memory usage may be unacceptable at that scale
- You already have a validated subgroup assignment and only need differential expression/signature
  analysis rather than stability-based method selection
- Small cell clusters are the target — downsampling in cola's consensus procedure
  may miss clusters of small size
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: cola
  role: Runs consensus partitioning and provides the 1-PAC stability statistic via
    get_stats() across top-value/partitioning combinations
  repo: https://jokergoo.github.io/cola_collection/
- name: R
  role: Implementation language for running cola and aggregating 1-PAC scores
- name: Bioconductor
  role: Source of input datasets (golubEsets, ALL, HSMMSingleCell) and cola distribution
- name: bsub
  role: Submits cola consensus partitioning jobs for each dataset/combination to the
    LSF cluster
  repo: https://github.com/jokergoo/bsub
- name: impute
  role: impute.knn() fills missing values before stability evaluation
- name: NMF
  role: Provides the NMF partitioning method evaluated in the 24-combination benchmark
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:d7769d14526ab81cdd5c38eafdef8ed4e78e69b3272b74516ea361dfe9eadfc8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jokergoo/cola_manuscript
  tools:
  - Bioconductor
  - NMF
  - R
  - bsub
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# partition-stability-evaluation-1pac

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate consensus partition stability in cola using the 1-PAC score (1 minus proportion of ambiguous clustering) across method combinations of top-value methods (SD, MAD, CV, ATC) and partitioning methods (hclust, kmeans, skmeans, pam, mclust, NMF), for k = 2–6.

## When to use

Use when consensus clustering has been run with cola (or an equivalent framework) and you must quantify which top-value/partitioning method combination and best k yields the most stable sample partitions.

## When NOT to use

- Input is single-cell RNA-Seq with several thousands of cells — the paper notes cola's runtime and memory usage may be unacceptable at that scale
- You already have a validated subgroup assignment and only need differential expression/signature analysis rather than stability-based method selection
- Small cell clusters are the target — downsampling in cola's consensus procedure may miss clusters of small size

## Inputs

- Preprocessed expression matrix (GDS microarray or recount2 RNA-Seq) processed with cola's adjust_matrix()
- cola result object from run_all_consensus_partitions covering 4 top-value × 6 partitioning methods, k = 2–6
- Benchmark scale: 206 GDS microarray matrices and 223 recount2 RNA-Seq matrices (recount2 downloaded from https://jhubiostatistics.shinyapps.io/recount/)

## Outputs

- Per-combination, per-k 1-PAC stability scores (values in [0, 1])
- Aggregated comparison table (e.g. benchmark_1pac_summary.csv) of mean 1-PAC and proportion of datasets with 1-PAC > 0.9 per method combination
- Identification of the best k per combination based on maximal 1-PAC

## How to apply

Run cola's consensus partitioning (e.g. run_all_consensus_partitions) over all 24 combinations of top-value methods (SD, MAD, CV, ATC) and partitioning methods (hclust, kmeans, skmeans, pam, mclust, NMF) for k = 2 to 6, on a matrix preprocessed with adjust_matrix() (impute missing values, adjust outliers, remove low-variance rows). For each combination, extract the 1-PAC score at the best k, where higher 1-PAC indicates a more stable consensus partition; 1-PAC > 0.9 is treated as a stable partition in the benchmark. Rationale: the cola paper shows on 435 public gene expression datasets (206 GDS microarray + 223 recount2 RNA-Seq) that combinations containing ATC and/or skmeans yield higher 1-PAC scores at the best k, while hclust-containing combinations yield low 1-PAC scores. Aggregate per-combination mean 1-PAC and the proportion of datasets with 1-PAC > 0.9 (e.g. to benchmark_1pac_summary.csv) to rank method combinations. Use the bsub package to submit each dataset/combination job to an LSF cluster for large-scale runs.

## Related tools

- **cola** (Runs consensus partitioning and provides the 1-PAC stability statistic via get_stats() across top-value/partitioning combinations) — https://jokergoo.github.io/cola_collection/
- **R** (Implementation language for running cola and aggregating 1-PAC scores)
- **Bioconductor** (Source of input datasets (golubEsets, ALL, HSMMSingleCell) and cola distribution)
- **bsub** (Submits cola consensus partitioning jobs for each dataset/combination to the LSF cluster) — https://github.com/jokergoo/bsub
- **impute** (impute.knn() fills missing values before stability evaluation)
- **NMF** (Provides the NMF partitioning method evaluated in the 24-combination benchmark)

## Examples

```
library(cola); rl = run_all_consensus_partitions(mat, top_value_method = "ATC", partition_method = "skmeans", k = 2:6); stats = get_stats(rl)
```

## Evaluation signals

- All 1-PAC values lie in [0, 1]; check no value falls outside this range
- Combinations containing ATC and/or skmeans attain higher 1-PAC scores at the best k than hclust-containing combinations, matching the reported benchmark finding
- On the Golub leukemia dataset, reproduced values match: k=2 ATC:kmeans 1-PAC = 1, ATC:NMF = 0.97, ATC:skmeans = 1; k=3 ATC:skmeans = 0.96 (within small numeric tolerance)
- A stable partition is flagged when 1-PAC > 0.9; verify the per-combination proportion of datasets exceeding this threshold is computed consistently
- Aggregated summary table (benchmark_1pac_summary.csv) contains one row per method combination (24 total) with mean 1-PAC and stable-partition proportion

## Limitations

- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies in consensus partitioning may miss small cell clusters, which can bias stability estimates
- GDS1761 was removed from the analysis due to an error of GEOquery — dataset-loading failures must be handled when benchmarking
- Weighting consensus partitions across different methods to merge classifications is complex and can produce ambiguous labels

## Evidence
