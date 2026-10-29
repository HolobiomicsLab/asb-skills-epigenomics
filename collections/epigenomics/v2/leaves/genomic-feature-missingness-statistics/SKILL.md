---
name: genomic-feature-missingness-statistics
description: Use when preparing variant-prioritization training matrices from ANNOVAR-annotated
  score columns (DANN, CADD, EIGEN, GWAVA, FATHMM) and you need to quantify per-score
  missingness before/after bPCA imputation.
when_to_use_negative:
- The score columns are already complete (no NaN entries), so missingness quantification
  and imputation add nothing
- The goal is variant prioritization on an already-trained ncDeepBrain/iMEGES model
  rather than training-data preparation
- The features are gene-level scores (RVIS, GTEx, haploinsufficiency, Phenolyzer)
  in the gene-prioritization step, which are not the target of this bPCA-based missingness
  handling
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: bPCA
  role: R code (bPCA.R) that performs Bayesian Principal Component Analysis to impute
    the missing DANN, CADD, EIGEN, GWAVA and FATHMM values being audited
  repo: https://github.com/WGLab/iMEGES
- name: iMEGES
  role: Pipeline whose ncDeepBrain step consumes the missingness-audited, imputed
    score matrix as training data
  repo: https://github.com/WGLab/iMEGES
- name: CADD
  role: One of the five precomputed functional scores whose missing entries are counted
    and imputed
- name: DANN
  role: One of the five precomputed functional scores whose missing entries are counted
    and imputed
- name: GWAVA
  role: One of the five precomputed functional scores whose missing entries are counted
    and imputed
- name: FATHMM-MKL
  role: One of the five precomputed functional scores whose missing entries are counted
    and imputed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:5110c35d09b0cb69e9a66de8c9e41599983590dc354a06fbe686ba015d495aa7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - bPCA
  - iMEGES
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# genomic-feature-missingness-statistics

## Summary

Computes per-feature missing-value counts and fractions for genome-wide functional scores (EIGEN, CADD, DANN, GWAVA, FATHMM) in an ANNOVAR-annotated variant matrix, both to characterize missingness (Additional file 1: Table S2 in iMEGES) and to validate that bPCA imputation has filled all gaps.

## When to use

Use when preparing variant-prioritization training matrices from ANNOVAR-annotated score columns (DANN, CADD, EIGEN, GWAVA, FATHMM) and you need to quantify per-score missingness before/after bPCA imputation.

## When NOT to use

- The score columns are already complete (no NaN entries), so missingness quantification and imputation add nothing
- The goal is variant prioritization on an already-trained ncDeepBrain/iMEGES model rather than training-data preparation
- The features are gene-level scores (RVIS, GTEx, haploinsufficiency, Phenolyzer) in the gene-prioritization step, which are not the target of this bPCA-based missingness handling

## Inputs

- ANNOVAR-annotated feature matrix of candidate SNVs with EIGEN, CADD, DANN, GWAVA and FATHMM score columns (with missing values)
- ANNOVAR input/bed format file of non-coding variants (input.bed)

## Outputs

- Per-feature missing-value counts/fractions table (missingness summary, e.g. missing_value_summary.csv)
- Fully imputed score matrix (e.g. imputed_scores.csv) with zero missing entries in the five score columns
- Verification report that row counts match the input matrix and missing-value rates match Additional file 1: Table S2

## How to apply

Load the ANNOVAR-annotated feature matrix containing genome-wide precomputed EIGEN, CADD, DANN, GWAVA and FATHMM scores for the candidate SNVs from the training datasets. Compute per-feature missing-value counts and fractions for each of the five scores to reproduce the statistics of Additional file 1: Table S2. After running the bPCA (Bayesian Principal Component Analysis) imputation — which estimates each missing value from the non-missing scores of each variant across all potential SNVs in the human whole genome — re-check the five score columns to confirm zero residual missing entries. This missingness audit both documents the data-quality problem motivating bPCA and serves as the post-imputation validation gate before the matrix feeds ncDeepBrain training.

## Related tools

- **bPCA** (R code (bPCA.R) that performs Bayesian Principal Component Analysis to impute the missing DANN, CADD, EIGEN, GWAVA and FATHMM values being audited) — https://github.com/WGLab/iMEGES
- **iMEGES** (Pipeline whose ncDeepBrain step consumes the missingness-audited, imputed score matrix as training data) — https://github.com/WGLab/iMEGES
- **CADD** (One of the five precomputed functional scores whose missing entries are counted and imputed)
- **DANN** (One of the five precomputed functional scores whose missing entries are counted and imputed)
- **GWAVA** (One of the five precomputed functional scores whose missing entries are counted and imputed)
- **FATHMM-MKL** (One of the five precomputed functional scores whose missing entries are counted and imputed)

## Examples

```
python ncDeepBrain.py -f1 input.bed -B bPCA.R -o imputed_scores.txt
```

## Evaluation signals

- missing_value_summary.csv reproduces the per-feature missing-value rates reported in Additional file 1: Table S2
- imputed_scores.csv exists with the same row count as the input feature matrix
- The five imputed score columns (DANN, CADD, EIGEN, GWAVA, FATHMM) contain zero missing/NaN entries after bPCA
- Pre-imputation and post-imputation matrices have identical variant identifiers and column ordering (only values were filled, no rows dropped or added)

## Limitations

- Imputation is a secondary analysis over precomputed genome-wide scores; if the underlying annotation source (e.g. GWAVA trained on HGMD) changes, missingness statistics and imputed values shift accordingly
- bPCA imputation quality depends on the non-missing score patterns of the other variants; heavily sparse columns may yield less reliable estimates
- The article notes a general limitation of obtaining large-scale high-quality data for training statistical models, of which missing functional annotations are a symptom

## Evidence

- [supplementary] imputation for DANN, CADD, EIGEN, GWAVA and FATHMM is performed using the non-missing scores of each variant across all potential SNVs in the human whole genome: [evidence span withheld — anchor status: not_found]
- [readme] -B bPCA.R, --bPCA bPCA.R The bPCA R code to do the imputation: "-B bPCA.R, --bPCA bPCA.R
                        The bPCA R code to do the imputation"
- [readme] We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind"
