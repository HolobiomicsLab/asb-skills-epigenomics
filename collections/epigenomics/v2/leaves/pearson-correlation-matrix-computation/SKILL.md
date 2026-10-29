---
name: pearson-correlation-matrix-computation
description: Use when a feature table of variant-prioritization predictor scores (e.g.,
  the 574 dsQTL-positive and 27,735 negative SNPs of training dataset 1) must be screened
  for collinearity before feeding scores into ncDeepBrain or gene prioritization.
when_to_use_negative:
- The input is already a correlation matrix or the goal is gene-level (not variant-feature-level)
  screening — the paper's gene-prioritization summary-statistics check is a separate
  skill.
- Features have not yet been imputed and you cannot run the bPCA fill; unimputed missing
  DANN/CADD/EIGEN/GWAVA/FATHMM values would distort the Pearson coefficients.
- 'The dataset is not the deltaSVM-based training data, so results would not be comparable
  to Additional file 1: Table S3.'
tools:
- name: CADD
  role: One of the eleven variant feature scores whose pairwise correlation is computed
- name: DANN
  role: One of the eleven variant feature scores whose pairwise correlation is computed;
    requires bPCA imputation for missing values
- name: GWAVA
  role: One of the eleven variant feature scores whose pairwise correlation is computed;
    requires bPCA imputation for missing values
- name: FATHMM-MKL
  role: One of the eleven variant feature scores whose pairwise correlation is computed;
    requires bPCA imputation for missing values
- name: deltaSVM
  role: Source of the training dataset 1 labels (574 positive dsQTL SNPs, 27,735 negatives)
    on which the correlation matrix is computed
- name: iMEGES
  role: Provides the bPCA imputation option (-B bPCA.R) used before computing the
    correlation matrix
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:bf88e38649b9363b779904a3dca995f6000cbe3aae015863f6f8246e97a188b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - deltaSVM
  - iMEGES
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pearson-correlation-matrix-computation

## Summary

Computes the pairwise Pearson correlation structure of the eleven non-coding variant feature scores (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTLs, H3K4me3, H3K4me1, H3K27me3, H3K27ac) used by ncDeepBrain, to test score redundancy before deep-learning integration.

## When to use

Use when a feature table of variant-prioritization predictor scores (e.g., the 574 dsQTL-positive and 27,735 negative SNPs of training dataset 1) must be screened for collinearity before feeding scores into ncDeepBrain or gene prioritization.

## When NOT to use

- The input is already a correlation matrix or the goal is gene-level (not variant-feature-level) screening — the paper's gene-prioritization summary-statistics check is a separate skill.
- Features have not yet been imputed and you cannot run the bPCA fill; unimputed missing DANN/CADD/EIGEN/GWAVA/FATHMM values would distort the Pearson coefficients.
- The dataset is not the deltaSVM-based training data, so results would not be comparable to Additional file 1: Table S3.

## Inputs

- Training dataset 1 (deltaSVM data): feature-score table of 574 dsQTL-positive SNPs and 27,735 negative SNPs (MAF > 5%) with eleven columns (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTLs from CommonMind, H3K4me3, H3K4me1, H3K27me3, H3K27ac)

## Outputs

- correlation_matrix.csv: an 11x11 symmetric Pearson correlation matrix with unit diagonal
- Correlation plot of pairwise score relationships analogous to Fig. 2(a) of the paper

## How to apply

Load training dataset 1 (deltaSVM data: 574 dsQTL-positive SNPs and 27,735 negative SNPs with MAF > 5%) with the eleven pre-computed feature scores into a pandas DataFrame. Impute missing feature values with bPCA (Bayesian Principal Component Analysis), as the paper applies bPCA fill for DANN, CADD, EIGEN, GWAVA and FATHMM scores via the iMEGES -B/--bPCA option. Compute the 11x11 pairwise Pearson matrix with pandas.DataFrame.corr, write it to correlation_matrix.csv, and produce a color-coded pairwise correlation plot analogous to Fig. 2(a). The rationale is that highly correlated, redundant predictors should be identified before they are integrated by the deep learning framework.

## Related tools

- **CADD** (One of the eleven variant feature scores whose pairwise correlation is computed)
- **DANN** (One of the eleven variant feature scores whose pairwise correlation is computed; requires bPCA imputation for missing values)
- **GWAVA** (One of the eleven variant feature scores whose pairwise correlation is computed; requires bPCA imputation for missing values)
- **FATHMM-MKL** (One of the eleven variant feature scores whose pairwise correlation is computed; requires bPCA imputation for missing values)
- **deltaSVM** (Source of the training dataset 1 labels (574 positive dsQTL SNPs, 27,735 negatives) on which the correlation matrix is computed)
- **iMEGES** (Provides the bPCA imputation option (-B bPCA.R) used before computing the correlation matrix) — https://github.com/WGLab/iMEGES

## Examples

```
corr = df[['EIGEN','CADD','DANN','GWAVA','FATHMM','GNOMAD','eQTL','H3K4me3','H3K4me1','H3K27me3','H3K27ac']].corr(method='pearson'); corr.to_csv('correlation_matrix.csv')
```

## Evaluation signals

- Output is an 11x11 symmetric matrix with unit diagonal and all coefficients in [-1, 1].
- Matrix structure is consistent with the correlation of variables tabulated in Additional file 1: Table S3.
- Feature values were bPCA-imputed before correlation (DANN, CADD, EIGEN, GWAVA, FATHMM columns have no remaining NaNs).
- Row/column labels match the eleven expected scores: EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, eQTLs, H3K4me3, H3K4me1, H3K27me3, H3K27ac.
- Highly correlated score pairs identified in the plot should match the redundant-predictor discussion around Fig. 2(a).

## Limitations

- Correlation is computed on the deltaSVM training dataset 1; correlations may not transfer to other variant sets or tissues.
- The paper notes missing feature values are common (bPCA fill needed for DANN, CADD, EIGEN, GWAVA, FATHMM), and imputation choice can affect coefficients.
- This step addresses feature collinearity only; the paper separately cautions that some procedures rely on imputed GWAS data where hits may be proxy markers rather than true causal variants.

## Evidence

- [results] First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization.: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
- [readme] We integrated five different scores from different predictors: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind and enhancer/promoters from PsychENCODE"
- [readme] The bPCA R code to do the imputation: "-B bPCA.R, --bPCA bPCA.R
                        The bPCA R code to do the imputation"
