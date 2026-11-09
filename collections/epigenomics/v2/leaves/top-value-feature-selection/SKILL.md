---
name: top-value-feature-selection
description: Use when preparing a gene-expression or methylation matrix (thousands
  of rows) for cola consensus partitioning and a feature-selection step is needed
  to pick rows that drive subgroup discovery, especially when comparing SD/MAD/CV
  against ATC.
when_to_use_negative:
- The input is already a low-dimensional or pre-filtered feature table where a top-n
  row selection would discard informative features
- The goal is single-cell clustering with thousands of cells, where cola's runtime/memory
  may be unacceptable and downsampling may miss small clusters
- Rows have not been preprocessed (missing values imputed, outliers adjusted, near-zero-variance
  rows removed), which would distort top-value scores
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: cola
  role: Provides the top-value methods (SD, MAD, CV, ATC), the adjust_matrix() preprocessing,
    and the consensus partitioning framework that consumes the selected features
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: Supplies impute.knn() used to fill missing values before row scores are computed
- name: bsub
  role: Used to submit the 24 method-combination consensus partitioning jobs to an
    LSF cluster during large-scale benchmarking
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:dbdfd8c054ac50755313263730a31e2f6a9a15678e915bba197353135ccc7e09
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
  - NMF
  - R
  - bsub
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# top-value-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects informative features (rows) for consensus partitioning in high-dimensional genomic data by scoring all rows with a top-value method — SD, MAD, CV, or cola's ATC (ability to correlate to other rows) — and retaining only the top-scoring rows before partitioning.

## When to use

Use when preparing a gene-expression or methylation matrix (thousands of rows) for cola consensus partitioning and a feature-selection step is needed to pick rows that drive subgroup discovery, especially when comparing SD/MAD/CV against ATC.

## When NOT to use

- The input is already a low-dimensional or pre-filtered feature table where a top-n row selection would discard informative features
- The goal is single-cell clustering with thousands of cells, where cola's runtime/memory may be unacceptable and downsampling may miss small clusters
- Rows have not been preprocessed (missing values imputed, outliers adjusted, near-zero-variance rows removed), which would distort top-value scores

## Inputs

- High-dimensional expression or methylation matrix (genes or CpG probes as rows, samples as columns), e.g. Golub leukemia data from golubEsets, TCGA GBM matrix, recount2 RNA-Seq matrices
- Choice of top-value method: SD, MAD, CV, or ATC
- Number of top rows n to retain for partitioning

## Outputs

- Ranked row scores per top-value method
- Filtered/scaled feature matrix passed to consensus partitioning
- Per-method-combination stability statistics (1-PAC scores for k = 2..6) for comparing top-value methods

## How to apply

First preprocess the matrix with cola's adjust_matrix(), which imputes missing values (e.g. impute.knn() from the R package impute), adjusts outliers, and removes rows with very small variance. Then compute a top-value score for every row — SD, MAD, CV, or ATC, which cola introduces to select features based on the global correlation structure of rows — order rows by score, and keep only the top n rows with the highest scores. Before partitioning, scale the selected rows (z-score or min-max) to remove differences of absolute levels between rows. For benchmarking, run all 24 combinations of the four top-value methods and six partitioning methods (hclust, kmeans, skmeans, pam, mclust, NMF) for k = 2 to 6, as done on 206 GDS microarray and 223 recount2 RNA-Seq datasets, then compare 1-PAC stability scores; combinations with ATC and/or skmeans are expected to yield more stable partitions than those with hclust.

## Related tools

- **cola** (Provides the top-value methods (SD, MAD, CV, ATC), the adjust_matrix() preprocessing, and the consensus partitioning framework that consumes the selected features) — https://github.com/jokergoo/cola_manuscript
- **impute** (Supplies impute.knn() used to fill missing values before row scores are computed)
- **bsub** (Used to submit the 24 method-combination consensus partitioning jobs to an LSF cluster during large-scale benchmarking) — https://github.com/jokergoo/bsub

## Examples

```
mat = adjust_matrix(mat); rl = run_all_consensus_partition_methods(mat, top_value_method = c("SD","MAD","CV","ATC"), partition_method = c("hclust","kmeans","skmeans","pam","mclust","NMF"), k = 2:6)
```

## Evaluation signals

- 1-PAC scores for the chosen top-value:partitioning combinations fall in [0, 1] and match reported values (e.g. Golub k=2 ATC:kmeans 1-PAC = 1, ATC:NMF = 0.97, ATC:skmeans = 1)
- In benchmarking, combinations containing ATC and/or skmeans attain higher mean 1-PAC and a higher proportion of datasets with 1-PAC > 0.9 at the best k than hclust combinations
- Rows with >25% missing values were removed and remaining missing values imputed before scoring; rows with zero variance or variance below the 5th percentile of all row variances were removed
- Selected top n rows are ordered strictly by the top-value score and scaled (z-score or min-max) before any partitioning method is applied

## Limitations

- Downsampling/row-sampling strategies may miss small subgroups; cell clusters with small sizes might not be detectable
- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- GDS1761 was removed from the GDS benchmark due to a GEOquery error, showing ingestion failures can drop datasets

## Evidence

- [results] Methods with ATC and/or skmeans generated partitions with higher 1-PAC scores for the best k, while methods with hclust generated low 1-PAC scores: [evidence span withheld — anchor status: not_found]
