---
name: row-vs-column-sampling-comparison
description: Use when running cola consensus partitioning on a high-dimensional matrix
  and you need to decide whether to resample rows or columns during bootstrapping,
  or to verify that both sampling types yield comparable subgroupings.
when_to_use_negative:
- The dataset is single-cell RNA-Seq with several thousands of cells, where cola runtime
  and memory usage may be unacceptable and downsampling may miss small cell clusters
- You only need a single consensus partition for one sampling scheme rather than an
  explicit row-versus-column sampling stability comparison
- The input is a methylation matrix needing probe-level filters (sex chromosomes,
  SNP-overlapping probes) that this GBM expression workflow does not include
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: cola
  role: Runs consensus partitioning with SD/ATC top-value methods, hclust/skmeans
    partitioning, and row/column resampling; provides adjust_matrix() preprocessing
    and PAC-based stability scoring
  repo: https://github.com/jokergoo/cola_manuscript
- name: R
  role: Implementation environment for the comparison workflow and CSV output
- name: Bioconductor
  role: Distribution framework for cola and impute packages
- name: impute
  role: Provides impute.knn() for imputing missing values before partitioning
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:8414b51a30c097f972cbe033f6d51a06adfd904dea1dd0f24278709973673e1a
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
  - Bioconductor
  - R
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# row-vs-column-sampling-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A cola-based benchmark that contrasts randomly sampling rows (features) versus columns (samples) during consensus partitioning, quantifying stability via 1-PAC and cross-sampling-type concordance. It tests whether the bootstrap resampling dimension affects subgroup stability on high-dimensional gene expression data such as TCGA GBM.

## When to use

Use when running cola consensus partitioning on a high-dimensional matrix and you need to decide whether to resample rows or columns during bootstrapping, or to verify that both sampling types yield comparable subgroupings.

## When NOT to use

- The dataset is single-cell RNA-Seq with several thousands of cells, where cola runtime and memory usage may be unacceptable and downsampling may miss small cell clusters
- You only need a single consensus partition for one sampling scheme rather than an explicit row-versus-column sampling stability comparison
- The input is a methylation matrix needing probe-level filters (sex chromosomes, SNP-overlapping probes) that this GBM expression workflow does not include

## Inputs

- Preprocessed TCGA GBM gene expression matrix from https://gdc.cancer.gov/about-data/publications/gbm_exp
- Raw expression matrix requiring cola adjust_matrix() preprocessing (impute.knn imputation, 5th/95th percentile winsorization, low/zero-variance row removal)

## Outputs

- gbm_row_column_sampling_comparison.csv with mean 1-PAC differences (row vs column sampling) per top-value/partitioning combination and concordance values
- Mean 1-PAC scores per top-value method (SD, ATC), partitioning method (hclust, skmeans), sampling type, and k (2–6)
- Mean concordance between row-sampled and column-sampled partitions (e.g., ATC:skmeans at k=3)

## How to apply

Load the preprocessed TCGA GBM expression matrix (https://gdc.cancer.gov/about-data/publications/gbm_exp) and apply cola's adjust_matrix() preprocessing: impute missing values with impute.knn(), winsorize values at the 5th/95th percentiles, and remove rows with zero variance or variance below the 5th percentile of all row variances. Run consensus partitioning with the SD and ATC top-value methods combined with hclust and skmeans partitioning, k from 2 to 6, 100 bootstrap repetitions, once sampling rows and once sampling columns. Compute the 1-PAC score per top-value/partitioning/sampling/k combination and average over repetitions, then take the mean 1-PAC difference (row minus column) per method combination — expect approximately 0.023 for SD:hclust, 0.019 for SD:skmeans, 0.037 for ATC:hclust, and 0.046 for ATC:skmeans. Also compute mean concordance between row-sampled and column-sampled partitions (expect ~0.939 for ATC:skmeans at k=3) and write the comparison table to gbm_row_column_sampling_comparison.csv. The rationale is that cola samples 'a subset of rows or columns... with a probability of p' during resampling, so this skill empirically checks which sampling dimension gives more stable consensus partitions and whether the two agree.

## Related tools

- **cola** (Runs consensus partitioning with SD/ATC top-value methods, hclust/skmeans partitioning, and row/column resampling; provides adjust_matrix() preprocessing and PAC-based stability scoring) — https://github.com/jokergoo/cola_manuscript
- **R** (Implementation environment for the comparison workflow and CSV output)
- **Bioconductor** (Distribution framework for cola and impute packages)
- **impute** (Provides impute.knn() for imputing missing values before partitioning)

## Examples

```
library(cola); mat = adjust_matrix(readRDS('gbm_exp.rds')); rl = run_all_consensus_partition_methods(mat, top_value_method = c('SD','ATC'), partition_method = c('hclust','skmeans'), k = 2:6, nrepeat = 100)
```

## Evaluation signals

- Mean 1-PAC differences (row minus column sampling) are close to reported values: 0.023 for SD:hclust, 0.019 for SD:skmeans, 0.037 for ATC:hclust, 0.046 for ATC:skmeans
- All four method combinations show a positive (row > column) 1-PAC advantage, confirming a small but consistent advantage for row sampling
- Mean concordance between row-sampled and column-sampled partitions for ATC:skmeans at k=3 is approximately 0.939, indicating high agreement
- adjust_matrix() preprocessing verified: no missing values remain after impute.knn(), outliers winsorized at 5th/95th percentiles, zero/low-variance rows removed
- Output file gbm_row_column_sampling_comparison.csv contains mean 1-PAC differences and concordance values for all combinations

## Limitations

- Row sampling's 1-PAC advantage is small (0.019–0.046), so the difference may not be significant for every dataset
- Downsampling strategies may miss small clusters ('the cell clusters with small sizes might not be detectable')
- cola runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- No version number for cola or R is stated in the source, limiting reproducibility of versioned results

## Evidence

- [full_text] Row sampling generated higher 1-PAC scores than column sampling: [evidence span withheld — anchor status: not_found]
