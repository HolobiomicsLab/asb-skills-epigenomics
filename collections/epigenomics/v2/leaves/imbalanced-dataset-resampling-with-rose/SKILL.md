---
name: imbalanced-dataset-resampling-with-rose
description: Use when training or evaluating a variant/gene classifier on datasets
  with extreme positive-to-negative imbalance (e.g., PGC schizophrenia GWAS-derived
  sets with hundreds of positives and tens of thousands of negatives), prior to fitting
  the model and computing ROC/AUC.
when_to_use_negative:
- When the positive/negative classes are already balanced — resampling would discard
  or duplicate variants needlessly
- When the goal is to report prevalence-realistic precision metrics on the untouched
  original testing distribution rather than cross-validated ROC/AUC
- When the task is feature imputation of missing scores (use bPCA via the -B/--bPCA
  option of ncDeepBrain.py instead)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ROSE
  role: R library used to balance the imbalanced positive/negative classes in training
    and testing sets
- name: iMEGES
  role: Pipeline whose ncDeepBrain training/testing data are resampled before model
    fitting and ROC/AUC evaluation
  repo: https://github.com/WGLab/iMEGES
- name: scikit-learn
  role: Used downstream to generate ROC curves and compute AUC scores on the resampled
    testing sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:a56dd2a7d1630169cd5f34314abdfa4af3f79f6dbdb9396c6cb4d0fe5c707cc8
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
  - ROSE
  - TensorFlow
  - deltaSVM
  - iMEGES
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# imbalanced-dataset-resampling-with-rose

## Summary

Balance highly imbalanced positive/negative variant datasets using the ROSE R library before training and testing the ncDeepBrain deep neural network, so that class skew (e.g., 574 disease positives vs 27,735 negatives in training dataset 1) does not dominate the learned model or the resulting ROC/AUC evaluation.

## When to use

Use when training or evaluating a variant/gene classifier on datasets with extreme positive-to-negative imbalance (e.g., PGC schizophrenia GWAS-derived sets with hundreds of positives and tens of thousands of negatives), prior to fitting the model and computing ROC/AUC.

## When NOT to use

- When the positive/negative classes are already balanced — resampling would discard or duplicate variants needlessly
- When the goal is to report prevalence-realistic precision metrics on the untouched original testing distribution rather than cross-validated ROC/AUC
- When the task is feature imputation of missing scores (use bPCA via the -B/--bPCA option of ncDeepBrain.py instead)

## Inputs

- Imbalanced training dataset 1 (deltaSVM data; 574 positives vs 27,735 negatives)
- Feature matrices with variant scores (EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, brain eQTLs, and PsychENCODE/Roadmap Epigenomics enhancer/promoter annotations
- Testing datasets: PGC schizophrenia, PGC ASD, validated regulatory SNPs, Gelfman synonymous variants

## Outputs

- Balanced training and testing variant sets with approximately equal positive and negative class counts
- Resampled matrices suitable for ten-fold cross-validation of the ncDeepBrain and ncLGBrain models

## How to apply

Identify the positive (disease-associated, e.g., PGC schizophrenia or PGC ASD GWAS variants) and negative (e.g., 1000 Genomes SNVs with allele frequency >= 1%) classes and quantify the imbalance. Apply ROSE (Random Over-Sampling Examples) from the ROSE R library to generate a balanced training set, and importantly also resample the testing sets (PGC schizophrenia, PGC ASD, validated regulatory SNPs, Gelfman synonymous variants) before running the ten-fold cross-validation. Rationale: with 574 positives vs 27,735 negatives, an unregularized neural network would be biased toward the majority class and AUC estimates would be misleading. Run resampling before feeding features (EIGEN, CADD, DANN, GWAVA, FATHMM, GNOMAD frequency, brain eQTLs, H3K4me3/H3K4me1/H3K27me3/H3K27Ac enhancer/promoter annotations) to the Keras/TensorFlow ncDeepBrain model, alongside bPCA imputation of missing DANN/CADD/EIGEN/GWAVA/FATHMM scores. Verify downstream validity by checking that the reported AUCs (ncDeepBrain ~80% on training dataset 1, 75% on schizophrenia testing dataset 1, 89% on regulatory variants testing dataset 3) are reproduced.

## Related tools

- **ROSE** (R library used to balance the imbalanced positive/negative classes in training and testing sets)
- **iMEGES** (Pipeline whose ncDeepBrain training/testing data are resampled before model fitting and ROC/AUC evaluation) — https://github.com/WGLab/iMEGES
- **scikit-learn** (Used downstream to generate ROC curves and compute AUC scores on the resampled testing sets)

## Examples

```
Rscript -e 'library(ROSE); balanced <- ROSE(label ~ ., data = train_set, seed = 42)$data'
```

## Evaluation signals

- After resampling, positive and negative class counts should be approximately equal (contrast with the original 574 vs 27,735 ratio in training dataset 1)
- Ten-fold cross-validation ROC/AUC on the balanced testing sets should reproduce the paper's values: ~80% on training dataset 1 (deltaSVM data), 75% on schizophrenia testing dataset 1, and 89% on regulatory variants testing dataset 3
- AUC curves (sensitivity vs specificity) should be computed consistently for both ncDeepBrain and the logistic-regression ncLGBrain comparator
- Missing feature values should be handled by bPCA before/alongside resampling, with no NaNs remaining in the balanced matrix

## Limitations

- Resampling both training and testing sets means evaluated performance does not reflect the true class prevalence of the original GWAS-derived data
- Imputed GWAS hits used to define positives may be proxy markers rather than true causal variants, limiting how meaningful the balanced evaluation is
- The article notes it is challenging to obtain large-scale high quality data for training such statistical models, so resampling cannot compensate for small absolute numbers of positives

## Evidence

- [other] Balance the highly imbalanced positive/negative classes (e.g., training dataset 1 has 574 positives vs 27,735 negatives) using the ROSE R library for both training and testing sets.: [evidence span withheld — anchor status: not_found]
- [results] The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [results] we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores: "we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores"
- [readme] The bPCA R code to do the imputation: "-B bPCA.R, --bPCA bPCA.R
                        The bPCA R code to do the imputation"
