---
name: cross-validated-roc-auc-evaluation
description: Use when a deep learning predictor (e.g., ncDeepBrain variant score)
  has been trained on labeled positives/negatives and you must quantify classification
  performance via cross-validated ROC curves and AUC.
when_to_use_negative:
- Input is already a set of computed AUC/ROC values and no trained model or labeled
  test data are available — this skill evaluates model predictions, not existing statistics
- The task is downstream gene prioritization (iMEGES step 2 scoring with RVIS, GTEx,
  haploinsufficiency and Phenolyzer disease scores) rather than validating the variant-score
  classifier
- No labeled positives/negatives exist for the variant set, making ROC/AUC discrimination
  undefined
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow
  role: Backend framework for the Keras deep neural network whose predictions are
    evaluated by ROC/AUC
  repo: https://github.com/WGLab/iMEGES
- name: Keras
  role: Model API used with the TensorFlow backend to produce the sigmoid-scored predictions
    being cross-validated
  repo: https://github.com/WGLab/iMEGES
- name: python scikit
  role: Generates the ROC curves (sensitivity vs specificity) and AUC scores from
    cross-validated predictions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:dbbd26b91a7db4ca4e92177997eee2258ae7af59c2da0f6675bca589580e0e60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - CADD
  - DANN
  - FATHMM-MKL
  - GWAVA
  - Keras
  - TensorFlow
  - deltaSVM
  - python scikit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-validated-roc-auc-evaluation

## Summary

Evaluates a trained variant/gene prioritization model (e.g., the ncDeepBrain network in iMEGES) by ten-fold cross-validation, generating ROC curves (sensitivity vs. specificity) and AUC scores for discriminating disease variants from non-disease variants using python scikit.

## When to use

Use when a deep learning predictor (e.g., ncDeepBrain variant score) has been trained on labeled positives/negatives and you must quantify classification performance via cross-validated ROC curves and AUC.

## When NOT to use

- Input is already a set of computed AUC/ROC values and no trained model or labeled test data are available — this skill evaluates model predictions, not existing statistics
- The task is downstream gene prioritization (iMEGES step 2 scoring with RVIS, GTEx, haploinsufficiency and Phenolyzer disease scores) rather than validating the variant-score classifier
- No labeled positives/negatives exist for the variant set, making ROC/AUC discrimination undefined

## Inputs

- Labeled variant feature matrix (574 positive dsQTL variants and 27,735 negative SNPs with MAF > 5%, annotated with 11 features via ANNOVAR, ROSE-balanced)
- Trained Keras/TensorFlow ncDeepBrain model with sigmoid output predictions in 0–1 range
- Testing-set labels (disease/non-disease variant status)

## Outputs

- Ten-fold cross-validated ROC curve (sensitivity vs. specificity) generated with python scikit
- Classification AUC score (expected ~80% on deltaSVM training dataset 1)

## How to apply

Split the labeled training data (e.g., deltaSVM dataset 1 with positive dsQTL variants and negative SNPs) into ten folds and repeat training/testing so every fold serves once as the held-out testing set. For each fold, feed held-out features (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTLs, enhancer/promoter indicators) through the trained Keras/TensorFlow model and collect the sigmoid outputs in the 0–1 range. Generate ROC curves plotting sensitivity vs. specificity with python scikit, then compute the AUC over the pooled or per-fold predictions. Judge success by reproducing the reported benchmark: the ncDeepBrain classification AUC should be approximately 80% on the deltaSVM training dataset 1 (Additional file 1: Figure S2(a)). Compare against competing scores under the same cross-validation protocol to establish relative performance.

## Related tools

- **TensorFlow** (Backend framework for the Keras deep neural network whose predictions are evaluated by ROC/AUC) — https://github.com/WGLab/iMEGES
- **Keras** (Model API used with the TensorFlow backend to produce the sigmoid-scored predictions being cross-validated) — https://github.com/WGLab/iMEGES
- **python scikit** (Generates the ROC curves (sensitivity vs specificity) and AUC scores from cross-validated predictions)

## Examples

```
python ncDeepBrain.py --help
```

## Evaluation signals

- The ten-fold cross-validated classification AUC of ncDeepBrain reproduces the reported ~80% on the deltaSVM training dataset 1 (Additional file 1: Figure S2(a))
- ROC curves plot sensitivity (y-axis) vs. specificity (x-axis) for discriminating disease variants from non-disease (normal) variants in the testing set
- AUC scores are computed per fold and reported consistently across the ten cross-validation folds
- Model sigmoid outputs fall in the 0–1 range before ROC threshold sweep, as designed for the ncDeepBrain score
- Performance comparison against competing approaches (CADD, DANN, GWAVA, FATHMM-MKL, deltaSVM, DeepSEA, GTEx) is run under the identical cross-validation protocol

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models, limiting generalization of the measured AUC
- Some procedures rely on imputed GWAS data, and many hits represent proxy markers rather than true causal variants, making the model less reliable than ideal
- Performance conclusions are dataset-dependent: iMEGES achieved improved performance than competing approaches only when large training dataset is available

## Evidence

- [results] we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores: "we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores"
- [results] ncDeepBrain achieved classification AUC of 80% on training dataset 1 (deltaSVM data): "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [readme] We used keras library with Tensorflow backend for deep learning in our study.: "We used keras library with Tensorflow backend for deep learning in our study."
- [abstract] iMEGES achieved improved performance than competing approaches when large training dataset is available: "it achieved improved performance than competing approaches when large training dataset is available"
