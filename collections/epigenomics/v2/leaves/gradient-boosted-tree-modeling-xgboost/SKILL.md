---
name: gradient-boosted-tree-modeling-xgboost
description: Use when a tabular sgRNA feature matrix (one-hot encoded sequences plus
  epigenetic features like nucleosome positioning, ATAC-seq accessibility, DNA methylation,
  RNA expression) needs both activity prediction and feature-importance interpretation
  via Tree SHAP.
when_to_use_negative:
- The analysis goal is pure deep-learning sequence prediction with one-hot encoded
  inputs rather than interpretable tree-based feature attribution
- The input is raw sequencing data rather than an already-assembled sgRNA feature/activity
  matrix
- Only a single sgRNA needs an activity score — use the prebuilt EpiCas-DL models
  (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi) via example.py instead of
  retraining
edam_operation: http://edamontology.org/operation_3934
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: XGBoost
  role: Gradient-boosted tree algorithm fitted on the epigenetic feature matrix; its
    SHAP values are used for feature identification
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TreeExplainer
  role: Interpreter computing mean |SHAP| values on the training dataset to rank features
    by average impact on model output magnitude
- name: GPyOpt
  role: Bayesian optimization package used instead of random/grid search to tune model
    hyperparameters over ~300 acquisitions with 5-fold cross-validation
- name: Python 3.6
  role: Custom-script environment (with pybedtools) used to extract and assemble the
    sgRNA epigenetic feature matrix
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:ede9facdda794a0fa41778fbd741ff82625b7e0c84d614760df14e3f65308846
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - GPyOpt
  - Python 3.6
  - TreeExplainer
  - XGBoost
  edam_operation: http://edamontology.org/operation_3934
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gradient-boosted-tree-modeling-xgboost

## Summary

Train an XGBoost gradient-boosted tree model on sgRNA sequence/epigenetic feature matrices to predict CRISPR editing activity and to expose interpretable Tree SHAP feature importances, as done in the EpiCas-DL framework for CRISPRoff, CRISPRi, and CRISPRa datasets.

## When to use

Use when a tabular sgRNA feature matrix (one-hot encoded sequences plus epigenetic features like nucleosome positioning, ATAC-seq accessibility, DNA methylation, RNA expression) needs both activity prediction and feature-importance interpretation via Tree SHAP.

## When NOT to use

- The analysis goal is pure deep-learning sequence prediction with one-hot encoded inputs rather than interpretable tree-based feature attribution
- The input is raw sequencing data rather than an already-assembled sgRNA feature/activity matrix
- Only a single sgRNA needs an activity score — use the prebuilt EpiCas-DL models (CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi) via example.py instead of retraining

## Inputs

- sgRNA sequence and epigenetic feature matrix (nucleosome positioning/distance to TSS, chromatin accessibility from ATACdb ATAC-seq, DNA methylation, RNA expression) assembled from CRISPRoff/CRISPRi/CRISPRa screening datasets
- Normalized sgRNA activity scores (phenotype scores normalized by the absolute average score of the top 3 sgRNAs per gene)

## Outputs

- Trained XGBoost model with tuned hyperparameters
- Ranked feature-importance table of top 30 features with mean |SHAP| values (table_s4_shap_feature_importance.csv)
- Model performance metrics (AUC or Spearman correlation on held-out 10% test set)

## How to apply

Assemble the feature matrix from sgRNA sequences (one-hot encoded image-like scheme) and min-max normalized epigenetic features extracted by custom Python 3.6/pybedtools scripts. Randomly split into 90% training and 10% testing, and tune hyperparameters with Bayesian optimization (GPyOpt, ~300 acquisitions) using 5-fold cross-validation on the training portion. Fit the XGBoost gradient-boosted tree model, then compute mean |SHAP| values (average impact on model output magnitude) with the TreeExplainer interpreter on the training dataset. Rank features by mean |SHAP| and export the top-ranked features (e.g., top 30) to a CSV such as table_s4_shap_feature_importance.csv; top features should correspond to the paper's favorable epigenetic features (distance to TSS/nucleosome positioning, RNA expression, chromatin accessibility).

## Related tools

- **XGBoost** (Gradient-boosted tree algorithm fitted on the epigenetic feature matrix; its SHAP values are used for feature identification) — https://github.com/yangqianq/EpiCas-DL
- **TreeExplainer** (Interpreter computing mean |SHAP| values on the training dataset to rank features by average impact on model output magnitude)
- **GPyOpt** (Bayesian optimization package used instead of random/grid search to tune model hyperparameters over ~300 acquisitions with 5-fold cross-validation)
- **Python 3.6** (Custom-script environment (with pybedtools) used to extract and assemble the sgRNA epigenetic feature matrix) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Output CSV exists with a feature-name column and a mean |SHAP| value column containing exactly 30 ranked rows
- Top-ranked features correspond to the epigenetic features reported as top favorable features in the paper: nucleosome positioning (distance to TSS), RNA expression, and chromatin accessibility (ATAC-seq)
- Hyperparameter tuning used GPyOpt Bayesian optimization with ~300 acquisitions under 5-fold cross-validation rather than random grid search
- Held-out 10% test performance reported as AUC or Spearman correlation reaches levels competitive with Gradient Boosting, Random Forest, DeepHF, and C-RNNcrispr baselines

## Limitations

- Only four epigenetic feature types (nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression) were considered; incomplete epigenetic feature information limited model performance
- Available CRISPRoff screening datasets are relatively small and mainly from HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets, and screening datasets may contain noise in measuring sgRNA knock-down/knock-up efficacy

## Evidence

- [methods] Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations): "Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations)"
- [methods] Mean |SHAP| values interpreted by TreeExplainer on the training dataset: "Mean |SHAP| values, the average impact on model output magnitude, were interpreted on the training dataset by TreeExplainer"
- [methods] Bayesian optimization with GPyOpt: "we applied a Bayesian optimization package, GPyOpt, which has been demonstrated to be more reliable than random grid search"
- [readme] README model choices and prediction invocation: "There have CRISPRoff_seq/CRISPRoff_epi/CRISPRa_seq/CRISPRa_epi four models for you."
