---
name: roc-auc-cross-validation-evaluation
description: Use when binary-classifying sgRNAs as highly active vs. inactive (e.g.,
  phenotype score < −0.1 labeled '1') and you need cross-validated ROC-AUC comparison
  of a CNN against classical ML baselines on sequence-plus-epigenetic features.
when_to_use_negative:
- The task is regression of continuous sgRNA activity scores (e.g., Spearman correlation)
  rather than binary highly-active/inactive classification
- Input sgRNAs lack positional coordinates or cell-type epigenetic tracks, so the
  11 x 40 sequence-plus-epigenetic matrix cannot be assembled
- Evaluating on datasets outside CRISPRoff/CRISPRi/CRISPRa screening data where the
  phenotype-score cutoff and normalization were validated
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: CNN classification model being evaluated; trained on one-hot sequence plus
    epigenetic features with GPyOpt-tuned hyperparameters
  repo: https://github.com/yangqianq/EpiCas-DL
- name: scikit-learn 0.24.2
  role: Backend for the Gradient Boosting, Random Forest, and cross-validation/ROC-AUC
    computation
- name: TensorFlow 2.4.1
  role: Backend for training and evaluating the EpiCas-DL deep learning model
- name: GPyOpt
  role: Bayesian optimization of EpiCas-DL hyperparameters via 5-fold cross-validation
    on the training split
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:a6ac509db8a4ac56542eea9c0aad90d9501d9f60e5ad315de1601a26021ee9ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - TensorFlow 2.4.1
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# roc-auc-cross-validation-evaluation

## Summary

Benchmark sgRNA activity prediction models by computing ROC-AUC under cross-validation on a 90%/10% train-test split, comparing a deep-learning model (e.g., EpiCas-DL CNN) against Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr baselines fit with scikit-learn.

## When to use

Use when binary-classifying sgRNAs as highly active vs. inactive (e.g., phenotype score < −0.1 labeled '1') and you need cross-validated ROC-AUC comparison of a CNN against classical ML baselines on sequence-plus-epigenetic features.

## When NOT to use

- The task is regression of continuous sgRNA activity scores (e.g., Spearman correlation) rather than binary highly-active/inactive classification
- Input sgRNAs lack positional coordinates or cell-type epigenetic tracks, so the 11 x 40 sequence-plus-epigenetic matrix cannot be assembled
- Evaluating on datasets outside CRISPRoff/CRISPRi/CRISPRa screening data where the phenotype-score cutoff and normalization were validated

## Inputs

- sgRNA sequence (40 bp) with one-hot encoding
- min–max normalized epigenetic feature channels (nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression)
- Binarized sgRNA activity labels (phenotype score < −0.1 → '1')
- CRISPRoff_tiling dataset (111,682 sgRNAs within ± 2.5 kb of the TSS of 520 genes)

## Outputs

- Per-model ROC-AUC scores under 10-fold cross-validation (EpiCas-DL vs. DeepHF, C-RNNcrispr, Random Forest, Gradient Boosting)
- Classification-schema comparison table (e.g., auc_comparison_table.csv)

## How to apply

Assemble the 11 x 40 input matrix (one-hot encoded 40 bp sgRNA sequence as 4 channels plus min–max normalized epigenetic channels: nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression), binarize labels using the published cutoff (phenotype scores lower than −0.1 as highly active '1'), and randomly divide into 90% training and 10% testing data. Train the EpiCas-DL CNN (five filter groups of 30 filters at 1–5 nt widths, MaxPooling, fully connected 80/60/40 units, dropout 0.4 after batch normalization, linear output) in TensorFlow, tuning hyperparameters with GPyOpt Bayesian optimization (30 initial points, 300 acquisitions, 5-fold cross-validation on the training split). Fit Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr with default configurations on the identical input using scikit-learn as the ML backend. Evaluate all five models on the held-out test set by computing ROC-AUC under 10-fold cross-validation, averaging per-fold scores with s.e.m., and assemble the comparison table.

## Related tools

- **EpiCas-DL** (CNN classification model being evaluated; trained on one-hot sequence plus epigenetic features with GPyOpt-tuned hyperparameters) — https://github.com/yangqianq/EpiCas-DL
- **scikit-learn 0.24.2** (Backend for the Gradient Boosting, Random Forest, and cross-validation/ROC-AUC computation)
- **TensorFlow 2.4.1** (Backend for training and evaluating the EpiCas-DL deep learning model)
- **GPyOpt** (Bayesian optimization of EpiCas-DL hyperparameters via 5-fold cross-validation on the training split)

## Examples

```
from sklearn.model_selection import cross_val_score; scores = cross_val_score(model, X_epi_seq, y_binary, cv=10, scoring='roc_auc')
```

## Evaluation signals

- Comparison table contains ROC-AUC entries for all five models (EpiCas-DL, DeepHF, C-RNNcrispr, Random Forest, Gradient Boosting)
- Baseline AUCs match reported values: DeepHF 0.751, C-RNNcrispr 0.790, Random Forest 0.767, Gradient Boosting 0.755
- EpiCas-DL achieves a competitive or superior AUC (0.879 reported on CRISPRoff_tiling)
- Train/test split is 9:1 as described, with 5-fold cross-validation for hyperparameter optimization and 10-fold cross-validation for evaluation
- Per-fold ROC-AUC scores are reported with s.e.m. averaged across folds

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance
- Small CRISPRoff screening datasets, mainly in HEK293T and K562 cell lines, may increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets

## Evidence

- [results] We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC): "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC)"
- [methods] Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [methods] we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [methods] To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr): "To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr)"
