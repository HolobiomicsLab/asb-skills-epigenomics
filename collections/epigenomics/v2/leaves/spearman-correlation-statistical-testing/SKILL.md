---
name: spearman-correlation-statistical-testing
description: Use when comparing regression or classification models for sgRNA activity
  prediction on datasets like CRISPRoff_tiling (111,682 sgRNAs, 520 genes) and you
  need rank-based correlation plus significance testing across CV folds.
when_to_use_negative:
- Your goal is feature attribution or mechanistic interpretation (use Tree SHAP /
  DeepSHAP with XGBoost instead, as in the paper's feature-identification workflow)
- You need binary editing-call classification metrics on data with no continuous activity
  scores — use AUC-based evaluation rather than Spearman correlation
- Predictions and observations come from different sgRNAs or unmatched samples, so
  a rank correlation is meaningless
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Backend (v0.24.2) for the comparator machine learning algorithms whose fold-level
    predictions are compared by Spearman correlation and Student's t-test
- name: TensorFlow
  role: Backend (v2.4.1) for training the EpiCas-DL CNN regressor whose predictions
    are evaluated per fold
- name: GPyOpt
  role: Bayesian optimization package used to tune EpiCas-DL hyperparameters (30 initial
    random points, 300 acquisitions) before fold-level metric evaluation
- name: EpiCas-DL
  role: Deep learning model producing the sgRNA activity predictions evaluated by
    Spearman correlation, MSE, and cosine similarity
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:2859318123f1a579ea6fbcd82d4e30381dec63824b933d6e99fc4c9e5563b654
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - TensorFlow
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Spearman correlation statistical testing

## Summary

Evaluate sgRNA activity prediction models by computing Spearman rank correlations between predicted and observed sgRNA gene-repression/activation efficiencies across cross-validation folds, and test performance differences (e.g., EpiCas-DL vs. Random Forest, Gradient Boosting, DeepHF, C-RNNcrispr) with Student's t-test, supported by MSE and cosine similarity.

## When to use

Use when comparing regression or classification models for sgRNA activity prediction on datasets like CRISPRoff_tiling (111,682 sgRNAs, 520 genes) and you need rank-based correlation plus significance testing across CV folds.

## When NOT to use

- Your goal is feature attribution or mechanistic interpretation (use Tree SHAP / DeepSHAP with XGBoost instead, as in the paper's feature-identification workflow)
- You need binary editing-call classification metrics on data with no continuous activity scores — use AUC-based evaluation rather than Spearman correlation
- Predictions and observations come from different sgRNAs or unmatched samples, so a rank correlation is meaningless

## Inputs

- Held-out test-fold predictions from EpiCas-DL (CNN regressor, TensorFlow 2.4.1) on CRISPRoff_tiling (111,682 sgRNAs, ±2.5 kb of TSS of 520 genes)
- Held-out test-fold predictions from comparator models (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr via scikit-learn 0.24.2)
- Gene-level normalized sgRNA activity scores (phenotype scores normalized by absolute average of top 3 sgRNAs per gene)
- Fold assignments from 10-fold cross-validation on the 90% training portion

## Outputs

- Per-fold Spearman correlation coefficients between predicted and observed sgRNA activity
- Per-fold mean squared error and cosine similarity values
- Student's t-test statistics/p-values comparing EpiCas-DL vs. each baseline across the 10 folds
- Model ranking summary analogous to Fig. 2b-2c

## How to apply

Load sgRNA-level activity scores and model predictions on the held-out 10% test split of the 9:1 train/test division, then compute Spearman correlation between predicted and observed efficiencies on each of the 10 cross-validation folds. For regression tasks (e.g., EpiCas-DL on CRISPRoff_tiling), report Spearman correlation, mean squared error, and cosine similarity per fold and compare against the four comparator algorithms (Gradient Boosting, Random Forest, DeepHF, C-RNNcrispr) fit on identical sequence-plus-epigenetic (nucleosome positioning, chromatin accessibility, DNA methylation, gene expression) inputs. Test fold-level metric differences between the proposed model and baselines using Student's t-test. Accept the comparison when the proposed model's Spearman correlation is significantly better (consistent with Fig. 2b-2c of the EpiCas-DL paper), not merely nominally higher. For classification-flavored evaluations the same pipeline applies with AUC replacing Spearman correlation.

## Related tools

- **scikit-learn** (Backend (v0.24.2) for the comparator machine learning algorithms whose fold-level predictions are compared by Spearman correlation and Student's t-test)
- **TensorFlow** (Backend (v2.4.1) for training the EpiCas-DL CNN regressor whose predictions are evaluated per fold)
- **GPyOpt** (Bayesian optimization package used to tune EpiCas-DL hyperparameters (30 initial random points, 300 acquisitions) before fold-level metric evaluation)
- **EpiCas-DL** (Deep learning model producing the sgRNA activity predictions evaluated by Spearman correlation, MSE, and cosine similarity) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Spearman correlation is computed on each of the 10 held-out test folds, not on training data
- EpiCas-DL's Spearman correlation, MSE, and cosine similarity on CRISPRoff_tiling match reported values within reasonable variance and are consistent with Fig. 2b-2c
- Student's t-test across folds shows EpiCas-DL's regression performance is significantly better than Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr
- All models are compared on identical sequence-plus-epigenetic feature inputs (4 one-hot sequence channels plus nucleosome positioning, chromatin accessibility, DNA methylation, gene expression)
- Activity scores are gene-level normalized (phenotype score divided by absolute average of top 3 sgRNAs per gene) before evaluation

## Limitations

- Small CRISPRoff screening datasets, mainly in HEK293T and K562 cell lines, may increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance when testing across systems
- Gene silencing and activation screening datasets may contain noise in measuring sgRNA knock-down or knock-up efficacy
- Incomplete epigenetic feature information (only nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression) limited model performance

## Evidence

- [results] calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation: "calculated the area under the receiver operating characteristic (ROC) curve (AUC) or Spearman correlation"
- [methods] Model comparison with four algorithms: "To compare the performance of EpiCas-DL with other four algorithms (Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr)"
- [methods] 9:1 split with cross-validation: "we randomly divided this dataset into training and testing datasets by 9:1 and performed 5-folds cross-validation"
- [readme] README prediction invocation: "python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057"
- [methods] scikit-learn and TensorFlow backends: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
