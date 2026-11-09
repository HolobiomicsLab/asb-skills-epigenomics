---
name: expression-matrix-preprocessing
description: Use when a raw numeric expression or methylation matrix (e.g., from golubEsets,
  ALL, HSMMSingleCell) must be cleaned before cola's feature selection and consensus
  partitioning.
when_to_use_negative:
- The matrix has already been through adjust_matrix() — re-applying would impute a
  second time and re-filter rows against shifted variance percentiles.
- You need dataset-specific filtering rules instead of the generic cleaning defaults,
  e.g. MCF10CA cells with <2 million reads, recount2 genes with <2 reads in >50% of
  samples, or methylation probes on sex chromosomes/overlapping known SNPs — those
  are separate per-dataset filter steps.
- You want to remove ambiguous samples (silhouette score < 0.5); that removal belongs
  to signature analysis, not to input-matrix preprocessing.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: cola
  role: Provides adjust_matrix(), the sequential imputation/outlier-adjustment/variance-filtering
    function applied to the input matrix
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: R package whose impute.knn() function fills remaining missing values after
    rows with >25% missing samples are removed
- name: R
  role: Runtime environment (R/Bioconductor) in which adjust_matrix() and impute.knn()
    are executed on the numeric matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:00ee4cdc949e21bbe952747929b4abc2ad90ee6b3235acc9557ade76483ed257
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
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# expression-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cleans a numeric gene-expression matrix via cola's adjust_matrix() before consensus partitioning: removes rows with >25% missing samples, imputes missing values with impute.knn(), winsorizes values at the 5th/95th percentiles, and removes rows with zero variance or variance below the 5th percentile of all row variances.

## When to use

Use when a raw numeric expression or methylation matrix (e.g., from golubEsets, ALL, HSMMSingleCell) must be cleaned before cola's feature selection and consensus partitioning.

## When NOT to use

- The matrix has already been through adjust_matrix() — re-applying would impute a second time and re-filter rows against shifted variance percentiles.
- You need dataset-specific filtering rules instead of the generic cleaning defaults, e.g. MCF10CA cells with <2 million reads, recount2 genes with <2 reads in >50% of samples, or methylation probes on sex chromosomes/overlapping known SNPs — those are separate per-dataset filter steps.
- You want to remove ambiguous samples (silhouette score < 0.5); that removal belongs to signature analysis, not to input-matrix preprocessing.

## Inputs

- numeric gene expression or methylation matrix (rows = genes/CpG probes, columns = samples), e.g. from golubEsets, ALL, HSMMSingleCell, TCGA GBM, GSE36278 or recount2 datasets
- matrix containing missing (NA) values and possible extreme outlier values

## Outputs

- cleaned numeric matrix with missing values imputed, outliers winsorized at the 5th/95th percentiles, and low-variance/zero-variance rows removed (e.g. preprocessed_matrix.csv)

## How to apply

Load the expression data as a numeric matrix and run the sequential adjust_matrix() procedure in cola. First remove rows in which more than 25% of the samples have missing values; then impute any remaining missing values with impute.knn() from the R package impute. Next adjust outliers by replacing values larger than the 95th percentile or smaller than the 5th percentile with the corresponding percentile values. Finally remove rows with zero variance, then remove rows whose variance is less than the 5th percentile of all row variances. The rationale is to remove unreliable or uninformative rows so that downstream top-value scoring (SD, MAD, CV, ATC) and partitioning operate on a stable matrix; this step precedes row scaling (z-score or min-max) and feature selection.

## Related tools

- **cola** (Provides adjust_matrix(), the sequential imputation/outlier-adjustment/variance-filtering function applied to the input matrix) — https://github.com/jokergoo/cola_manuscript
- **impute** (R package whose impute.knn() function fills remaining missing values after rows with >25% missing samples are removed)
- **R** (Runtime environment (R/Bioconductor) in which adjust_matrix() and impute.knn() are executed on the numeric matrix)

## Examples

```
library(cola); m2 = adjust_matrix(m)  # m: numeric expression matrix with rows = genes, columns = samples
```

## Evaluation signals

- No missing (NA) values remain in the cleaned matrix (all were either row-removed or imputed via impute.knn()).
- Every column contains at most 25% missing values among retained rows — rows exceeding that threshold are absent.
- No cleaned value lies above the 95th percentile or below the 5th percentile of its row distribution (values were clamped to the percentile values).
- No retained row has zero variance; the minimum row variance is >= the 5th percentile of the row-variance distribution computed after winsorization.
- Row count decreases monotonically through the steps; the output matrix is still numeric with the original sample columns intact.

## Limitations

- Imputation via impute.knn() depends on correlated neighbors, so in very sparse or weakly correlated matrices imputed values may be unreliable.
- The 25% missingness, 5th/95th percentile winsorization, and 5th-percentile variance cutoffs are fixed defaults; datasets needing different thresholds require custom handling.
- These cleaning defaults are dataset-agnostic — per-dataset filters such as the MCF10CA <2-million-reads cell filter or recount2 gene filters are separate steps not covered by adjust_matrix().

## Evidence
