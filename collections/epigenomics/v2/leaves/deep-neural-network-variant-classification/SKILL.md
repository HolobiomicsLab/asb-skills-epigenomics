---
name: deep-neural-network-variant-classification
description: Use when building or reproducing a supervised classifier for non-coding
  variant prioritization in mental-disorder genomes, where labeled disease vs. non-disease
  variant sets exist and AUC/ROC benchmarking against comparators (e.g., logistic
  regression ncLGBrain) is required.
when_to_use_negative:
- Input is already a ranked variant list or gene-level scores — this skill trains
  a variant-level classifier, not a gene prioritizer (that is iMEGES step 2)
- No labeled positive/negative variant sets are available — the network requires supervised
  training data such as GWAS-derived or validated regulatory variants
- Coding variant impact prediction — ncDeepBrain targets whole-genome non-coding variants
  affecting brain functions
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: TensorFlow
  role: Backend framework for the deep neural network used to integrate variant and
    tissue-related scores in ncDeepBrain
  repo: https://www.tensorflow.org/
- name: iMEGES
  role: Pipeline whose step 1 variant prioritization uses the ncDeepBrain deep neural
    network classifier
  repo: https://github.com/WGLab/iMEGES
- name: CADD
  role: Pre-computed deleteriousness feature score fed into the feature matrix
  repo: http://cadd.gs.washington.edu/
- name: DANN
  role: Pre-computed deleteriousness feature score fed into the feature matrix
  repo: https://cbcl.ics.uci.edu/public_data/DANN/
- name: GWAVA
  role: Regulatory variant feature score; GWAVA paper data is one training set
  repo: https://www.sanger.ac.uk/sanger/StatGen_Gwava
- name: FATHMM-MKL
  role: Functional impact feature score fed into the feature matrix
  repo: http://fathmm.biocompute.org.uk/
- name: deltaSVM
  role: Feature score; deltaSVM data constitutes training dataset 1
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:b8aa5e0d592fc342c123b2ee4375f6633570381c8ed75082cec3755d09ee9ffd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CADD
  - DANN
  - DeepSEA
  - FATHMM-MKL
  - GWAVA
  - TensorFlow
  - deltaSVM
  - iMEGES
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# deep-neural-network-variant-classification

## Summary

Train and evaluate a multilayer deep neural network (ncDeepBrain within iMEGES) that classifies non-coding variants as likely to affect brain function, by integrating variant functional scores, population frequency, brain eQTLs, and enhancer/promoter annotations, with performance measured by ten-fold cross-validation ROC/AUC.

## When to use

Use when building or reproducing a supervised classifier for non-coding variant prioritization in mental-disorder genomes, where labeled disease vs. non-disease variant sets exist and AUC/ROC benchmarking against comparators (e.g., logistic regression ncLGBrain) is required.

## When NOT to use

- Input is already a ranked variant list or gene-level scores — this skill trains a variant-level classifier, not a gene prioritizer (that is iMEGES step 2)
- No labeled positive/negative variant sets are available — the network requires supervised training data such as GWAS-derived or validated regulatory variants
- Coding variant impact prediction — ncDeepBrain targets whole-genome non-coding variants affecting brain functions

## Inputs

- Feature matrices of non-coding variants with variant scores (EIGEN, CADD, DANN, GWAVA, FATHMM/deltaSVM), GNOMAD frequency, brain eQTL overlap (CommonMind), and enhancer/promoter overlap (PsychENCODE, Roadmap Epigenomics H3K4me3/H3K4me1/H3K27me3/H3K27Ac)
- Labeled training/testing variant sets: PGC schizophrenia GWAS, PGC ASD GWAS, validated regulatory SNPs, Gelfman synonymous variants, deltaSVM training data, GWAVA paper data (HGMD-trained), 1000 Genomes negatives (allele frequency >= 1%)

## Outputs

- Trained ncDeepBrain Keras/TensorFlow model with sigmoid probability output scaled 0-1 per variant
- ROC curves (sensitivity vs specificity) and AUC values for ncDeepBrain and the ncLGBrain logistic-regression comparator per testing dataset
- ncDeepBrain variant scores used downstream in iMEGES gene prioritization

## How to apply

Assemble a feature matrix per training/testing set from pre-computed variant scores (EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, known brain eQTLs from the CommonMind project, and enhancer/promoter annotations (H3K4me3, H3K4me1, H3K27me3, H3K27Ac) from PsychENCODE and Roadmap Epigenomics. Impute missing feature values with bPCA (Bayesian Principal Component Analysis), as applied to DANN, CADD, EIGEN, GWAVA and FATHMM scores. Balance highly imbalanced classes (e.g., training dataset 1 has 574 positives vs 27,735 negatives) with the ROSE R library for both training and testing sets. Train the ncDeepBrain multilayer neural network (4 hidden layers, dropout regularization, sigmoid output layer scaled to 0-1, negative log-likelihood loss optimized by SGD with momentum) in Keras with a TensorFlow backend, then run ten-fold cross-validation on the four testing datasets (PGC schizophrenia, PGC ASD, validated regulatory SNPs, Gelfman synonymous variants), generating ROC curves and AUCs with python scikit-learn.

## Related tools

- **TensorFlow** (Backend framework for the deep neural network used to integrate variant and tissue-related scores in ncDeepBrain) — https://www.tensorflow.org/
- **iMEGES** (Pipeline whose step 1 variant prioritization uses the ncDeepBrain deep neural network classifier) — https://github.com/WGLab/iMEGES
- **CADD** (Pre-computed deleteriousness feature score fed into the feature matrix) — http://cadd.gs.washington.edu/
- **DANN** (Pre-computed deleteriousness feature score fed into the feature matrix) — https://cbcl.ics.uci.edu/public_data/DANN/
- **GWAVA** (Regulatory variant feature score; GWAVA paper data is one training set) — https://www.sanger.ac.uk/sanger/StatGen_Gwava
- **FATHMM-MKL** (Functional impact feature score fed into the feature matrix) — http://fathmm.biocompute.org.uk/
- **deltaSVM** (Feature score; deltaSVM data constitutes training dataset 1)

## Examples

```
python ncDeepBrain.py --help
```

## Evaluation signals

- Ten-fold cross-validation AUC on testing dataset 1 (PGC schizophrenia) should reproduce the reported ~75%; on testing dataset 3 (regulatory variants) ~89%
- Training-set AUCs should match reported values: ~80% on deltaSVM training dataset 1 and ~91% on the GWAVA paper data
- ROC curves (sensitivity vs specificity) generated with scikit-learn for both ncDeepBrain and the ncLGBrain logistic-regression comparator should be produced for each of the four testing datasets
- bPCA imputation should have been applied to all five predictor scores (EIGEN, CADD, DANN, GWAVA, FATHMM) with no missing values remaining in the feature matrix
- Class balance should be verified after ROSE resampling (e.g., training dataset 1 imbalanced at 574 positives vs 27,735 negatives must be rebalanced for both training and testing sets)

## Limitations

- It is challenging to obtain large-scale high quality data for training statistical models
- Some procedures rely on imputed GWAS data, where many hits may be proxy markers rather than true causal variants, making the model less reliable than ideal
- Improved performance over competing approaches is only expected when a large training dataset is available

## Evidence

- [abstract] Leveraging deep neural network on the TensorFlow framework, we developed a computational tool, integrated Mental-disorder GEnome Score (iMEGES): "Leveraging deep neural network on the TensorFlow framework, we developed a computational tool, integrated Mental-disorder GEnome Score (iMEGES)"
- [results] The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [results] we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores: "we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores"
- [readme] We used keras library with Tensorflow backend for deep learning in our study.: "We used keras library with Tensorflow backend for deep learning in our study."
- [abstract] it achieved improved performance than competing approaches when large training dataset is available: "it achieved improved performance than competing approaches when large training dataset is available"
