---
name: epigenetic-feature-ablation-analysis
description: Use when you need to test whether integrating all epigenetic features
  with sgRNA sequence improves CRISPRoff/CRISPRi/CRISPRa efficiency prediction versus
  sequence-only or sequence-plus-one-epigenetic-feature models.
when_to_use_negative:
- The goal is feature interpretation/importance ranking rather than ablation performance
  comparison — use Tree SHAP/DeepSHAP with XGBoost instead
- Input data lacks epigenetic feature tracks (no ATAC, methylation, RNA expression,
  or TSS coordinates) so only sequence-only models are feasible
- The task is benchmarking EpiCas-DL against external predictors (DeepHF, C-RNNcrispr,
  Gradient Boosting, Random Forest) rather than internal feature ablation
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0654
tools:
- name: EpiCas-DL
  role: CNN framework trained per feature-combination condition to measure predictive
    contribution of each epigenetic channel
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend implementing the multi-filter-size convolutional models for each ablation
    condition
- name: GPyOpt
  role: Bayesian hyperparameter optimization (30 initial points, 300 acquisitions)
    per ablation condition via 5-fold cross-validation
- name: scikit-learn 0.24.2
  role: ROC-AUC/Spearman evaluation metrics and cross-validation utilities for condition
    comparison
- name: pybedtools
  role: Python scripting to build per-sgRNA epigenetic feature matrices for the channel
    subsets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:48b11cfbd9232c44ca871568712d0b5593c0e1465237a0006ef47d917de59d2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python
  - TensorFlow 2.4.1
  - pybedtools
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# epigenetic-feature-ablation-analysis

## Summary

Ablation analysis skill for EpiCas-DL that systematically removes or adds epigenetic feature channels (nucleosome positioning/TSS distance, chromatin accessibility, DNA methylation, gene expression) to quantify their contribution to sgRNA editing-efficiency prediction relative to sequence-only encoding.

## When to use

Use when you need to test whether integrating all epigenetic features with sgRNA sequence improves CRISPRoff/CRISPRi/CRISPRa efficiency prediction versus sequence-only or sequence-plus-one-epigenetic-feature models.

## When NOT to use

- The goal is feature interpretation/importance ranking rather than ablation performance comparison — use Tree SHAP/DeepSHAP with XGBoost instead
- Input data lacks epigenetic feature tracks (no ATAC, methylation, RNA expression, or TSS coordinates) so only sequence-only models are feasible
- The task is benchmarking EpiCas-DL against external predictors (DeepHF, C-RNNcrispr, Gradient Boosting, Random Forest) rather than internal feature ablation

## Inputs

- Preprocessed CRISPRoff (and CRISPRi/CRISPRa) sgRNA datasets encoded as 11 x 40 matrices with 4 sequence, 4 TSS/nucleosome, 1 ATAC, 1 methylation, and 1 RNA expression channels
- Min-max normalized epigenetic feature tracks (ATACdb accessibility, DNA methylation, gene expression, TSS distance)
- Activity-score-normalized sgRNA efficiency labels (phenotype score normalized by absolute average of top 3 sgRNAs per gene)

## Outputs

- feature_ablation_results.csv with one row per feature combination (sequence-only, each Seq+single-feature, all features) and columns for AUC and Spearman correlation plus s.e.m.
- Per-condition trained CNN checkpoints from TensorFlow
- Bayesian-optimized hyperparameter configurations per condition from GPyOpt

## How to apply

Encode each sgRNA target site as an 11 x 40 matrix (4 one-hot sequence channels, 4 nucleosome-positioning/TSS-distance channels, 1 chromatin accessibility, 1 DNA methylation, 1 gene expression) and select channel subsets per ablation condition: sequence-only (4 channels), Seq+TSS, Seq+ATAC, Seq+RNA, Seq+Methylation, and the full EpiCas-DL input. Split each condition 9:1 into training/testing and run 5-fold cross-validation on the training set with GPyOpt Bayesian optimization (30 initial random points, 300 acquisitions) to tune the CNN, whose architecture uses five convolution filter sizes (30 filters each of 1-, 2-, 3-, 4-, and 5-nt), max pooling, three fully connected layers (80/60/40 units), dropout 0.4 after batch normalization, and a linear output. Train each condition in TensorFlow 2.4.1, evaluate on the held-out 10% with ROC-AUC (classification) and Spearman correlation (regression), attach error bars from 10-fold cross-validation (s.e.m.), and compare conditions by Student's t-test. The rationale is to isolate the marginal predictive value of each epigenetic modality, since the paper found sites with DNA methylation, high chromatin accessibility, and high gene expression show higher editing efficiency.

## Related tools

- **EpiCas-DL** (CNN framework trained per feature-combination condition to measure predictive contribution of each epigenetic channel) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend implementing the multi-filter-size convolutional models for each ablation condition)
- **GPyOpt** (Bayesian hyperparameter optimization (30 initial points, 300 acquisitions) per ablation condition via 5-fold cross-validation)
- **scikit-learn 0.24.2** (ROC-AUC/Spearman evaluation metrics and cross-validation utilities for condition comparison)
- **pybedtools** (Python scripting to build per-sgRNA epigenetic feature matrices for the channel subsets)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- feature_ablation_results.csv exists and contains a row for every ablation condition plus the all-features (EpiCas-DL) condition
- The all-features model attains the highest AUC and Spearman correlation among all conditions, reproducing the ordering in Fig. 1c-d
- Each condition uses the same 9:1 train/test split and 5-fold cross-validated hyperparameter search (30 random points, 300 acquisitions), so comparisons are matched
- Uncertainty is reported as s.e.m. from 10-fold cross-validation and condition differences pass Student's t-test
- Per-sgRNA input matrices are 11 x 40 with correct channel counts for each subset (4 for sequence-only, 8 for sequence-plus-one-feature)

## Limitations

- Incomplete epigenetic feature information limits EpiCas-DL model performance, weakening ablation conclusions for missing modalities
- Available CRISPRoff screening datasets are relatively small and mainly in HEK293T and K562 cell lines, increasing overfitting risk and limiting generalization of ablation findings
- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are considered; more complex architectures (RNNs, GANs, RL) were not explored, so ablation rankings may not transfer to other model classes

## Evidence

- [methods] we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [methods] we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [methods] Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [results] We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [readme] The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
