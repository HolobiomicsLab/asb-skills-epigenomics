---
name: variant-annotation-feature-preprocessing
description: Use when preparing ANNOVAR-annotated, genome-wide variant score matrices
  for iMEGES variant-prioritization training and the DANN, CADD, EIGEN, GWAVA or FATHMM
  score columns contain missing values.
when_to_use_negative:
- The variant scores are already complete (no missing DANN/CADD/EIGEN/GWAVA/FATHMM
  entries), so imputation would only add noise.
- The task is gene prioritization (step 2 of iMEGES), which uses gene-level scores
  (RVIS, GTEx, haploinsufficiency, Phenolyzer) rather than variant-level annotation
  imputation.
- Predictor scores are missing because the variant was never annotated — re-run ANNOVAR/the
  individual predictors first rather than imputing from an empty feature column.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0102
tools:
- name: bPCA
  role: Bayesian Principal Component Analysis R script (bPCA.R) invoked by ncDeepBrain.py
    to impute missing feature scores from non-missing scores across all potential
    human SNVs
  repo: https://github.com/WGLab/iMEGES
- name: iMEGES
  role: Pipeline whose variant-prioritization training data is produced by this preprocessing/imputation
    step; exposes bPCA via the ncDeepBrain.py -B option
  repo: https://github.com/WGLab/iMEGES
- name: ANNOVAR
  role: Produces the annotated input/bed feature matrix that the imputation step consumes
- name: CADD
  role: One of the five imputed non-coding variant score columns (with DANN, EIGEN,
    GWAVA, FATHMM)
- name: DANN
  role: One of the five imputed non-coding variant score columns (with CADD, EIGEN,
    GWAVA, FATHMM)
- name: GWAVA
  role: One of the five imputed non-coding variant score columns (with CADD, DANN,
    EIGEN, FATHMM)
- name: FATHMM-MKL
  role: One of the five imputed non-coding variant score columns (with CADD, DANN,
    EIGEN, GWAVA)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:bfd37e78b14adeaf3e82c55f3a5f509eca3f742cb7b12ce3a092f192d08ee898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - ANNOVAR
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - bPCA
  - iMEGES
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# variant-annotation-feature-preprocessing

## Summary

Preprocess ANNOVAR-annotated variant feature matrices for iMEGES/ncDeepBrain training by quantifying missingness and imputing missing DANN, CADD, EIGEN, GWAVA and FATHMM scores with bPCA (Bayesian Principal Component Analysis), so the deep neural network receives a complete feature matrix.

## When to use

Use when preparing ANNOVAR-annotated, genome-wide variant score matrices for iMEGES variant-prioritization training and the DANN, CADD, EIGEN, GWAVA or FATHMM score columns contain missing values.

## When NOT to use

- The variant scores are already complete (no missing DANN/CADD/EIGEN/GWAVA/FATHMM entries), so imputation would only add noise.
- The task is gene prioritization (step 2 of iMEGES), which uses gene-level scores (RVIS, GTEx, haploinsufficiency, Phenolyzer) rather than variant-level annotation imputation.
- Predictor scores are missing because the variant was never annotated — re-run ANNOVAR/the individual predictors first rather than imputing from an empty feature column.

## Inputs

- ANNOVAR-annotated feature matrix of candidate SNVs with genome-wide precomputed EIGEN, CADD, DANN, GWAVA and FATHMM scores (with missing entries)
- bPCA.R imputation script (WGLab/iMEGES)

## Outputs

- imputed_scores.csv: complete feature matrix with zero missing entries in the five imputed score columns and the same row count as the input
- missing_value_summary.csv: per-feature missing-value counts and fractions matching Additional file 1: Table S2

## How to apply

Load the ANNOVAR-annotated feature matrix with genome-wide precomputed EIGEN, CADD, DANN, GWAVA and FATHMM scores for the candidate SNVs from the training datasets. Compute per-feature missing-value counts and fractions for each of the five scores to reproduce the missingness statistics of Additional file 1: Table S2. Run the bPCA (Bayesian Principal Component Analysis) R script (bPCA.R), which iMEGES exposes via the ncDeepBrain.py '-B/--bPCA' option, to estimate each missing value from the non-missing scores of each variant across all potential SNVs in the human whole genome. Verify no residual missingness remains in the five imputed score columns before writing the completed matrix for deep-learning training.

## Related tools

- **bPCA** (Bayesian Principal Component Analysis R script (bPCA.R) invoked by ncDeepBrain.py to impute missing feature scores from non-missing scores across all potential human SNVs) — https://github.com/WGLab/iMEGES
- **iMEGES** (Pipeline whose variant-prioritization training data is produced by this preprocessing/imputation step; exposes bPCA via the ncDeepBrain.py -B option) — https://github.com/WGLab/iMEGES
- **ANNOVAR** (Produces the annotated input/bed feature matrix that the imputation step consumes)
- **CADD** (One of the five imputed non-coding variant score columns (with DANN, EIGEN, GWAVA, FATHMM))
- **DANN** (One of the five imputed non-coding variant score columns (with CADD, EIGEN, GWAVA, FATHMM))
- **GWAVA** (One of the five imputed non-coding variant score columns (with CADD, DANN, EIGEN, FATHMM))
- **FATHMM-MKL** (One of the five imputed non-coding variant score columns (with CADD, DANN, EIGEN, GWAVA))

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o imputed_scores.csv -buildver hg19
```

## Evaluation signals

- imputed_scores.csv exists with the same row count as the input feature matrix
- Zero residual missing entries in the DANN, CADD, EIGEN, GWAVA and FATHMM columns of imputed_scores.csv
- Computed per-feature missing-value rates match the Additional file 1: Table S2 summary before imputation
- Non-missing score values are unchanged by imputation (only originally-missing cells differ from the input matrix)
- The ncDeepBrain.py run completes with the -B bPCA.R flag and produces the named output file without errors

## Limitations

- Imputation estimates missing DANN/CADD/EIGEN/GWAVA/FATHMM values from genome-wide non-missing scores of each variant, which is statistical inference rather than the predictor's true output for that variant.
- bPCA imputation quality depends on the coverage of the five score columns across all potential human whole-genome SNVs; heavily sparse columns may yield less reliable estimates.

## Evidence

- [methods] iMEGES uses bPCA to estimate missing values: "The bPCA R code to do the imputation"
- [readme] Five predictor scores integrated by iMEGES variant prioritization: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM)"
- [results] Feature validity check before gene prioritization: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
