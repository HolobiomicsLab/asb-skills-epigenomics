---
name: ten-fold-cross-validation-evaluation
description: Use when evaluating a deep-learning variant-prioritization score (e.g.
  ncDeepBrain) against labeled disease/non-disease variant sets, where ROC curves
  and AUC on cross-validated testing data are the required performance metrics.
when_to_use_negative:
- The goal is gene-level prioritization (iMEGES step 2 score with GTEx, RVIS, haploinsufficiency,
  Phenolyzer features) rather than non-coding variant classification
- The input is already a trained model output or rank list, with no labeled positive/negative
  testing set to compute ROC/AUC
- Class balance and feature imputation (ROSE, bPCA) cannot be applied because the
  raw variant-score annotations are unavailable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: iMEGES
  role: Source tool whose ncDeepBrain variant score is being cross-validated and benchmarked
  repo: https://github.com/WGLab/iMEGES
- name: TensorFlow
  role: Backend for the Keras deep neural network trained and evaluated in the ten-fold
    cross-validation
  repo: https://www.tensorflow.org/
- name: Keras
  role: Library used to build and train the ncDeepBrain multilayer network being evaluated
  repo: https://keras.io/
- name: scikit-learn
  role: Python library used to generate ROC curves and compute AUC scores during cross-validation
  repo: http://scikit-learn.org/stable/documentation.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:999f33f898d4a845aa0ace6fd3d6d336351f0ca6505c0226a311e798905aaf1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/iMEGES
  tools:
  - CADD
  - DANN
  - DeepSEA
  - FATHMM-MKL
  - GWAVA
  - Keras
  - TensorFlow
  - deltaSVM
  - iMEGES
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# ten-fold-cross-validation-evaluation

## Summary

Benchmark the ncDeepBrain non-coding variant score in iMEGES by training its Keras/TensorFlow multilayer neural network on four feature matrices and running ten-fold cross-validation on held-out testing datasets, generating ROC curves and AUC values to verify classification performance against the published figures.

## When to use

Use when evaluating a deep-learning variant-prioritization score (e.g. ncDeepBrain) against labeled disease/non-disease variant sets, where ROC curves and AUC on cross-validated testing data are the required performance metrics.

## When NOT to use

- The goal is gene-level prioritization (iMEGES step 2 score with GTEx, RVIS, haploinsufficiency, Phenolyzer features) rather than non-coding variant classification
- The input is already a trained model output or rank list, with no labeled positive/negative testing set to compute ROC/AUC
- Class balance and feature imputation (ROSE, bPCA) cannot be applied because the raw variant-score annotations are unavailable

## Inputs

- Feature matrices for four training datasets and four testing datasets (variant scores: EIGEN, CADD, DANN, GWAVA, FATHMM; GNOMAD frequency; brain eQTLs; enhancer/promoter marks H3K4me3, H3K4me1, H3K27me3, H3K27Ac)
- Training dataset 1 (deltaSVM data), GWAVA paper data, PGC schizophrenia and PGC ASD imputed GWAS SNPs, validated regulatory SNPs, Gelfman synonymous variants
- Trained ncDeepBrain model (Keras with TensorFlow backend) and ncLGBrain logistic-regression comparator

## Outputs

- ROC curves (sensitivity vs specificity) per testing dataset per model
- AUC values for ncDeepBrain and ncLGBrain on each of the four testing datasets
- Ten-fold cross-validation performance distribution confirming reported AUCs (~75% schizophrenia dataset 1, ~89% regulatory variants dataset 3)

## How to apply

Assemble a feature matrix per training/testing dataset from pre-computed variant scores (EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind, and enhancer/promoter annotations (H3K4me3, H3K4me1, H3K27me3, H3K27Ac) from PsychENCODE and Roadmap Epigenomics. Impute missing feature values with bPCA, then balance highly imbalanced classes (e.g. 574 positives vs 27,735 negatives in training dataset 1) using the ROSE R library on both training and testing sets. Train the ncDeepBrain network (4 hidden layers, dropout regularization, sigmoid output scaled 0-1, negative log-likelihood loss optimized by SGD with momentum) via Keras with a TensorFlow backend. Run ten-fold cross-validation on the four testing datasets — PGC schizophrenia, PGC ASD, validated regulatory SNPs, and Gelfman synonymous variants — generating ROC curves (sensitivity vs specificity) and computing AUC with python scikit-learn for both ncDeepBrain and the logistic-regression comparator ncLGBrain. Validate by checking that computed AUCs match reported values (~80% on training dataset 1 deltaSVM data, ~91% on GWAVA paper data, 75% on testing dataset 1, 89% on testing dataset 3).

## Related tools

- **iMEGES** (Source tool whose ncDeepBrain variant score is being cross-validated and benchmarked) — https://github.com/WGLab/iMEGES
- **TensorFlow** (Backend for the Keras deep neural network trained and evaluated in the ten-fold cross-validation) — https://www.tensorflow.org/
- **Keras** (Library used to build and train the ncDeepBrain multilayer network being evaluated) — https://keras.io/
- **scikit-learn** (Python library used to generate ROC curves and compute AUC scores during cross-validation) — http://scikit-learn.org/stable/documentation.html

## Examples

```
python ncDeepBrain.py --help
```

## Evaluation signals

- Computed ncDeepBrain AUC reaches ~80% on training dataset 1 (deltaSVM data) and ~91% on the GWAVA paper data
- Ten-fold cross-validation AUCs reproduce reported testing results: ~75% on PGC schizophrenia testing dataset 1 and ~89% on the regulatory variants testing dataset 3
- ROC curves are generated for discriminating disease variants from non-disease (normal) variants on each testing set
- ncLGBrain (logistic regression comparator) AUCs are computed on the same folds for side-by-side comparison
- ROSE-balanced testing folds show the imbalanced original class structure (e.g. 574 positives vs 27,735 negatives before balancing) is mitigated without train/test leakage

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models
- Some procedures rely on imputed GWAS data, where many hits represent proxy markers rather than true causal variants, making the model less reliable than ideal
- Each non-coding variant is associated to its closest gene, which may be suboptimal for non-promoter regulatory elements

## Evidence

- [results] ROC curves and AUC computation on testing sets: "we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores"
- [results] Reported ncDeepBrain AUC on training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [readme] Deep learning framework used for the model being evaluated: "We used keras library with Tensorflow backend for deep learning in our study."
- [readme] Feature integration for the ncDeepBrain score being cross-validated: "We integrated five different scores from different predictors (such as EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from CommonMind"
