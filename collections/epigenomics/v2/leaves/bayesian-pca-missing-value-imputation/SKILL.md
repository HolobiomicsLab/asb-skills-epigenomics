---
name: bayesian-pca-missing-value-imputation
description: Use when an ANNOVAR-annotated variant feature table (e.g. EIGEN, CADD,
  DANN, GWAVA, FATHMM scores) contains missing entries and a complete matrix is required
  before computing summary statistics, correlations, or training a deep learning model.
when_to_use_negative:
- The input is already a complete feature table with no missing entries in the five
  predictor score columns
- Missingness affects metadata columns (gene names, coordinates) rather than the DANN/CADD/EIGEN/GWAVA/FATHMM
  score columns, since bPCA imputation is designed for the numeric score block
- The goal is gene-level scores (RVIS, GTEx, haploinsufficiency, Phenolyzer) rather
  than variant-level non-coding predictor scores
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: bPCA
  role: Bayesian Principal Component Analysis R script (bPCA.R) used to fill missing
    DANN, CADD, EIGEN, GWAVA and FATHMM values
- name: iMEGES
  role: Pipeline whose ncDeepBrain.py accepts the bPCA R code via the -B/--bPCA option
    and consumes the imputed scores for variant prioritization
  repo: https://github.com/WGLab/iMEGES
- name: CADD
  role: One of the five predictor score columns whose missing entries are imputed
- name: DANN
  role: One of the five predictor score columns whose missing entries are imputed
- name: GWAVA
  role: One of the five predictor score columns whose missing entries are imputed
- name: FATHMM-MKL
  role: One of the five predictor score columns whose missing entries are imputed
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:d889b4cec7a7c98212f9e47f3c91124669954be2906f945f62217f4cc4ca7bff
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
  - deltaSVM
  - iMEGES
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bayesian-pca-missing-value-imputation

## Summary

Imputes missing variant feature scores (DANN, CADD, EIGEN, GWAVA, FATHMM) in the iMEGES/ncDeepBrain training matrices using Bayesian Principal Component Analysis (bPCA), so the deep-learning variant-prioritization step receives a complete feature matrix without dropping variants.

## When to use

Use when an ANNOVAR-annotated variant feature table (e.g. EIGEN, CADD, DANN, GWAVA, FATHMM scores) contains missing entries and a complete matrix is required before computing summary statistics, correlations, or training a deep learning model.

## When NOT to use

- The input is already a complete feature table with no missing entries in the five predictor score columns
- Missingness affects metadata columns (gene names, coordinates) rather than the DANN/CADD/EIGEN/GWAVA/FATHMM score columns, since bPCA imputation is designed for the numeric score block
- The goal is gene-level scores (RVIS, GTEx, haploinsufficiency, Phenolyzer) rather than variant-level non-coding predictor scores

## Inputs

- ANNOVAR-annotated feature matrix with genome-wide EIGEN, CADD, DANN, GWAVA and FATHMM scores for candidate SNVs (bed/Annovar input format)
- Training dataset tables (e.g. 574 dsQTL-positive SNPs and 27,735 negative SNPs with MAF > 5% from deltaSVM data)

## Outputs

- imputed_scores.csv — feature matrix with identical row count and zero missing entries in the five imputed score columns
- missing_value_summary.csv — per-feature missing-value counts and fractions (Table S2 reproduction)

## How to apply

Load the ANNOVAR-annotated feature matrix with genome-wide precomputed EIGEN, CADD, DANN, GWAVA and FATHMM scores for the candidate SNVs, then compute per-feature missing counts and fractions to reproduce the statistics of Additional file 1: Table S2. Run the bPCA R script (bPCA.R, passed to ncDeepBrain.py via the -B/--bPCA flag) to impute each missing value using the non-missing scores of each variant across all potential SNVs in the human whole genome. Verify the five score columns contain no residual missing values before proceeding to downstream steps such as the 11x11 Pearson correlation matrix of ncDeepBrain features. bPCA is preferred because it exploits the PCA structure of genome-wide score vectors rather than per-column mean fill, preserving inter-score covariance.

## Related tools

- **bPCA** (Bayesian Principal Component Analysis R script (bPCA.R) used to fill missing DANN, CADD, EIGEN, GWAVA and FATHMM values)
- **iMEGES** (Pipeline whose ncDeepBrain.py accepts the bPCA R code via the -B/--bPCA option and consumes the imputed scores for variant prioritization) — https://github.com/WGLab/iMEGES
- **CADD** (One of the five predictor score columns whose missing entries are imputed)
- **DANN** (One of the five predictor score columns whose missing entries are imputed)
- **GWAVA** (One of the five predictor score columns whose missing entries are imputed)
- **FATHMM-MKL** (One of the five predictor score columns whose missing entries are imputed)

## Examples

```
python ncDeepBrain.py -f1 variants.bed -E brain_eqtls.bed -B bPCA.R -o imputed_scores.csv -buildver hg38
```

## Evaluation signals

- imputed_scores.csv exists with the same row count as the input feature matrix
- Zero residual missing (NaN/NA) entries in the DANN, CADD, EIGEN, GWAVA and FATHMM columns after imputation
- Computed per-feature missing-value rates match the reported Additional file 1: Table S2 summary
- Imputation preserves the pairwise Pearson correlation structure of the five scores, consistent with the 11x11 correlation matrix in Additional file 1: Table S3 (symmetric, unit diagonal, coefficients in [-1, 1])
- bPCA.R is supplied to ncDeepBrain.py via the -B/--bPCA flag so the imputation runs inside the iMEGES workflow

## Limitations

- Imputation depends on genome-wide non-missing scores across all potential SNVs; if whole-genome score coverage is sparse, bPCA estimates may be unreliable
- bPCA is applied only to the five variant-predictor scores (DANN, CADD, EIGEN, GWAVA, FATHMM); other features such as GNOMAD frequency or eQTL indicators are not covered by this step
- Downstream correlation or deep-learning results can be sensitive to imputed values since the same five scores feed both the Table S3 redundancy check and ncDeepBrain training

## Evidence

- [methods] iMEGES uses bPCA (Bayesian Principal Component Analysis) to estimate missing values: [evidence span withheld — anchor status: not_found]
- [readme] bPCA flag in README options: "-B bPCA.R, --bPCA bPCA.R
                        The bPCA R code to do the imputation"
- [methods] bPCA fill applied before correlation analysis: [evidence span withheld — anchor status: not_found]
- [readme] ncDeepBrain integrates five predictor scores: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind"
- [results] summary statistics checked before gene prioritization: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
