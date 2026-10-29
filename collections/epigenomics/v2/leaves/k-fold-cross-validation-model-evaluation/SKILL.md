---
name: k-fold-cross-validation-model-evaluation
description: Use when benchmarking regression or classification models on sgRNA screening
  datasets (e.g., 111,682 sgRNAs x 520 genes) and you need held-out performance estimates
  comparable across models via cross-validated metrics.
when_to_use_negative:
- The task is only sequence-only sgRNA scoring without held-out evaluation — use the
  pre-trained CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi models for prediction
  instead of re-running the CV benchmark
- Datasets span different CRISPRoff vs. CRISPRi systems without accounting for the
  differences the authors warn may affect transfer of prediction performance between
  systems
- Screening data is too small (risk of overfitting noted for the limited CRISPRoff
  datasets mainly from HEK293T and K562) for a stable 10-fold split
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: CNN regressor evaluated via 10-fold cross-validation against baselines
  repo: https://github.com/yangqianq/EpiCas-DL
- name: GPyOpt
  role: Bayesian optimization package used to tune hyperparameters during cross-validation
- name: TensorFlow 2.4.1
  role: Backend for training the EpiCas-DL deep learning model on each fold
- name: scikit-learn 0.24.2
  role: Backend for the machine learning baseline algorithms (Gradient Boosting, Random
    Forest) fit on identical inputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:972273220f360c219d891c2c2dc70eb3b0b3dbd967fe37bbb2c0cad1d7889ac4
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# k-fold cross-validation model evaluation

## Summary

Random 9:1 train/test splitting with 10-fold cross-validation to evaluate sgRNA activity prediction models (EpiCas-DL and comparators) by Spearman correlation, MSE, and cosine similarity on CRISPRoff/CRISPRi/CRISPRa screening datasets.

## When to use

Use when benchmarking regression or classification models on sgRNA screening datasets (e.g., 111,682 sgRNAs x 520 genes) and you need held-out performance estimates comparable across models via cross-validated metrics.

## When NOT to use

- The task is only sequence-only sgRNA scoring without held-out evaluation — use the pre-trained CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi models for prediction instead of re-running the CV benchmark
- Datasets span different CRISPRoff vs. CRISPRi systems without accounting for the differences the authors warn may affect transfer of prediction performance between systems
- Screening data is too small (risk of overfitting noted for the limited CRISPRoff datasets mainly from HEK293T and K562) for a stable 10-fold split

## Inputs

- Gene-level normalized sgRNA activity scores from CRISPRoff_tiling (111,682 sgRNAs, ±2.5 kb of TSS of 520 genes) or other CRISPRi/CRISPRa screening datasets
- 11x40 encoded feature matrices per sgRNA (4 one-hot sequence channels + nucleosome positioning, chromatin accessibility, DNA methylation, gene expression)
- Min-max normalized epigenetic feature channels

## Outputs

- Per-fold and mean Spearman correlation between predicted and observed sgRNA activity
- Per-fold and mean mean-squared-error (MSE) and cosine similarity
- Student's t-test comparison of EpiCas-DL vs. comparator models across the 10 folds
- GPyOpt-optimized hyperparameter configuration

## How to apply

Randomly split the encoded dataset (11x40 feature matrices: one-hot sequence channels plus nucleosome positioning, chromatin accessibility, DNA methylation, and gene expression) into 90% training and 10% testing. Run k-fold cross-validation on the training portion (10-fold for reported results; 5-fold was used during GPyOpt Bayesian hyperparameter tuning with 30 initial random points and 300 acquisitions). Train the EpiCas-DL CNN (five 30-filter groups of 1-5 nt, MaxPooling, FC layers 80/60/40, dropout 0.4 after batch norm, ReLU activations, linear output) and fit comparators (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) on identical inputs with default configurations. Evaluate on each held-out fold with Spearman correlation, MSE, and cosine similarity, then compare models using Student's t-test.

## Related tools

- **EpiCas-DL** (CNN regressor evaluated via 10-fold cross-validation against baselines) — https://github.com/yangqianq/EpiCas-DL
- **GPyOpt** (Bayesian optimization package used to tune hyperparameters during cross-validation)
- **TensorFlow 2.4.1** (Backend for training the EpiCas-DL deep learning model on each fold)
- **scikit-learn 0.24.2** (Backend for the machine learning baseline algorithms (Gradient Boosting, Random Forest) fit on identical inputs)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Reported Spearman correlation, MSE, and cosine similarity for EpiCas-DL on CRISPRoff_tiling are reproduced within reasonable variance across the 10 folds
- Student's t-test shows EpiCas-DL's regression performance is significantly better than Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr (consistent with Fig. 2b-2c)
- Train/test split is a random 9:1 division and CV folds are computed only on the 90% training portion
- Epigenetic channels are min-max normalized and sequences are one-hot encoded into 11x40 matrices before any fold is drawn

## Limitations

- Incomplete epigenetic feature information (only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression) limited model performance
- Small CRISPRoff screening datasets mainly from HEK293T and K562 cell lines increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy

## Evidence

- [methods] 9:1 split with 5-fold CV during hyperparameter optimization: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [methods] GPyOpt Bayesian optimization preferred over grid search: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [results] 10-fold CV with Spearman correlation and AUC on 9:1 splits: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [results] Pooled 90% training across five datasets with 10% held-out testing: "we combined 90 % of the five independent datasets for model training and examined the model performance on the remaining 10 % of the data"
- [readme] README invocation of the pre-trained models that this CV evaluation supports: "python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq"
