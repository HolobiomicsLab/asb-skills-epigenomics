---
name: shap-feature-importance-interpretation
description: Use when a fitted gradient-boosted tree model needs post-hoc interpretation
  into ranked feature contributions, e.g. identifying which epigenetic features (nucleosome
  positioning, chromatin accessibility, RNA expression) favor sgRNA silencing efficiency.
when_to_use_negative:
- The model is a deep neural network rather than a tree ensemble — the paper uses
  DeepSHAP for the deep learning framework instead of Tree SHAP
- The input is raw sgRNA screening data without a fitted model; SHAP interpretation
  requires a trained model first
- Goal is predictive performance benchmarking (e.g. AUC/Spearman comparison against
  DeepHF or C-RNNcrispr) rather than feature attribution
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_3474
tools:
- name: XGBoost
  role: Gradient-boosted tree model whose structure supports exact Tree SHAP computation
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TreeExplainer
  role: SHAP interpreter that computes mean |SHAP| values (average impact on model
    output magnitude) on the training dataset
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Custom scripting environment (with pybedtools 0.9.0) used to assemble the
    feature matrix and run the SHAP analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:462be3ed6dc4dedd720d393c218b91fdab8aff72271e8b85d0ffb6be63bb4331
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - TreeExplainer
  - XGBoost
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# shap-feature-importance-interpretation

## Summary

Interpret a trained tree-based model (e.g. XGBoost on CRISPRoff sgRNA epigenetic features) by computing Tree SHAP values and ranking features by mean |SHAP| to reveal which epigenetic and sequence features drive sgRNA activity predictions.

## When to use

Use when a fitted gradient-boosted tree model needs post-hoc interpretation into ranked feature contributions, e.g. identifying which epigenetic features (nucleosome positioning, chromatin accessibility, RNA expression) favor sgRNA silencing efficiency.

## When NOT to use

- The model is a deep neural network rather than a tree ensemble — the paper uses DeepSHAP for the deep learning framework instead of Tree SHAP
- The input is raw sgRNA screening data without a fitted model; SHAP interpretation requires a trained model first
- Goal is predictive performance benchmarking (e.g. AUC/Spearman comparison against DeepHF or C-RNNcrispr) rather than feature attribution

## Inputs

- Trained XGBoost gradient-boosted tree model (e.g. trained on the CRISPRoff dataset of 111,682 tiling or 20,221 genome-wide sgRNAs)
- sgRNA feature matrix: one-hot encoded 20 nt sequences plus min–max normalized epigenetic features (nucleosome positioning/distance to TSS, ATAC-seq chromatin accessibility, DNA methylation, RNA expression)
- Training dataset portion used to compute mean |SHAP| values

## Outputs

- Ranked feature-importance table (e.g. table_s4_shap_feature_importance.csv) with a feature-name column and a mean |SHAP| value column for the top 30 features
- Interpretation of which epigenetic/sequence features most favorably contribute to predicted sgRNA activity

## How to apply

Fit an XGBoost gradient-boosted tree model on the min–max normalized sgRNA feature matrix (one-hot encoded sequence plus epigenetic features such as nucleosome positioning/distance to TSS, chromatin accessibility from ATAC-seq, DNA methylation, and RNA expression). Instantiate a TreeExplainer interpreter on the trained model and compute mean |SHAP| values — the average impact on model output magnitude — on the training dataset. Rank all features by mean |SHAP| value and export the top-ranked features (e.g. top 30) with their SHAP scores to a CSV table. Validate that top-ranked features correspond to biologically interpretable epigenetic features; in the EpiCas-DL paper, nucleosome positioning, RNA expression levels, and chromatin accessibility emerged among the top five favorable features for CRISPRoff editing.

## Related tools

- **XGBoost** (Gradient-boosted tree model whose structure supports exact Tree SHAP computation) — https://github.com/yangqianq/EpiCas-DL
- **TreeExplainer** (SHAP interpreter that computes mean |SHAP| values (average impact on model output magnitude) on the training dataset) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Custom scripting environment (with pybedtools 0.9.0) used to assemble the feature matrix and run the SHAP analysis)

## Examples

```
from shap import TreeExplainer; shap_values = TreeExplainer(xgb_model).shap_values(X_train); mean_abs_shap = abs(shap_values).mean(axis=0)
```

## Evaluation signals

- Output CSV exists and contains a feature-name column and a mean |SHAP| value column with 30 ranked rows
- Mean |SHAP| values are non-negative (absolute-value aggregation) and monotonically non-increasing across the ranked rows
- Top-ranked features correspond to the epigenetic features reported as top favorable features in the paper: nucleosome positioning (distance to TSS), RNA expression, and chromatin accessibility
- SHAP values are computed on the training dataset with the TreeExplainer on the fitted XGBoost model, not on an untrained or different model

## Limitations

- Model interpretability is limited by incomplete epigenetic feature information — only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression were considered
- SHAP attribution reflects the training data distribution; available CRISPRoff screening datasets were relatively small and mainly from HEK293T and K562 cell lines, raising overfitting risk that may skew feature importance
- Differences between CRISPRoff and CRISPRi systems may affect which features appear important when transferring the interpretation across systems

## Evidence

- [results] Tree SHAP ranked epigenetic features among the top favorable features: [evidence span withheld — anchor status: not_found]
- [methods] Tree SHAP method description: "Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations)"
- [methods] TreeExplainer mean absolute SHAP computation: "Mean |SHAP| values, the average impact on model output magnitude, were interpreted on the training dataset by TreeExplainer"
- [methods] XGBoost association with SHAP values: "which associated the SHAP values with the XGBoost algorithm"
- [methods] Min-max normalization of epigenetic features before model input: "All the epigenetic features were min–max normalized before model input"
