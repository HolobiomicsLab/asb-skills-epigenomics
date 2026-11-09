---
name: baseline-model-benchmark-comparison
description: Use when a trained predictor (e.g. a CNN regressor for sgRNA activity)
  must be benchmarked head-to-head against comparator algorithms on identical train/test
  splits, with significance testing on correlation, MSE, or cosine similarity metrics.
when_to_use_negative:
- The task is a classification benchmark (AUC/ROC comparison) rather than sgRNA activity
  regression — the article uses Spearman correlation, MSE, and cosine similarity for
  the regression setting
- The proposed model and baselines were trained on different training splits or different
  feature encodings — comparisons on non-identical inputs are invalid
- There is insufficient test-set size or fold count to support a Student's t-test
  across folds
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow 2.4.1
  role: Backend for the deep learning models being benchmarked, including the EpiCas-DL
    CNN regressor
  repo: https://github.com/yangqianq/EpiCas-DL
- name: scikit-learn 0.24.2
  role: Backend providing the baseline machine learning algorithms (Gradient Boosting,
    Random Forest) compared against EpiCas-DL
- name: GPyOpt
  role: Bayesian optimization package used to tune the EpiCas-DL hyperparameters before
    the benchmark comparison (30 initial points, 300 acquisitions)
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:5fe6ecf980744633cf27266d7bbd8c8c54aaef52ff3a1ef29487b1d1168d58c4
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
n_claims: 11
---

# baseline-model-benchmark-comparison

## Summary

Reproduce and evaluate a proposed deep learning model (EpiCas-DL CNN regressor) against competing state-of-the-art baselines (Random Forest, Gradient Boosting, DeepHF, C-RNNcrispr) on the same held-out data, using Spearman correlation, MSE, and cosine similarity with a Student's t-test to establish statistically significant superiority.

## When to use

Use when a trained predictor (e.g. a CNN regressor for sgRNA activity) must be benchmarked head-to-head against comparator algorithms on identical train/test splits, with significance testing on correlation, MSE, or cosine similarity metrics.

## When NOT to use

- The task is a classification benchmark (AUC/ROC comparison) rather than sgRNA activity regression — the article uses Spearman correlation, MSE, and cosine similarity for the regression setting
- The proposed model and baselines were trained on different training splits or different feature encodings — comparisons on non-identical inputs are invalid
- There is insufficient test-set size or fold count to support a Student's t-test across folds

## Inputs

- CRISPRoff_tiling dataset: 111,682 sgRNAs within ±2.5 kb of the TSS of 520 genes, with gene-level normalized activity scores
- 11x40 one-hot encoded feature matrices (4 sequence channels + nucleosome positioning, chromatin accessibility, DNA methylation, gene expression channels)
- Trained EpiCas-DL model and comparator models (Random Forest, Gradient Boosting, DeepHF, C-RNNcrispr)

## Outputs

- Per-fold Spearman correlation coefficients between predicted and observed sgRNA activity for each model
- Per-fold mean squared error (MSE) and cosine similarity values for each model
- Student's t-test p-values comparing EpiCas-DL vs. each comparator across the 10 folds

## How to apply

Fit the proposed model (EpiCas-DL: five convolutional filter groups of 30 filters each spanning 1-5 nt, MaxPooling, three fully connected layers of 80/60/40 units, dropout 0.4 after batch normalization, ReLU activations, linear output) in TensorFlow 2.4.1, with hyperparameters tuned via GPyOpt Bayesian optimization (30 initial random points, 300 acquisitions). Fit each comparator (Gradient Boosting, Random Forest via scikit-learn 0.24.2, plus DeepHF and C-RNNcrispr) with default configurations on the identical sequence-plus-epigenetic 11x40 encoded inputs (4 one-hot sequence channels plus nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression). Evaluate all models on the same held-out test folds (10-fold cross-validation over a 9:1 train/test split for CRISPRoff_tiling, 111,682 sgRNAs) using Spearman correlation, mean squared error, and cosine similarity. Use Student's t-test across the folds to decide whether the proposed model's regression performance is significantly better than each baseline, mirroring Fig. 2b-2c.

## Related tools

- **TensorFlow 2.4.1** (Backend for the deep learning models being benchmarked, including the EpiCas-DL CNN regressor) — https://github.com/yangqianq/EpiCas-DL
- **scikit-learn 0.24.2** (Backend providing the baseline machine learning algorithms (Gradient Boosting, Random Forest) compared against EpiCas-DL)
- **GPyOpt** (Bayesian optimization package used to tune the EpiCas-DL hyperparameters before the benchmark comparison (30 initial points, 300 acquisitions)) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Spearman correlation values for EpiCas-DL on CRISPRoff_tiling match the paper's reported values within reasonable variance across 10 cross-validation folds
- MSE and cosine similarity for EpiCas-DL are the best among all five models, consistent with Fig. 2b-2c
- Student's t-test shows EpiCas-DL's regression performance is significantly better than Random Forest, Gradient Boosting, DeepHF, and C-RNNcrispr
- All models were trained and evaluated on identical 9:1 train/test splits of the same 111,682-sgRNA dataset

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered; incomplete epigenetic feature information limited model performance
- Small CRISPRoff screening datasets mainly in HEK293T and K562 cell lines increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets

## Evidence

- [methods] Model comparison scope: "To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr)"
- [results] Split and metric definition: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [methods] Tool backends: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [methods] Bayesian hyperparameter tuning: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
