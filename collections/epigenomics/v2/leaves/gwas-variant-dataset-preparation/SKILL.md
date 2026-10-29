---
name: gwas-variant-dataset-preparation
description: Use when preparing a training set of SNPs annotated with multiple variant-prioritization
  scores (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, eQTLs, histone marks) and you
  must check collinearity/redundancy before integrating scores.
when_to_use_negative:
- 'Input is already a cleaned, imputed feature table whose correlation structure has
  been assessed (e.g. Additional file 1: Table S3 reproduced).'
- The goal is gene prioritization (step 2 of iMEGES) on gene-based scores such as
  RVIS, GTEx, haploinsufficiency — this skill addresses variant-level score redundancy
  only.
- The training set does not contain multiple overlapping predictor scores to compare;
  with a single score per variant there is no collinearity question.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: iMEGES
  role: Source of ncDeepBrain.py and bPCA.R used to assemble the eleven-score feature
    table and impute missing values
  repo: https://github.com/WGLab/iMEGES
- name: CADD
  role: One of the eleven non-coding variant feature scores tabulated for each training
    SNP
  repo: http://cadd.gs.washington.edu/
- name: DANN
  role: Non-coding variant predictor score included in the feature matrix (bPCA-imputed)
  repo: https://cbcl.ics.uci.edu/public_data/DANN/
- name: GWAVA
  role: Non-coding variant predictor score included in the feature matrix (bPCA-imputed)
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: FATHMM-MKL
  role: Non-coding variant predictor score included in the feature matrix (bPCA-imputed)
  repo: http://fathmm.biocompute.org.uk/
- name: deltaSVM
  role: Source of the training data labels (574 dsQTL-positive SNPs vs 27,735 negative
    SNPs with MAF > 5%)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:b53a667f4a9f826bd81c1e331270a56bdf7d5a00b9e26885f0ac5154b075f5fd
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gwas-variant-dataset-preparation

## Summary

Assemble a GWAS/deltaSVM-derived variant feature table (positive dsQTL SNPs and frequency-matched negative SNPs) annotated with multiple non-coding variant predictor scores, impute missing values with bPCA, and compute pairwise Pearson correlations among the eleven feature scores to test for redundancy before deep learning integration.

## When to use

Use when preparing a training set of SNPs annotated with multiple variant-prioritization scores (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, eQTLs, histone marks) and you must check collinearity/redundancy before integrating scores.

## When NOT to use

- Input is already a cleaned, imputed feature table whose correlation structure has been assessed (e.g. Additional file 1: Table S3 reproduced).
- The goal is gene prioritization (step 2 of iMEGES) on gene-based scores such as RVIS, GTEx, haploinsufficiency — this skill addresses variant-level score redundancy only.
- The training set does not contain multiple overlapping predictor scores to compare; with a single score per variant there is no collinearity question.

## Inputs

- BED/Annovar-format variant file of dsQTL-positive SNPs and MAF > 5% negative SNPs from deltaSVM data
- Per-SNP feature score tracks: EIGEN, CADD, DANN, GWAVA, FATHMM-MKL, GNOMAD frequency, brain eQTLs (CommonMind), H3K4me3, H3K4me1, H3K27me3, H3K27ac
- bPCA.R imputation script from the iMEGES repository

## Outputs

- correlation_matrix.csv — an 11x11 symmetric Pearson correlation matrix with unit diagonal
- Correlation plot of pairwise feature panels (analogous to Fig. 2(a))
- Summary statistics per feature variable for validity checking before prioritization

## How to apply

Load training dataset 1 (574 dsQTL-positive SNPs and 27,735 negative SNPs with MAF > 5% from the deltaSVM data) with the eleven pre-computed feature scores into a pandas DataFrame. Impute missing feature values using bPCA (Bayesian Principal Component Analysis), as the paper applies bPCA fill for DANN, CADD, EIGEN, GWAVA and FATHMM scores — the iMEGES CLI exposes this via the bPCA.R option. Compute the 11x11 pairwise Pearson correlation coefficient matrix with pandas.DataFrame.corr and write it out (correlation_matrix.csv) alongside a color-coded pairwise correlation plot analogous to Fig. 2(a). The rationale is to detect redundant, highly correlated predictors before the ncDeepBrain deep learning framework integrates general non-coding variant scores with tissue-related scores (brain eQTLs from CommonMind, PsychENCODE/RoadMap enhancer/promoter marks).

## Related tools

- **iMEGES** (Source of ncDeepBrain.py and bPCA.R used to assemble the eleven-score feature table and impute missing values) — https://github.com/WGLab/iMEGES
- **CADD** (One of the eleven non-coding variant feature scores tabulated for each training SNP) — http://cadd.gs.washington.edu/
- **DANN** (Non-coding variant predictor score included in the feature matrix (bPCA-imputed)) — https://cbcl.ics.uci.edu/public_data/DANN/
- **GWAVA** (Non-coding variant predictor score included in the feature matrix (bPCA-imputed)) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **FATHMM-MKL** (Non-coding variant predictor score included in the feature matrix (bPCA-imputed)) — http://fathmm.biocompute.org.uk/
- **deltaSVM** (Source of the training data labels (574 dsQTL-positive SNPs vs 27,735 negative SNPs with MAF > 5%))

## Examples

```
import pandas as pd; df = pd.read_csv('training_ds1_features.csv'); df = bPCA_impute(df, ['DANN','CADD','EIGEN','GWAVA','FATHMM']); df.corr(method='pearson').to_csv('correlation_matrix.csv')
```

## Evaluation signals

- Output correlation matrix must be exactly 11x11, symmetric, with unit diagonal and all off-diagonal coefficients in [-1, 1].
- Matrix structure should match the correlation of variables in training dataset 1 tabulated in Additional file 1: Table S3.
- No missing values should remain after bPCA imputation of DANN, CADD, EIGEN, GWAVA and FATHMM scores.
- Summary statistics for all feature variables must be examined for validity/outliers before the scores enter gene prioritization.
- Downstream sanity check: ncDeepBrain trained on this dataset should reach a classification AUC of 80% on deltaSVM training dataset 1.

## Limitations

- Missing predictor scores require bPCA imputation, which introduces model dependence for DANN, CADD, EIGEN, GWAVA and FATHMM.
- It is challenging to obtain large-scale high quality data for training the statistical/deep learning models, limiting feature-table generalization.
- Some procedures rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the derived labels less reliable than ideal.
- The feature table is tied to the specific eleven predictors and PsychENCODE/RoadMap enhancer/promoter annotations used by ncDeepBrain; swapping sources changes the correlation structure.

## Evidence

- [readme] We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind and enhancer/promoters from PsychENCODE and Roadmap Epigenomics projects: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind and enhancer/promoters from PsychENCODE"
- [results] The two types of scores were integrated by a deep learning framework (ncDeepBrain for short in iMEGES).: "The two types of scores were integrated by a deep learning framework (ncDeepBrain for short in iMEGES)."
- [results] First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization.: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
