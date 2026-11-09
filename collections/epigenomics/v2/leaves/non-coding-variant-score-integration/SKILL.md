---
name: non-coding-variant-score-integration
description: Use when annotating candidate non-coding SNVs with heterogeneous per-variant
  scores (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, brain eQTLs, H3K4me3/H3K4me1/H3K27me3/H3K27ac)
  and needing a single prioritization score.
when_to_use_negative:
- The input is a set of coding variants or protein-altering scores — ncDeepBrain is
  built specifically for non-coding variants affecting brain functions
- The goal is gene-level prioritization (step 2, iMEGES gene score integrating RVIS,
  GTEx, haploinsufficiency, Phenolyzer) rather than variant-level scoring
- Only a single variant score is available and no brain-specific annotations (eQTLs,
  enhancer/promoter marks) can be obtained — the integration approach requires the
  multi-score feature matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0199
tools:
- name: TensorFlow
  role: Deep learning framework (via Keras) used to train the ncDeepBrain integration
    model with 4 hidden layers and sigmoid output
  repo: https://github.com/WGLab/iMEGES
- name: CADD
  role: One of the five general non-coding variant predictor scores integrated; missing
    values filled by bPCA
  repo: http://cadd.gs.washington.edu/
- name: DANN
  role: One of the five general non-coding variant predictor scores integrated; missing
    values filled by bPCA
  repo: https://cbcl.ics.uci.edu/public_data/DANN/
- name: GWAVA
  role: One of the five general non-coding variant predictor scores integrated; missing
    values filled by bPCA
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: FATHMM-MKL
  role: One of the five general non-coding variant predictor scores integrated; missing
    values filled by bPCA
  repo: http://fathmm.biocompute.org.uk/
- name: ANNOVAR
  role: Annotates candidate variants in BED/Annovar input format with the feature
    scores
  repo: http://annovar.openbioinformatics.org/en/latest/
- name: bPCA
  role: Bayesian Principal Component Analysis R code (bPCA.R) that imputes missing
    DANN, CADD, EIGEN, GWAVA and FATHMM values
  repo: https://github.com/WGLab/iMEGES
- name: deltaSVM
  role: 'Source of labeled training data (dataset 1: 574 dsQTL-positive vs 27,735
    negative SNPs) for training the score-integration model'
- name: iMEGES
  role: Pipeline in which ncDeepBrain constitutes step 1 (variant prioritization)
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:b88182134ed878ea972af940d271441eb1c63a42ecf31579dc2b150a580dff60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - ANNOVAR
  - CADD
  - DANN
  - DeepSEA
  - FATHMM-MKL
  - GWAVA
  - TensorFlow
  - bPCA
  - deltaSVM
  - iMEGES
  - python scikit
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# non-coding-variant-score-integration

## Summary

Integration of five general non-coding variant predictor scores (EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, brain eQTLs, and brain enhancer/promoter histone marks into a single ncDeepBrain deep-learning variant score for prioritizing non-coding variants affecting brain functions in iMEGES.

## When to use

Use when annotating candidate non-coding SNVs with heterogeneous per-variant scores (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD, brain eQTLs, H3K4me3/H3K4me1/H3K27me3/H3K27ac) and needing a single prioritization score.

## When NOT to use

- The input is a set of coding variants or protein-altering scores — ncDeepBrain is built specifically for non-coding variants affecting brain functions
- The goal is gene-level prioritization (step 2, iMEGES gene score integrating RVIS, GTEx, haploinsufficiency, Phenolyzer) rather than variant-level scoring
- Only a single variant score is available and no brain-specific annotations (eQTLs, enhancer/promoter marks) can be obtained — the integration approach requires the multi-score feature matrix

## Inputs

- Annovar input / BED format file of candidate non-coding variants (e.g., input.bed)
- Pre-computed EIGEN, CADD, DANN, GWAVA and FATHMM scores genome-wide
- GNOMAD allele frequencies
- Known brain eQTLs from CommonMind (list of brain eqtls, e.g., -E input.bed)
- Enhancer/promoter region annotations for H3K4me3, H3K4me1, H3K27me3, H3K27ac from PsychENCODE and Roadmap Epigenomics
- Training dataset with class labels (e.g., deltaSVM data: 574 dsQTL-positive and 27,735 negative SNPs with MAF > 5%)

## Outputs

- ncDeepBrain variant score per non-coding variant (deep-learning probability scaled to 0-1)
- Imputed feature matrix (imputed_scores.csv) with no residual missing values in the five imputed score columns
- 11x11 pairwise Pearson correlation matrix of feature scores (correlation_matrix.csv, Additional file 1: Table S3)
- ROC curves and AUC values from ten-fold cross-validation on testing datasets

## How to apply

Annotate variants (e.g., with ANNOVAR from a BED/Annovar input file) and assemble an 11-feature matrix: EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTLs score (default 0.5 when no known brain eQTL from CommonMind), and binary enhancer/promoter indicators for H3K4me3, H3K4me1, H3K27me3, H3K27ac from PsychENCODE and Roadmap Epigenomics brain tissues. Impute missing DANN, CADD, EIGEN, GWAVA and FATHMM values with bPCA (bPCA.R), and balance imbalanced classes (e.g., 574 dsQTL positives vs 27,735 negatives with MAF > 5%) using the ROSE R library. Check pairwise Pearson correlations of the feature scores (11x11 matrix, Fig. 2(a)) before training to detect redundant predictors. Train a Keras-with-TensorFlow multilayer network (4 hidden layers, dropout regularization, sigmoid output scaled 0-1, negative log-likelihood loss optimized by SGD with momentum) to produce the ncDeepBrain score, and evaluate with ten-fold cross-validation ROC/AUC using scikit-learn.

## Related tools

- **TensorFlow** (Deep learning framework (via Keras) used to train the ncDeepBrain integration model with 4 hidden layers and sigmoid output) — https://github.com/WGLab/iMEGES
- **CADD** (One of the five general non-coding variant predictor scores integrated; missing values filled by bPCA) — http://cadd.gs.washington.edu/
- **DANN** (One of the five general non-coding variant predictor scores integrated; missing values filled by bPCA) — https://cbcl.ics.uci.edu/public_data/DANN/
- **GWAVA** (One of the five general non-coding variant predictor scores integrated; missing values filled by bPCA) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **FATHMM-MKL** (One of the five general non-coding variant predictor scores integrated; missing values filled by bPCA) — http://fathmm.biocompute.org.uk/
- **ANNOVAR** (Annotates candidate variants in BED/Annovar input format with the feature scores) — http://annovar.openbioinformatics.org/en/latest/
- **bPCA** (Bayesian Principal Component Analysis R code (bPCA.R) that imputes missing DANN, CADD, EIGEN, GWAVA and FATHMM values) — https://github.com/WGLab/iMEGES
- **deltaSVM** (Source of labeled training data (dataset 1: 574 dsQTL-positive vs 27,735 negative SNPs) for training the score-integration model)
- **iMEGES** (Pipeline in which ncDeepBrain constitutes step 1 (variant prioritization)) — https://github.com/WGLab/iMEGES

## Examples

```
python ncDeepBrain.py -f1 input.bed -E brain_eqtls.bed -B bPCA.R -o out.txt -buildver hg19
```

## Evaluation signals

- Classification AUC of the ncDeepBrain model should reach ~80% on the deltaSVM training dataset 1, ~91% on the GWAVA paper data, 75% on the PGC schizophrenia testing dataset 1, and 89% on the regulatory variants testing dataset 3
- After bPCA imputation, the DANN, CADD, EIGEN, GWAVA and FATHMM columns must contain zero missing values, and imputed_scores.csv must have the same row count as the input matrix
- The feature correlation output must be an 11x11 symmetric Pearson matrix with unit diagonal, coefficients in [-1, 1], consistent with Additional file 1: Table S3
- Imputed-score missingness rates must match the reported Additional file 1: Table S2 summary
- Model predictions must fall in the 0-1 range from the sigmoid output layer

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models (lack of large-scale training data for deep learning)
- Some procedures rely on imputed GWAS data, where hits may be proxy markers rather than true causal variants, making the model less reliable than ideal
- Non-coding variants are assigned to their closest gene (within 100 KB), a strategy that works well for promoters but may be less optimal for other types of regulatory elements

## Evidence

- [abstract] Leveraging deep neural network on the TensorFlow framework, we developed a computational tool, integrated Mental-disorder GEnome Score (iMEGES): "Leveraging deep neural network on the TensorFlow framework, we developed a computational tool, integrated Mental-disorder GEnome Score (iMEGES)"
- [readme] We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind"
- [results] The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [results] The two types of scores were integrated by a deep learning framework (ncDeepBrain for short in iMEGES).: "The two types of scores were integrated by a deep learning framework (ncDeepBrain for short in iMEGES)."
