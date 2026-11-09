---
name: tabular-imputation-quality-control
description: Use when a variant-by-feature score matrix (e.g. EIGEN, CADD, DANN, GWAVA,
  FATHMM columns from ANNOVAR annotation) contains missing entries that must be filled
  before feeding the features into the ncDeepBrain/iMEGES deep learning model.
when_to_use_negative:
- The feature table is already complete (zero missing values) — imputation is unnecessary
  and could distort true scores.
- The goal is variant calling or annotation rather than preparing numeric feature
  scores for deep-learning gene/variant prioritization.
- The missingness structure involves features outside the five score types (EIGEN,
  CADD, DANN, GWAVA, FATHMM) for which the genome-wide non-missing-score bPCA strategy
  was validated.
edam_operation: http://edamontology.org/operation_0286
tools:
- name: bPCA
  role: Bayesian Principal Component Analysis R code (bPCA.R) invoked by ncDeepBrain.py
    (-B/--bPCA) to impute missing feature-score values
  repo: https://github.com/WGLab/iMEGES
- name: iMEGES
  role: Pipeline whose ncDeepBrain step consumes the imputed DANN, CADD, EIGEN, GWAVA
    and FATHMM scores for variant prioritization
  repo: https://github.com/WGLab/iMEGES
- name: ANNOVAR
  role: Provides the annotated input file format (-f1 input.bed, Annovar input format)
    for the feature matrix submitted to imputation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:6ac7b541a15df0c9e2dc97effa28167e48d4cd9add03639d12d3d7b086abc1f9
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tabular-imputation-quality-control

## Summary

Imputes missing feature scores (DANN, CADD, EIGEN, GWAVA, FATHMM) in ANNOVAR-annotated variant feature matrices using Bayesian Principal Component Analysis (bPCA), and audits the imputation with per-feature missingness counts before downstream deep-learning prioritization in iMEGES.

## When to use

Use when a variant-by-feature score matrix (e.g. EIGEN, CADD, DANN, GWAVA, FATHMM columns from ANNOVAR annotation) contains missing entries that must be filled before feeding the features into the ncDeepBrain/iMEGES deep learning model.

## When NOT to use

- The feature table is already complete (zero missing values) — imputation is unnecessary and could distort true scores.
- The goal is variant calling or annotation rather than preparing numeric feature scores for deep-learning gene/variant prioritization.
- The missingness structure involves features outside the five score types (EIGEN, CADD, DANN, GWAVA, FATHMM) for which the genome-wide non-missing-score bPCA strategy was validated.

## Inputs

- ANNOVAR-annotated feature matrix of genome-wide precomputed EIGEN, CADD, DANN, GWAVA and FATHMM scores for candidate SNVs from the training datasets
- bPCA.R R script for Bayesian Principal Component Analysis imputation

## Outputs

- imputed_scores.csv — completed matrix with zero missing entries in the five imputed score columns and the same row count as the input
- missing_value_summary.csv — table of per-feature missing-value counts and fractions matching Additional file 1: Table S2

## How to apply

Load the ANNOVAR-annotated feature matrix of genome-wide precomputed EIGEN, CADD, DANN, GWAVA and FATHMM scores for the candidate SNVs from the training datasets. First compute per-feature missing-value counts and fractions to reproduce the statistics reported in Additional file 1: Table S2. Then run the bPCA (Bayesian Principal Component Analysis) R code (bPCA.R, wired into ncDeepBrain.py via the -B/--bPCA option) to impute each missing value using the non-missing scores of each variant across all potential SNVs in the human whole genome. Finally verify that the five score columns contain no residual missing values and that the row count matches the input before writing the completed matrix forward to deep learning training.

## Related tools

- **bPCA** (Bayesian Principal Component Analysis R code (bPCA.R) invoked by ncDeepBrain.py (-B/--bPCA) to impute missing feature-score values) — https://github.com/WGLab/iMEGES
- **iMEGES** (Pipeline whose ncDeepBrain step consumes the imputed DANN, CADD, EIGEN, GWAVA and FATHMM scores for variant prioritization) — https://github.com/WGLab/iMEGES
- **ANNOVAR** (Provides the annotated input file format (-f1 input.bed, Annovar input format) for the feature matrix submitted to imputation)

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o imputed_out --buildver hg19
```

## Evaluation signals

- imputed_scores.csv exists and has the same row count as the input matrix.
- Zero missing entries remain in the DANN, CADD, EIGEN, GWAVA and FATHMM columns after imputation.
- Computed per-feature missing-value rates match the reported Additional file 1: Table S2 summary statistics.
- The bPCA.R script was passed correctly via the -B/--bPCA flag so the imputation step actually ran inside ncDeepBrain.py.
- Imputed values derive from non-missing scores of each variant across all potential SNVs in the human whole genome, not from per-feature means only.

## Limitations

- Imputation quality depends on the genome-wide non-missing score landscape; the strategy is specific to the five score types (DANN, CADD, EIGEN, GWAVA, FATHMM) used in iMEGES.
- The procedure relies on precomputed genome-wide scores, so coverage depends on the ANNOVAR-annotated source matrix.
- Some downstream procedures must rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the model less reliable than ideal.

## Evidence

- [readme] The bPCA R code to do the imputation: "-B bPCA.R, --bPCA bPCA.R
                        The bPCA R code to do the imputation"
- [readme] We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs"
- [results] First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization.: "First we examined the summary statistics for all feature variables to ensure validity of these variables used in gene prioritization."
