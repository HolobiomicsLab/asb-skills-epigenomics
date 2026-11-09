---
name: roc-auc-model-evaluation
description: Use when a trained variant- or gene-prioritization classifier (positive
  disease variants/genes vs. negatives) must be validated, and discriminative performance
  on held-out or cross-validated test sets must be reported as ROC curves with AUC
  values.
when_to_use_negative:
- When the goal is ranking or prioritization output quality rather than binary discrimination
  — e.g. the iMEGES gene score evaluation compares disease vs non-disease genes but
  proxies come from imputed GWAS P-values where hits may be proxy markers, not causal
  variants
- When class labels are absent or unreliable (no positives of known disease variants/genes),
  since ROC/AUC requires labeled positive and negative sets
- When input is raw sequencing data (VCF/BED) rather than model scores or an already-trained
  classifier — run the ncDeepBrain variant score step first
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: TensorFlow
  role: Backend for the Keras deep learning model (ncDeepBrain) whose classification
    performance is evaluated by ROC/AUC
  repo: https://www.tensorflow.org/
- name: Keras
  role: Library used to build the 4-hidden-layer, dropout-regularized network with
    sigmoid output that is cross-validated
  repo: https://keras.io/
- name: scikit-learn
  role: Used to generate ROC curves (sensitivity vs specificity) and compute AUC values
    over the testing sets
  repo: http://scikit-learn.org/stable/documentation.html
- name: ROSE
  role: R library that balances highly imbalanced positive/negative classes in training
    and testing sets before evaluation
- name: iMEGES
  role: The two-step pipeline (ncDeepBrain variant score, gene prioritization) whose
    variant and gene classification performance is benchmarked
  repo: https://github.com/WGLab/iMEGES
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2469-7
    title: 'iMEGES: integrated mental-disorder GEnome score by deep neural network
      for prioritizing the susceptibility genes for mental disorders in personal genomes'
  source_content_hash: sha256:4a63e0ab85235d0a4cd3284bf8aecc9ab7f5b957ab39aa2f3f7aab895bcfae8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CADD
  - DANN
  - DeepSEA
  - FATHMM-MKL
  - GTEx
  - GWAVA
  - Keras
  - ROSE
  - TensorFlow
  - deltaSVM
  - iMEGES
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# roc-auc-model-evaluation

## Summary

Evaluates a trained deep learning prioritization model (e.g. iMEGES/ncDeepBrain) by ten-fold cross-validation, generating sensitivity-vs-specificity ROC curves and AUC scores that quantify how well the model discriminates disease variants or genes from non-disease counterparts.

## When to use

Use when a trained variant- or gene-prioritization classifier (positive disease variants/genes vs. negatives) must be validated, and discriminative performance on held-out or cross-validated test sets must be reported as ROC curves with AUC values.

## When NOT to use

- When the goal is ranking or prioritization output quality rather than binary discrimination — e.g. the iMEGES gene score evaluation compares disease vs non-disease genes but proxies come from imputed GWAS P-values where hits may be proxy markers, not causal variants
- When class labels are absent or unreliable (no positives of known disease variants/genes), since ROC/AUC requires labeled positive and negative sets
- When input is raw sequencing data (VCF/BED) rather than model scores or an already-trained classifier — run the ncDeepBrain variant score step first

## Inputs

- Balanced feature matrix of variant scores (EIGEN, CADD, DANN, GWAVA, FATHMM), GNOMAD frequency, brain eQTLs, and enhancer/promoter annotations with class labels (disease vs non-disease)
- Trained model prediction scores (0-1 probabilities from the sigmoid output) for each testing sample
- Ten-fold cross-validation splits over the four testing datasets (PGC schizophrenia, PGC ASD, validated regulatory SNPs, Gelfman synonymous variants)

## Outputs

- ROC curve plots (sensitivity vs specificity) for ncDeepBrain and the ncLGBrain comparator on each testing dataset
- AUC values per testing dataset (e.g. 75% schizophrenia testing dataset 1, 89% testing dataset 3)
- Saved ROC curve figure files recording model comparison results

## How to apply

Split the labeled data into training and testing sets, balancing the highly imbalanced positive/negative classes (e.g. 574 positives vs 27,735 negatives in training dataset 1) with the ROSE R library for both training and testing. Train the multilayer neural network (4 hidden layers, dropout regularization, sigmoid output scaled 0-1, negative log-likelihood loss via SGD with momentum) in Keras with a TensorFlow backend, then run ten-fold cross-validation on the testing datasets. Generate ROC curves (sensitivity vs specificity) and compute AUC values with python scikit-learn, comparing the deep model against a logistic-regression comparator such as ncLGBrain. Validate results by checking computed AUCs against expected values (e.g. ncDeepBrain: 80% on training dataset 1 deltaSVM data, 91% on GWAVA data, 75% on PGC schizophrenia testing dataset 1, 89% on regulatory variants testing dataset 3).

## Related tools

- **TensorFlow** (Backend for the Keras deep learning model (ncDeepBrain) whose classification performance is evaluated by ROC/AUC) — https://www.tensorflow.org/
- **Keras** (Library used to build the 4-hidden-layer, dropout-regularized network with sigmoid output that is cross-validated) — https://keras.io/
- **scikit-learn** (Used to generate ROC curves (sensitivity vs specificity) and compute AUC values over the testing sets) — http://scikit-learn.org/stable/documentation.html
- **ROSE** (R library that balances highly imbalanced positive/negative classes in training and testing sets before evaluation)
- **iMEGES** (The two-step pipeline (ncDeepBrain variant score, gene prioritization) whose variant and gene classification performance is benchmarked) — https://github.com/WGLab/iMEGES

## Examples

```
python -c "from sklearn.metrics import roc_curve, auc; fpr,tpr,_=roc_curve(y_true, y_score); print('AUC', auc(fpr,tpr))"
```

## Evaluation signals

- Computed AUCs match paper values: ncDeepBrain ~80% on training dataset 1 (deltaSVM data), ~91% on GWAVA paper data, 75% on PGC schizophrenia testing dataset 1, 89% on testing dataset 3
- Gene-prioritization cross-validation AUCs are modest (~57% schizophrenia, ~58% ASD) when discriminating disease genes from non-disease genes
- ROC axes are sensitivity vs specificity over 0-1, produced with ten-fold cross-validation rather than a single holdout
- Class balance is verified before evaluation: positives and negatives were ROSE-resampled (e.g. 574 vs 27,735 becomes balanced) for both training and testing
- Deep-model AUC is compared against the logistic-regression ncLGBrain comparator on the same test sets

## Limitations

- Imputed GWAS data means many positive hits are proxy markers rather than true causal variants, making the model less reliable than ideal
- Lack of large-scale, high-quality training data limits performance of the deep learning models
- Highly imbalanced classes require ROSE resampling; without it, AUC estimates would be biased by the excess of negatives
- Performance is dataset-dependent: ncDeepBrain improves over competing approaches only when a large training dataset is available

## Evidence

- [results] ROC/AUC evaluation procedure: "we generated ROC curves for discriminating disease variants from non-disease variants (normal) in the testing set and calculated AUC scores"
- [results] Expected baseline AUC: "The classification AUC value of ncDeepBrain is 80% on the deltaSVM data training dataset 1"
- [readme] Deep learning stack used for the evaluated model: "We used keras library with Tensorflow backend for deep learning in our study."
- [abstract] Dataset-dependence of performance: "it achieved improved performance than competing approaches when large training dataset is available"
