---
name: imbalanced-class-resampling
description: Use when building a supervised classifier (e.g. a Keras/TensorFlow variant-scoring
  network) from a training set where positives are orders of magnitude rarer than
  negatives — in iMEGES, 574 dsQTL positives vs 27,735 negatives — so the sigmoid
  output is not dominated by the majority class.
when_to_use_negative:
- The training classes are already balanced or nearly so — ROSE synthetic oversampling
  would add noise without benefit and distort the natural class prior.
- 'Resampling the held-out test/validation folds: evaluation AUCs must be computed
  on the original imbalanced distribution; ROSE is only for the training data feeding
  the Keras network.'
- You need calibrated per-variant probabilities reflecting population frequencies
  — resampled training changes the effective prior, so sigmoid outputs must be interpreted
  as ranking scores (ncDeepBrain scores), not calibrated probabilities.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: ROSE
  role: R library used to balance the imbalanced training classes (574 positives vs
    27,735 negatives) via synthetic oversampling before deep network training
- name: bPCA
  role: Bayesian PCA R routine run BEFORE resampling to impute missing EIGEN/CADD/DANN/GWAVA/FATHMM
    feature values
- name: Keras
  role: Defines the 4-hidden-layer network with sigmoid output trained on the ROSE-balanced
    matrix
  repo: https://keras.io/
- name: TensorFlow
  role: Backend engine for the Keras deep learning model
  repo: https://www.tensorflow.org/
- name: python scikit (scikit-learn)
  role: Generates ROC curves (sensitivity vs specificity) and AUC during ten-fold
    cross-validation of the resampled model
  repo: http://scikit-learn.org/stable/documentation.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:a7668c6fb4b475c540d33a100a07a0215308aec4e0627ead697b3719983d9f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - Keras
  - ROSE
  - TensorFlow
  - bPCA
  - deltaSVM
  - python scikit
  - python scikit (scikit-learn)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# imbalanced-class-resampling

## Summary

Balances severely skewed positive/negative class ratios in a training feature matrix using the ROSE R library before fitting a deep neural network — here applied to ncDeepBrain's 574 positive dsQTL SNPs vs 27,735 negative SNPs (MAF > 5%) used to train the iMEGES non-coding variant score.

## When to use

Use when building a supervised classifier (e.g. a Keras/TensorFlow variant-scoring network) from a training set where positives are orders of magnitude rarer than negatives — in iMEGES, 574 dsQTL positives vs 27,735 negatives — so the sigmoid output is not dominated by the majority class.

## When NOT to use

- The training classes are already balanced or nearly so — ROSE synthetic oversampling would add noise without benefit and distort the natural class prior.
- Resampling the held-out test/validation folds: evaluation AUCs must be computed on the original imbalanced distribution; ROSE is only for the training data feeding the Keras network.
- You need calibrated per-variant probabilities reflecting population frequencies — resampled training changes the effective prior, so sigmoid outputs must be interpreted as ranking scores (ncDeepBrain scores), not calibrated probabilities.

## Inputs

- Annotated variant feature matrix (e.g. ANNOVAR-annotated dsQTL training SNPs: 574 positive, 27,735 negative with MAF > 5%) with 11 columns (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTL score, H3K4me3/H3K4me1/H3K27me3/H3K27ac binary indicators)
- bPCA-imputed feature matrix (missing DANN/CADD/EIGEN/GWAVA/FATHMM values filled before resampling)

## Outputs

- Class-balanced training set produced by the ROSE R library (synthetic minority samples approximating parity with the 27,735 negatives)
- Trained Keras/TensorFlow ncDeepBrain model with sigmoid output scaled to 0–1
- Ten-fold cross-validation ROC curves and classification AUC (~80% expected on deltaSVM training dataset 1)

## How to apply

First assemble the feature matrix (here 11 features: EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTL score defaulting to 0.5, and binary H3K4me3/H3K4me1/H3K27me3/H3K27ac enhancer/promoter indicators) and impute missing predictor values with bPCA fitted on non-missing genome-wide scores. Then apply the ROSE R library to synthetically balance the 574 positive vs 27,735 negative classes before training. Train the balanced data on the multilayer Keras network (4 hidden layers, sigmoid output) by minimizing binary cross-entropy with SGD plus momentum. Finally, evaluate with ten-fold cross-validation and ROC/AUC computed in python scikit; in this paper the ncDeepBrain model reached ~80% AUC on the deltaSVM training dataset 1. The rationale is that without resampling, the ~48:1 negative-to-positive ratio would bias the network toward the negative class and collapse sensitivity for pathogenic non-coding variants.

## Related tools

- **ROSE** (R library used to balance the imbalanced training classes (574 positives vs 27,735 negatives) via synthetic oversampling before deep network training)
- **bPCA** (Bayesian PCA R routine run BEFORE resampling to impute missing EIGEN/CADD/DANN/GWAVA/FATHMM feature values)
- **Keras** (Defines the 4-hidden-layer network with sigmoid output trained on the ROSE-balanced matrix) — https://keras.io/
- **TensorFlow** (Backend engine for the Keras deep learning model) — https://www.tensorflow.org/
- **python scikit (scikit-learn)** (Generates ROC curves (sensitivity vs specificity) and AUC during ten-fold cross-validation of the resampled model) — http://scikit-learn.org/stable/documentation.html

## Examples

```
python ncDeepBrain.py --bed1 input.bed --bed2 brain_eqtls.bed --bPCA bPCA.R -o out.txt --hg hg38
```

## Evaluation signals

- After ROSE resampling, positive and negative class counts in the training set are (near-)equal instead of 574 vs 27,735.
- Ten-fold cross-validation ROC curves and AUC reproduce the reported ~80% classification AUC on deltaSVM training dataset 1 (Additional file 1: Figure S2(a)).
- ROC curves (sensitivity vs specificity) are generated with python scikit, as in the paper's evaluation step.
- Feature imputation (bPCA) is completed and no NaN values remain in DANN, CADD, EIGEN, GWAVA, FATHMM columns before balancing.
- Trained sigmoid outputs fall in the 0–1 range and rank positive (dsQTL) variants above negatives on held-out folds.

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models; with small training sets iMEGES (and its resampling) underperforms compared to competitors when rivals have larger training data.
- ROSE synthetic oversampling cannot create new biological information — the positive class remains only 574 real dsQTL SNPs, so synthetic samples may not capture full variant heterogeneity.
- Resampling inflates apparent minority performance if applied before train/test splitting, so cross-validation folds must be handled carefully.

## Evidence

- [methods] Balance the imbalanced classes (574 positives vs 27,735 negatives) with the ROSE R library.: [evidence span withheld — anchor status: not_found]
- [methods] Impute missing values of DANN, CADD, EIGEN, GWAVA and FATHMM features using bPCA: [evidence span withheld — anchor status: not_found]
- [results] The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [readme] We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind and enhancer/promoters"
- [discussion] it is challenging to obtain large-scale high quality data for training statistical models.: "First, it is challenging to obtain large-scale high quality data for training statistical models."
