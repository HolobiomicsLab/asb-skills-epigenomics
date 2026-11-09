---
name: partition-concordance-evaluation
description: Use when choosing the number of resampling repetitions or the sampling
  type (rows vs columns) for cola consensus partitioning, or when verifying that a
  discovered subgroup partition is stable rather than an artifact of sampling.
when_to_use_negative:
- The partition is already stable and the goal is biological interpretation (signature
  analysis, functional enrichment) rather than resampling robustness.
- Single-cell datasets with several thousands of cells, where cola's runtime and memory
  usage may be unacceptable, making many repeated sampling settings infeasible.
- Detecting very small subgroups is the priority, since downsampling/resampling-based
  consensus may not detect small clusters.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: Provides consensus_partition()/run_all_consensus_partition_methods() to generate
    the partitions under each sampling setting, adjust_matrix() for preprocessing,
    and the 1-PAC score used as the stability metric
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Analysis environment in which the sampling comparisons, standard deviations
    of 1-PAC, and concordance statistics are computed and exported to CSV
- name: impute
  role: Supplies impute.knn() to fill missing values during matrix preprocessing so
    that resampling benchmarks run on complete data
- name: clue
  role: Its cl_consensus() infers consensus subgroup labels from individual partitions
    and partition labels can be adjusted between any two sets of partitions, enabling
    concordance computation
- name: bsub
  role: Submits the repeated cola consensus partitioning jobs to an LSF cluster when
    benchmarking many sampling settings in the manuscript scripts
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:8385c3aeb12c56fba5d0e44bb6066f3da7add0b50f09a05f701aef4ce233ae75
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# partition-concordance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for cola consensus partitions that quantifies stability (average standard deviation of the 1-PAC score across repetitions) and partition concordance while varying resampling settings — number of random samplings (25/50/100/200) and row- versus column-sampling — on high-dimensional expression matrices such as TCGA GBM.

## When to use

Use when choosing the number of resampling repetitions or the sampling type (rows vs columns) for cola consensus partitioning, or when verifying that a discovered subgroup partition is stable rather than an artifact of sampling.

## When NOT to use

- The partition is already stable and the goal is biological interpretation (signature analysis, functional enrichment) rather than resampling robustness.
- Single-cell datasets with several thousands of cells, where cola's runtime and memory usage may be unacceptable, making many repeated sampling settings infeasible.
- Detecting very small subgroups is the priority, since downsampling/resampling-based consensus may not detect small clusters.

## Inputs

- Preprocessed high-dimensional expression matrix (e.g. TCGA GBM from https://gdc.cancer.gov/about-data/publications/gbm_exp) passed through cola's adjust_matrix()
- Sampling settings to compare: number of random samplings (25, 50, 100, 200) and/or sampling dimension (rows vs columns)
- Method combinations and k values: top-value methods (MAD, SD, ATC) x partitioning methods (hclust, skmeans), k from 2 to 6, with 100 bootstrap repetitions per setting

## Outputs

- Per-setting stability statistics: average standard deviation of the 1-PAC score across the 100 repetitions for each sampling size
- Mean partition concordance between settings (e.g. 25 vs 200 samplings; row- vs column-sampled partitions per top-value:partitioning:k combination)
- Summary table written to CSV (e.g. stability_summary.csv, gbm_row_column_sampling_comparison.csv)

## How to apply

First preprocess the expression matrix with cola's adjust_matrix(), which imputes missing values with impute.knn(), adjusts outliers (winsorizing at the 5th/95th percentiles) and removes rows with very small variance, so stability comparisons reflect sampling effects rather than data-quality artifacts. Then run consensus partitioning repeatedly per setting — e.g. MAD top values with hclust at k=2 under 25, 50, 100 and 200 random row samplings (100 repetitions each), or SD/ATC top values with hclust/skmeans at k=2-6 comparing row- versus column-sampling. For each setting compute the average standard deviation of the 1-PAC score across the 100 repetitions as the stability metric; on TCGA GBM it decreases only marginally (0.0077, 0.0063, 0.0054, 0.0050 for 25/50/100/200 samplings), showing that modest sampling counts suffice. Next, adjust partition labels between settings and compute mean concordance between partitions from different settings, e.g. 25 vs 200 samplings (mean concordance 0.973 vs 0.991 for MAD:hclust, k=2) or row vs column sampling (approximately 0.939 for ATC:skmeans at k=3). Expect row sampling to show a small but consistent 1-PAC advantage over column sampling (mean differences of about 0.023, 0.019, 0.037, 0.046 for SD:hclust, SD:skmeans, ATC:hclust, ATC:skmeans). Finally, assemble the per-setting stability and concordance statistics into a summary table (e.g. stability_summary.csv or gbm_row_column_sampling_comparison.csv).

## Related tools

- **cola** (Provides consensus_partition()/run_all_consensus_partition_methods() to generate the partitions under each sampling setting, adjust_matrix() for preprocessing, and the 1-PAC score used as the stability metric) — https://github.com/jokergoo/cola_manuscript
- **R** (Analysis environment in which the sampling comparisons, standard deviations of 1-PAC, and concordance statistics are computed and exported to CSV)
- **impute** (Supplies impute.knn() to fill missing values during matrix preprocessing so that resampling benchmarks run on complete data)
- **clue** (Its cl_consensus() infers consensus subgroup labels from individual partitions and partition labels can be adjusted between any two sets of partitions, enabling concordance computation)
- **bsub** (Submits the repeated cola consensus partitioning jobs to an LSF cluster when benchmarking many sampling settings in the manuscript scripts) — https://github.com/jokergoo/bsub

## Examples

```
library(cola); mat = adjust_matrix(mat); rl = consensus_partition(mat, top_value_method = 'MAD', partition_method = 'hclust', max_k = 2)
```

## Evaluation signals

- Average SD of 1-PAC across 100 repetitions should decrease as samplings increase, near 0.0077, 0.0063, 0.0054, 0.0050 for 25/50/100/200 row samplings (MAD:hclust, k=2, TCGA GBM).
- Mean concordance between partitions from 25 vs 200 samplings should indicate near-complete agreement (0.973 vs 0.991 for MAD:hclust, k=2).
- Row vs column sampling 1-PAC differences should be small but consistently positive: approximately 0.023 (SD:hclust), 0.019 (SD:skmeans), 0.037 (ATC:hclust), 0.046 (ATC:skmeans).
- Row- and column-sampled partitions should show high agreement, e.g. mean concordance approximately 0.939 for ATC:skmeans at k=3.
- Output CSV must contain one entry per top-value:partitioning:k:sampling setting with both the stability (SD of 1-PAC) and concordance columns populated.

## Limitations

- Increasing the number of random samplings only marginally improves stability (average SD of 1-PAC drops from 0.0077 at 25 to 0.0050 at 200), so large resampling budgets buy little.
- Downsampling-based consensus can miss small subgroups: the cell clusters with small sizes might not be detectable.
- For single-cell datasets with several thousands of cells, cola's runtime and memory usage might not be acceptable, constraining how many sampling settings can be benchmarked.

## Evidence
