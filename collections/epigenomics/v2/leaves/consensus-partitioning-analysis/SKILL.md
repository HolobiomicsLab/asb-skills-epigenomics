---
name: consensus-partitioning-analysis
description: Use when a high-dimensional genomic data matrix (gene expression, methylation
  probes, or scRNA-Seq) must be classified into subgroups and no known labels exist,
  i.e. unsupervised consensus clustering with stability assessment is the goal.
when_to_use_negative:
- Single-cell datasets with several thousands of cells, where cola's runtime and memory
  usage may be unacceptable and downsampling may miss small cell clusters
- When a trusted sample classification already exists and only differential analysis
  is needed — consensus partitioning is unsupervised and does not use known labels
- When rows are already a small curated feature set — ATC/SD top-value feature selection
  is designed for high-dimensional matrices with many features
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: cola
  role: Core R/Bioconductor package providing the consensus partitioning framework,
    adjust_matrix(), top-value methods (SD, MAD, CV, ATC), run_all_consensus_partitions(),
    and HTML report generation
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: Provides impute.knn() for k-nearest-neighbour imputation of missing values
    during matrix preprocessing
- name: NMF
  role: R package used to perform NMF as one of the six partitioning methods in the
    consensus framework
- name: bsub
  role: Submits cola analysis jobs to an LSF computing cluster directly from R
  repo: https://github.com/jokergoo/bsub
- name: clue (cl_consensus)
  role: Infers consensus subgroup labels from the individual (NP) partitions
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:be1eb50804b25fefc7bceff6786c010c02907bc8f3d50d9b424a2e07db75a41f
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
  - clue (cl_consensus)
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# consensus-partitioning-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply consensus partitioning via the cola R/Bioconductor framework to discover molecular subgroups in high-throughput genomic matrices, combining feature selection (top-value methods SD, MAD, CV, ATC) with repeated random sampling and multiple partitioning methods, and validating stability with the 1-PAC score.

## When to use

Use when a high-dimensional genomic data matrix (gene expression, methylation probes, or scRNA-Seq) must be classified into subgroups and no known labels exist, i.e. unsupervised consensus clustering with stability assessment is the goal.

## When NOT to use

- Single-cell datasets with several thousands of cells, where cola's runtime and memory usage may be unacceptable and downsampling may miss small cell clusters
- When a trusted sample classification already exists and only differential analysis is needed — consensus partitioning is unsupervised and does not use known labels
- When rows are already a small curated feature set — ATC/SD top-value feature selection is designed for high-dimensional matrices with many features

## Inputs

- Preprocessed numeric genomic data matrix (rows = genes/probes/features, columns = samples), e.g. TCGA GBM expression matrix from https://gdc.cancer.gov/about-data/publications/gbm_exp
- Expression datasets from Bioconductor packages (golubEsets Golub leukemia, ALL Ritz dataset, HSMMSingleCell HSMM)
- Methylation 450K matrix from GEO (GSE36278)
- Parameter settings: top-value method, partitioning method, k range, number of samplings/repetitions

## Outputs

- Consensus subgroup partitions per method/k combination with 1-PAC stability scores
- Signature gene lists per subgroup (with silhouette-based sample filtering)
- cola result objects (e.g. golub_cola_results.rds)
- Automated comprehensive HTML report and comparison tables (e.g. CSV files of 1-PAC and concordance statistics)

## How to apply

Preprocess the input matrix with cola's adjust_matrix(): impute missing values with impute.knn() from the R package impute, winsorize values at the 5th/95th percentiles, and remove rows with zero variance or variance below the 5th percentile. Score rows with top-value methods (SD, MAD, CV, or the correlation-based ATC method) and select the top n rows for partitioning; scale rows (z-score or min-max) before partitioning to remove absolute level differences. Run run_all_consensus_partitions() (or consensus_partition() in a loop) over combinations of top-value methods and partitioning methods (hclust, kmeans, skmeans, pam, mclust, NMF), for a range of k (e.g. 2–6) with repeated random row/column sampling (e.g. 100 repetitions, sampling probability p); consensus labels are inferred with cl_consensus() from the R package clue. Assess stability with the 1-PAC score per method combination and k, extract signature genes (removing samples with silhouette scores < 0.5 by default), and compare partitions across methods or sampling schemes via concordance; ATC and skmeans are recommended based on benchmarks of 435 public datasets.

## Related tools

- **cola** (Core R/Bioconductor package providing the consensus partitioning framework, adjust_matrix(), top-value methods (SD, MAD, CV, ATC), run_all_consensus_partitions(), and HTML report generation) — https://github.com/jokergoo/cola_manuscript
- **impute** (Provides impute.knn() for k-nearest-neighbour imputation of missing values during matrix preprocessing)
- **NMF** (R package used to perform NMF as one of the six partitioning methods in the consensus framework)
- **bsub** (Submits cola analysis jobs to an LSF computing cluster directly from R) — https://github.com/jokergoo/bsub
- **clue (cl_consensus)** (Infers consensus subgroup labels from the individual (NP) partitions)

## Examples

```
library(cola); mat = adjust_matrix(mat); rl = run_all_consensus_partition_methods(mat, top_value_method = c("SD","ATC"), partition_method = c("hclust","skmeans"), k = 2:6); get_stat(rl)
```

## Evaluation signals

- Reproduced 1-PAC scores fall within [0, 1] and match reported values within a small tolerance, e.g. ATC:skmeans k=2 1-PAC = 1 and k=3 1-PAC = 0.96 on the Golub leukemia dataset
- ATC:skmeans two-group partition on Golub contains exactly 1810 signature genes
- Row-versus-column sampling comparison reproduces mean 1-PAC differences near 0.023 (SD:hclust), 0.019 (SD:skmeans), 0.037 (ATC:hclust), 0.046 (ATC:skmeans), and concordance ~0.939 for ATC:skmeans at k=3
- Increasing row-sampling size from 25 to 200 yields decreasing SD of 1-PAC (0.0077 → 0.0050) and near-agreement of stable partitions (concordance ~0.973–0.991)
- On the GBM 450K dataset, SD- and ATC-selected top-5000 probes overlap near 1.76%, with CpG-island probe fractions of 60.6% (SD) vs 6.6% (ATC), and SD-selected features recover the DKFZ subtype classification

## Limitations

- Runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling (random sampling) strategies may miss small cell clusters
- Merging/weighting consensus partitions across different methods is complex and can produce ambiguous labels
- Rows in which >25% of samples have missing values are removed, which can be limiting for sparse matrices

## Evidence

- [abstract] We implemented a new R/Bioconductor package, cola, that provides a general framework for consensus partitioning.: "We implemented a new R/Bioconductor package, cola, that provides a general framework for consensus partitioning."
