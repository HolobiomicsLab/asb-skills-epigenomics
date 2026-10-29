---
name: methylation-array-probe-preprocessing
description: Use when the input is a raw Illumina 450K methylation beta-value matrix
  (probes x samples, e.g. from GEO GSE36278) containing sex-chromosome probes, SNP-overlapping
  probes, missing values, outliers, and low-variance rows prior to cola consensus
  partitioning.
when_to_use_negative:
- The matrix is already preprocessed (missing values imputed, outliers adjusted, low-variance
  rows removed) — re-running adjust_matrix() would not change it
- The input is a single-cell RNA-Seq matrix (e.g. MCF10CA), where cola instead filters
  cells with <2 million reads and non-protein-coding genes, not methylation probes
- Only feature selection or consensus partitioning is needed and the matrix has already
  passed through this preprocessing step
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: cola
  role: Provides adjust_matrix() to remove sex-chromosome and SNP-overlapping probes,
    impute missing values, adjust outliers and remove low-variance rows
  repo: https://github.com/jokergoo/cola_manuscript
- name: impute
  role: R package providing impute.knn() used by adjust_matrix() to impute missing
    methylation values
- name: GEOquery
  role: Downloads the raw GBM 450K methylation matrix from GEO (accession GSE36278)
    before preprocessing
- name: bsub
  role: Used to submit the cola preprocessing/analysis jobs to an LSF computing cluster
  repo: https://github.com/jokergoo/bsub
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1146
    title: '<i>cola</i>: an R/Bioconductor package for consensus partitioning through
      a general framework'
  source_content_hash: sha256:edc3d98b6cb313ba64f086ea3dfc6549812b43efb9a88fcfc47b55f96f7a3e85
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
  - GEOquery
  - R
  - bsub
  - cola
  - impute
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# methylation-array-probe-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess Illumina DNA methylation array matrices (e.g. the GBM 450K dataset GSE36278) with cola's adjust_matrix() before consensus partitioning: drop sex-chromosome and SNP-overlapping probes, impute missing values, winsorize outliers, and remove low-variance rows so that top-value feature selection (SD vs ATC) operates on a clean probe matrix.

## When to use

Use when the input is a raw Illumina 450K methylation beta-value matrix (probes x samples, e.g. from GEO GSE36278) containing sex-chromosome probes, SNP-overlapping probes, missing values, outliers, and low-variance rows prior to cola consensus partitioning.

## When NOT to use

- The matrix is already preprocessed (missing values imputed, outliers adjusted, low-variance rows removed) — re-running adjust_matrix() would not change it
- The input is a single-cell RNA-Seq matrix (e.g. MCF10CA), where cola instead filters cells with <2 million reads and non-protein-coding genes, not methylation probes
- Only feature selection or consensus partitioning is needed and the matrix has already passed through this preprocessing step

## Inputs

- Illumina 450K methylation matrix (probes x samples), e.g. GBM 450K dataset from GEO accession GSE36278
- Probe annotation (chromosome location, known-SNP overlap, CpG-island association categories)

## Outputs

- Cleaned, imputed and filtered methylation matrix suitable for cola::run_all_consensus_partitions()
- Downstream comparison outputs such as gbm_450k_sd_atc_comparison.csv and the automated gbm_450k_cola_report.html

## How to apply

Fetch the 450K methylation matrix (e.g. GSE36278 via GEOquery), then pass it through cola's adjust_matrix(). First remove probes that are on sex chromosomes and probes that overlap known SNPs; remove rows in which >25% of samples have missing values, then impute remaining missing values with impute.knn() from the R package impute. Winsorize row values by replacing values larger than the 95th percentile or smaller than the 5th percentile with the corresponding percentiles. Finally drop rows with zero variance and rows with variance less than the 5th percentile of all row variances. The cleaned matrix is the input for run_all_consensus_partitions() with top-value methods SD and ATC; on the GBM 450K data this preprocessing led to an SD/ATC top-5000 probe overlap of only 1.76% and CpG-island probe fractions of 60.6% (SD) vs 6.6% (ATC).

## Related tools

- **cola** (Provides adjust_matrix() to remove sex-chromosome and SNP-overlapping probes, impute missing values, adjust outliers and remove low-variance rows) — https://github.com/jokergoo/cola_manuscript
- **impute** (R package providing impute.knn() used by adjust_matrix() to impute missing methylation values)
- **GEOquery** (Downloads the raw GBM 450K methylation matrix from GEO (accession GSE36278) before preprocessing)
- **bsub** (Used to submit the cola preprocessing/analysis jobs to an LSF computing cluster) — https://github.com/jokergoo/bsub

## Examples

```
mat = cola::adjust_matrix(methylation_mat)  # 450K matrix: drop sex-chr/SNP probes, impute.knn, winsorize 5th/95th, drop low-variance rows
```

## Evaluation signals

- Output matrix contains no probes on sex chromosomes and no probes overlapping known SNPs
- No missing values remain after impute.knn() imputation; rows with >25% missing values were removed first
- Row values fall within the 5th/95th percentile bounds (winsorized), and no row has variance below the 5th percentile of all row variances
- Downstream on GSE36278: SD- and ATC-based top-5000 probe sets intersect at ~1.76% of probes
- Annotated top-5000 probe sets show CpG-island probe fractions of 60.6% (SD) vs 6.6% (ATC), and SD-selected probes recover the DKFZ GBM subtype classification

## Limitations

- No version number is stated for the cola package or R in the manuscript, limiting reproducibility of versioned results; no changelog is available
- The article states rows with >25% missing samples are removed and zero-variance rows removed, but per-dataset missingness patterns may leave fewer or more rows than expected; verify counts before partitioning

## Evidence
