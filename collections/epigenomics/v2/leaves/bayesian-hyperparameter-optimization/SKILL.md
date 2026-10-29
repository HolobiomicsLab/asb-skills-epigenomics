---
name: bayesian-hyperparameter-optimization
description: Use when tuning hyperparameters (e.g., convolution filter counts/sizes,
  fully-connected layer widths, dropout) of the EpiCas-DL TensorFlow CNN, and grid/random
  search is deemed unreliable.
when_to_use_negative:
- When the model is a tree-based method (e.g., Gradient Boosting, Random Forest, XGBoost)
  rather than the EpiCas-DL deep CNN — the article's Bayesian optimization targeted
  the deep learning hyperparameters
- When training data is too small to support 5-fold cross-validation scoring of each
  acquisition, increasing overfitting risk (a limitation the authors note for the
  relatively small CRISPRoff screen datasets)
edam_operation: http://edamontology.org/operation_3892
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: GPyOpt
  role: Bayesian optimization package used to search the EpiCas-DL CNN hyperparameter
    space (30 initial points, 300 acquisitions) via 5-fold cross-validation
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend in which the EpiCas-DL CNN whose hyperparameters are optimized is
    defined and trained
  repo: https://github.com/yangqianq/EpiCas-DL
- name: scikit-learn 0.24.2
  role: Backend for machine learning algorithms used alongside TensorFlow during model
    training/evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:15fcd8dcfff82aff53268bb4257c01f0b6f512e8b19d0645362b01fa3e7021be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python
  - TensorFlow 2.4.1
  - scikit-learn
  - scikit-learn 0.24.2
  edam_operation: http://edamontology.org/operation_3892
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# bayesian-hyperparameter-optimization

## Summary

Bayesian (Gaussian-process-based) hyperparameter tuning of the EpiCas-DL CNN using the GPyOpt package, replacing random or grid search when optimizing deep learning architectures for sgRNA editing-efficiency prediction.

## When to use

Use when tuning hyperparameters (e.g., convolution filter counts/sizes, fully-connected layer widths, dropout) of the EpiCas-DL TensorFlow CNN, and grid/random search is deemed unreliable.

## When NOT to use

- When the model is a tree-based method (e.g., Gradient Boosting, Random Forest, XGBoost) rather than the EpiCas-DL deep CNN — the article's Bayesian optimization targeted the deep learning hyperparameters
- When training data is too small to support 5-fold cross-validation scoring of each acquisition, increasing overfitting risk (a limitation the authors note for the relatively small CRISPRoff screen datasets)

## Inputs

- Encoded sgRNA feature matrices (11 x 40: one-hot sequence, nucleosome/TSS-distance, chromatin accessibility, DNA methylation, gene expression channels) with 9:1 train/test split
- EpiCas-DL CNN architecture hyperparameter search space (filter sizes/counts, FC layer units, dropout rate) defined for GPyOpt

## Outputs

- Optimal hyperparameter configuration for the EpiCas-DL model per dataset/condition
- Trained EpiCas-DL models (e.g., CRISPRoff_seq, CRISPRoff_epi, CRISPRa_seq, CRISPRa_epi) evaluated by ROC-AUC and Spearman correlation on the held-out 10%

## How to apply

After splitting the sgRNA dataset (CRISPRoff, CRISPRi, or CRISPRa activity-score matrices encoding 4 one-hot sequence channels plus epigenetic channels) 9:1 into training and testing sets, run 5-fold cross-validation on the training data to score candidate hyperparameter settings. Configure GPyOpt with 30 initial random exploration points and then execute 300 acquisitions to approximate the global optimum of the validation objective. Apply the selected hyperparameters to the final CNN (multi-size convolution filters, max pooling, three fully connected layers of 80/60/40 units, dropout 0.4 after batch normalization, ReLU activations, linear output) built in TensorFlow 2.4.1. Rationale: the article reports GPyOpt 'has been demonstrated to be more reliable than random grid search' for obtaining globally optimal deep learning settings.

## Related tools

- **GPyOpt** (Bayesian optimization package used to search the EpiCas-DL CNN hyperparameter space (30 initial points, 300 acquisitions) via 5-fold cross-validation) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend in which the EpiCas-DL CNN whose hyperparameters are optimized is defined and trained) — https://github.com/yangqianq/EpiCas-DL
- **scikit-learn 0.24.2** (Backend for machine learning algorithms used alongside TensorFlow during model training/evaluation)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- 5-fold cross-validation objective on the 90% training split improves monotonically toward the optimum across the 300 GPyOpt acquisitions
- Final tuned model outperforms baseline algorithms (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) on the 10% held-out test set by AUC/Spearman
- Feature-ablation ordering reproduced: the seq + all-epigenetic-features model attains the highest AUC/Spearman versus sequence-only or seq + single-feature models
- Tuned architecture matches the reported EpiCas-DL design: five convolution filter sizes (30 filters each of 1–5 nt), FC layers 80/60/40, dropout 0.4 after batch normalization

## Limitations

- Optimization cost: 30 random initial points plus 300 acquisitions per model may be prohibitive for large search spaces or repeated across many conditions
- Relatively small CRISPRoff screens (mainly HEK293T and K562) increase overfitting risk, which cross-validation-based optimization cannot fully eliminate
- Incomplete epigenetic feature information limited final model performance regardless of hyperparameter tuning

## Evidence

- [methods] GPyOpt more reliable than random search: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [methods] 300 acquisitions for global optimum: "300 acquisitions were executed in an attempt to obtain the global optimal value"
- [methods] 9:1 split with 5-fold CV: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
