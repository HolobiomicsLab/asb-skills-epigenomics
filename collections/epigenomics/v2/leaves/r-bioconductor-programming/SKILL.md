---
name: r-bioconductor-programming
description: Use when an analysis must be scripted in R with Bioconductor packages
  — e.g. loading golubEsets expression matrices, imputing with impute.knn(), or customizing
  cola's consensus partitioning framework.
when_to_use_negative:
- The analysis is already available as a ready-made pipeline outside R (e.g. Python
  tooling) and no user-definable methods from cola are needed
- Single-cell datasets with several thousands of cells where cola's runtime and memory
  usage may be unacceptable without downsampling
- The task only requires shallow statistics without a feature-selection/consensus-partitioning
  workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: R/Bioconductor package providing the general framework for consensus partitioning,
    feature selection (ATC), skmeans clustering and HTML reports
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: Provides impute.knn() to impute missing values during matrix preprocessing
- name: bsub
  role: R package to submit R code/scripts/shell commands to LSF clusters when running
    cola analyses at scale
  repo: https://github.com/jokergoo/bsub
- name: clue
  role: Supplies cl_consensus() to infer consensus subgroup labels from individual
    partitions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:cdffcb77f258ae6844d621872280605138d9725af32dea02238559a0822d91be
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
  - bsub
  - clue
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# r-bioconductor-programming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement and run scientific analyses in R using the R/Bioconductor ecosystem, as exemplified by the cola package, which formalizes consensus partitioning with user-definable methods (e.g. ATC feature scoring, skmeans clustering) and automated HTML reporting.

## When to use

Use when an analysis must be scripted in R with Bioconductor packages — e.g. loading golubEsets expression matrices, imputing with impute.knn(), or customizing cola's consensus partitioning framework.

## When NOT to use

- The analysis is already available as a ready-made pipeline outside R (e.g. Python tooling) and no user-definable methods from cola are needed
- Single-cell datasets with several thousands of cells where cola's runtime and memory usage may be unacceptable without downsampling
- The task only requires shallow statistics without a feature-selection/consensus-partitioning workflow

## Inputs

- Numeric matrix (e.g. gene expression matrix from the golubEsets Bioconductor package) with rows as features and columns as samples
- Genomic datasets such as TCGA GBM, Golub leukemia (golubEsets), Ritz ALL (ALL package), HSMM single-cell (HSMMSingleCell), recount2 RNA-Seq, and GEO accession GSE36278
- R scripts or R code chunks for submission to an LSF cluster via bsub

## Outputs

- Per-row feature scores as a two-column table (feature name, ATC score) written to atc_scores.csv
- Consensus partition / subgroup labels inferred from individual partitions
- Comprehensive automated HTML report of the analysis

## How to apply

Install required packages from CRAN or Bioconductor (e.g. install.packages('bsub'), devtools::install_github('jokergoo/bsub')) and load the preprocessed numeric matrix with rows as features and columns as samples. Follow cola's pipeline: use adjust_matrix() to impute missing values, adjust outliers and remove low-variance rows; apply a top-value method such as ATC, where ATC_i = 1 - integral of the empirical CDF of row i's absolute correlations to all other rows, then select the top n rows by score. Scale rows (z-score or min-max) before partitioning, run the chosen partitioning method (cola recommends spherical k-means, skmeans) over random row/column subsets, and infer consensus labels with cl_consensus() from the R clue package. For cluster-scale runs, submit R chunks or scripts with the bsub package to an LSF cluster (bsub_chunk, bsub_script, bsub_cmd, bkill, bjobs). Persist intermediate results (e.g. saveRDS to permanent files) and package the analysis as reproducible scripts, as done in the jokergoo/cola_manuscript repository.

## Related tools

- **cola** (R/Bioconductor package providing the general framework for consensus partitioning, feature selection (ATC), skmeans clustering and HTML reports) — https://github.com/jokergoo/cola_manuscript
- **impute** (Provides impute.knn() to impute missing values during matrix preprocessing)
- **bsub** (R package to submit R code/scripts/shell commands to LSF clusters when running cola analyses at scale) — https://github.com/jokergoo/bsub
- **clue** (Supplies cl_consensus() to infer consensus subgroup labels from individual partitions)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("jokergoo/bsub")
```

## Evaluation signals

- R scripts run end-to-end without errors and required packages load (library checks pass)
- atc_scores.csv exists, contains exactly one score per input row, and every ATC value lies within [0, 1]
- Preprocessing invariants hold: rows with >25% missing values removed, missing values imputed via impute.knn(), zero-variance and <5th-percentile-variance rows removed
- Consensus partition labels are reproducible and consistent with the individual partitions generated by repeated sampling
- Outputs (HTML report, saved RDS objects) are persisted to permanent files as recommended in the bsub workflow

## Limitations

- cola's runtime and memory usage may be unacceptable for scRNA-Seq datasets with several thousands of cells
- Downsampling strategies may miss small cell clusters
- No changelog available in the manuscript metadata, limiting reproducibility of versioned results

## Evidence
