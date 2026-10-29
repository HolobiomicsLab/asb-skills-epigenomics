---
name: genomic-feature-collinearity-diagnostics
description: Use when a multi-score genomic feature table (e.g., variant scores from
  multiple predictors plus epigenomic annotations) must be checked for pairwise redundancy/collinearity
  before feeding features into an integrative model such as ncDeepBrain.
when_to_use_negative:
- The feature table is already a post-integration score (e.g., ncDeepBrain or iMEGES
  output) rather than raw per-predictor scores
- The goal is general summary-statistic QA of gene-prioritization variables rather
  than pairwise linear correlation of variant scores
- Features are categorical or heavily non-linear, where Pearson correlation would
  misrepresent association
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0203
tools:
- name: CADD
  role: One of the eleven variant-prioritization feature scores whose pairwise correlation
    is computed
  repo: http://cadd.gs.washington.edu/
- name: DANN
  role: One of the eleven variant-prioritization feature scores whose pairwise correlation
    is computed
  repo: https://cbcl.ics.uci.edu/public_data/DANN/
- name: GWAVA
  role: One of the eleven variant-prioritization feature scores whose pairwise correlation
    is computed
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: FATHMM-MKL
  role: One of the eleven variant-prioritization feature scores whose pairwise correlation
    is computed
  repo: http://fathmm.biocompute.org.uk/
- name: deltaSVM
  role: Source of the dsQTL training labels/data used as the correlation-analysis
    dataset
- name: bPCA (bPCA.R)
  role: Imputation of missing DANN, CADD, EIGEN, GWAVA and FATHMM scores prior to
    correlation
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:d4ff7e66388907950f907f441bbecd704bda0eb6d75ff6b95006a4f544d2b4ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - bPCA (bPCA.R)
  - deltaSVM
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genomic-feature-collinearity-diagnostics

## Summary

Computes and visualizes the 11x11 pairwise Pearson correlation matrix of the eleven non-coding variant feature scores integrated by ncDeepBrain (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, brain eQTLs, H3K4me3, H3K4me1, H3K27me3, H3K27ac) to detect redundant, collinear predictors before deep learning integration.

## When to use

Use when a multi-score genomic feature table (e.g., variant scores from multiple predictors plus epigenomic annotations) must be checked for pairwise redundancy/collinearity before feeding features into an integrative model such as ncDeepBrain.

## When NOT to use

- The feature table is already a post-integration score (e.g., ncDeepBrain or iMEGES output) rather than raw per-predictor scores
- The goal is general summary-statistic QA of gene-prioritization variables rather than pairwise linear correlation of variant scores
- Features are categorical or heavily non-linear, where Pearson correlation would misrepresent association

## Inputs

- Training dataset 1 feature table: 574 dsQTL-positive and 27,735 negative SNPs (MAF > 5%) with columns EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTL (CommonMind), and enhancer/promoter indicators H3K4me3, H3K4me1, H3K27me3, H3K27ac (PsychENCODE / RoadMap Epigenomics)

## Outputs

- 11x11 symmetric Pearson correlation coefficient matrix (unit diagonal, coefficients in [-1, 1]) written to correlation_matrix.csv
- Color-coded pairwise correlation plot analogous to Fig. 2(a)

## How to apply

Load the feature-score matrix into a pandas DataFrame — for the iMEGES/ncDeepBrain use case this is training dataset 1 (574 dsQTL-positive SNPs and 27,735 negative SNPs with MAF > 5% from the deltaSVM data) carrying the eleven pre-computed scores. Impute missing predictor values with bPCA (Bayesian Principal Component Analysis), which the paper applies to fill DANN, CADD, EIGEN, GWAVA and FATHMM scores. Compute the pairwise Pearson correlation matrix with pandas.DataFrame.corr, export it (e.g., correlation_matrix.csv), and render a color-coded pairwise correlation plot analogous to Fig. 2(a) of the paper. Highly correlated score pairs indicate redundancy that informs which features are worth integrating in the downstream deep neural network.

## Related tools

- **CADD** (One of the eleven variant-prioritization feature scores whose pairwise correlation is computed) — http://cadd.gs.washington.edu/
- **DANN** (One of the eleven variant-prioritization feature scores whose pairwise correlation is computed) — https://cbcl.ics.uci.edu/public_data/DANN/
- **GWAVA** (One of the eleven variant-prioritization feature scores whose pairwise correlation is computed) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **FATHMM-MKL** (One of the eleven variant-prioritization feature scores whose pairwise correlation is computed) — http://fathmm.biocompute.org.uk/
- **deltaSVM** (Source of the dsQTL training labels/data used as the correlation-analysis dataset)
- **bPCA (bPCA.R)** (Imputation of missing DANN, CADD, EIGEN, GWAVA and FATHMM scores prior to correlation) — https://github.com/WGLab/iMEGES

## Examples

```
corr = df[['EIGEN','CADD','DANN','GWAVA','FATHMM','GNOMAD','eQTL','H3K4me3','H3K4me1','H3K27me3','H3K27ac']].corr(); corr.to_csv('correlation_matrix.csv')
```

## Evaluation signals

- Output correlation matrix must be 11x11 and symmetric with a unit diagonal
- All correlation coefficients must fall within [-1, 1]
- Matrix structure should match the correlation of variables in training dataset 1 tabulated in Additional file 1: Table S3
- Plot should visually reproduce the pairwise-panel layout of Fig. 2(a)
- Feature values must be bPCA-imputed (not dropped) so no NaNs propagate into the correlation matrix

## Limitations

- bPCA imputation for missing predictor scores can distort correlation estimates where coverage is sparse
- Pearson correlation captures only linear pairwise redundancy, not higher-order or non-linear dependence among the eleven scores

## Evidence

- [full_text] The paper computed Pearson correlations between each pair of the non-coding variant predictor scores on the training data: [evidence span withheld — anchor status: not_found]
- [readme] the imputation step: "The bPCA R code to do the imputation"
- [readme] We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind and enhancer/promoters: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind and enhancer/promoters"
- [results] collinearity QA of gene-prioritization features: [evidence span withheld — anchor status: not_found]
