---
name: classification-regression-metric-evaluation
description: Use when trained sgRNA editing-efficiency predictors on CRISPRoff/CRISPRi/CRISPRa
  screening data must be scored on a 10% held-out test split, compared across feature-combination
  ablations via AUC and Spearman correlation with s.e.m. and t-tests.
when_to_use_negative:
- Input is a single model's raw predictions without a defined classification threshold/label
  scheme or efficiency score — AUC and Spearman require paired binary labels and continuous
  efficiency values, respectively
- The goal is feature attribution or model interpretation (use Tree SHAP/DeepSHAP
  instead) rather than held-out performance estimation
- Data cannot support a 9:1 train/test split with 5-fold cross-validation (e.g., too
  few sgRNAs per condition for stable error bars)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: Deep learning framework whose trained sgRNA activity prediction models (seq-only
    and seq+epigenetic variants) are evaluated with AUC and Spearman correlation on
    the 10% held-out test data
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend used as the deep learning model framework whose predictions feed the
    ROC-AUC and Spearman correlation evaluations
- name: scikit-learn 0.24.2
  role: Backend for machine learning algorithms used to compute classification (ROC-AUC)
    and regression (Spearman) performance metrics
- name: GPyOpt
  role: Bayesian optimization package (30 initial random points, 300 acquisitions)
    producing the tuned hyperparameter settings whose resulting models are evaluated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:8e46567d9fbfb577a27dd8e8c2a614843e6d9cc6012c397cc503218c05577d2d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# classification-regression-metric-evaluation

## Summary

Evaluate sgRNA editing-efficiency prediction models (e.g., EpiCas-DL CNN variants) on held-out test data using ROC-AUC for the classification schema and Spearman correlation for the regression schema, with error bars from cross-validation and significance testing between feature-combination conditions.

## When to use

Use when trained sgRNA editing-efficiency predictors on CRISPRoff/CRISPRi/CRISPRa screening data must be scored on a 10% held-out test split, compared across feature-combination ablations via AUC and Spearman correlation with s.e.m. and t-tests.

## When NOT to use

- Input is a single model's raw predictions without a defined classification threshold/label scheme or efficiency score — AUC and Spearman require paired binary labels and continuous efficiency values, respectively
- The goal is feature attribution or model interpretation (use Tree SHAP/DeepSHAP instead) rather than held-out performance estimation
- Data cannot support a 9:1 train/test split with 5-fold cross-validation (e.g., too few sgRNAs per condition for stable error bars)

## Inputs

- Trained EpiCas-DL CNN model checkpoints per feature-combination condition (TensorFlow 2.4.1 backend)
- Held-out 10% test split of the CRISPRoff sgRNA dataset encoded as 11 x 40 matrices (sequence and epigenetic channels) with binary labels and efficiency scores
- Per-fold cross-validation predictions from the 5-fold hyperparameter-optimization runs

## Outputs

- Per-condition ROC-AUC values for the classification schema
- Per-condition Spearman correlation values for the regression schema
- Cross-validation error bars (s.e.m.) per condition
- feature_ablation_results.csv: one row per feature combination, columns for AUC and Spearman correlation plus s.e.m., with Student's t-test comparisons

## How to apply

Randomly divide each condition's data into a 90% training and 10% testing set (with 5-fold cross-validation on training data for hyperparameter optimization via GPyOpt Bayesian optimization). Evaluate each trained model on the held-out 10% by computing the area under the ROC curve (AUC) for the binary classification schema and Spearman correlation for the regression schema, attaching error bars derived from cross-validation (s.e.m.). Aggregate per-condition metrics across the feature-combination ablations (sequence-only, sequence + single epigenetic feature, sequence + all epigenetic features), compare conditions with Student's t-test, and write a summary table (one row per feature combination, columns for AUC and Spearman plus s.e.m.) to feature_ablation_results.csv. Validate that the all-features EpiCas-DL model attains the highest AUC/Spearman among all conditions, reproducing the ordering reported in Fig. 1c-d.

## Related tools

- **EpiCas-DL** (Deep learning framework whose trained sgRNA activity prediction models (seq-only and seq+epigenetic variants) are evaluated with AUC and Spearman correlation on the 10% held-out test data) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend used as the deep learning model framework whose predictions feed the ROC-AUC and Spearman correlation evaluations)
- **scikit-learn 0.24.2** (Backend for machine learning algorithms used to compute classification (ROC-AUC) and regression (Spearman) performance metrics)
- **GPyOpt** (Bayesian optimization package (30 initial random points, 300 acquisitions) producing the tuned hyperparameter settings whose resulting models are evaluated)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_seq
```

## Evaluation signals

- feature_ablation_results.csv exists and contains one row for each ablation condition (seq-only, seq+TSS/nucleosome, seq+ATAC, seq+RNA, seq+methylation) plus the all-features condition
- The all-features (seq + all epigenetic channels) model attains the highest AUC and highest Spearman correlation among all conditions, reproducing the ordering reported in Fig. 1c-d
- Each metric value is reported with a cross-validation error bar (s.e.m.) and conditions are compared by Student's t-test
- Evaluation is restricted to the 10% held-out test set, consistent with the 9:1 random split and 5-fold cross-validation protocol on the training portion
- Metrics are exactly ROC-AUC for the classification schema and Spearman correlation for the regression schema (no other primary metrics substituted)

## Limitations

- Small CRISPRoff screening datasets (mainly HEK293T and K562) increase overfitting risk and limit generalization of the evaluated performance estimates
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance measured on the testing datasets
- Gene silencing and activation screening datasets may contain noise in the measured sgRNA knock-down or knock-up efficacy, which propagates into the Spearman/AUC values
- Incomplete epigenetic feature information limited model performance, capping the achievable AUC/Spearman for the all-features condition

## Evidence

- [results] calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation: "We randomly divided the dataset into a training dataset (90%) and a testing dataset (10%) and calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [methods] 9:1 split with 5-fold cross-validation: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [methods] Metric backends: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [readme] Tool scope from README: "EpiCas-DL is a deeplearning based prediction tool for the sgRNA editing effects of CRISPR/dCas editing system."
