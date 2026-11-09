---
name: crispr-off-sgrna-activity-feature-ranking
description: Use when a trained CRISPRoff sgRNA-activity model (or comparable tree-based
  efficiency model) must be interpreted to identify which epigenetic features (nucleosome
  positioning, chromatin accessibility, DNA methylation, RNA expression) most favor
  editing efficiency.
when_to_use_negative:
- The goal is sgRNA activity prediction itself rather than model interpretation —
  use the EpiCas-DL prediction models (CRISPRoff_seq/CRISPRoff_epi) instead.
- Only sequence information is available with no epigenetic annotations (methylation,
  expression, accessibility, TSS distance), since the feature ranking is meant to
  expose epigenetic contributions.
- The dataset is CRISPRa activation screening where feature-favorability directions
  may differ from the CRISPRoff repression findings being reproduced.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: XGBoost
  role: Gradient-boosted tree algorithm trained on the CRISPRoff feature matrix; its
    tree structure enables exact SHAP computation
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TreeExplainer
  role: SHAP interpreter that computes mean |SHAP| values on the training dataset
    for feature importance ranking
  repo: https://github.com/yangqianq/EpiCas-DL
- name: pybedtools
  role: Used in Python 3.6 custom scripts to assemble sgRNA and epigenetic feature
    matrices (e.g., distance to TSS, overlapping annotations)
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:32e18ce33e8a438225792d879e12a95eb4cdb8e7086d910f8d7454fc9eece736
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - TreeExplainer
  - XGBoost
  - pybedtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# crispr-off-sgrna-activity-feature-ranking

## Summary

Ranks epigenetic and sequence features that drive sgRNA activity in CRISPRoff gene silencing by computing mean |SHAP| values with Tree SHAP on an XGBoost model, exposing features such as nucleosome positioning (distance to TSS), RNA expression, and chromatin accessibility as top favorable predictors.

## When to use

Use when a trained CRISPRoff sgRNA-activity model (or comparable tree-based efficiency model) must be interpreted to identify which epigenetic features (nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression) most favor editing efficiency.

## When NOT to use

- The goal is sgRNA activity prediction itself rather than model interpretation — use the EpiCas-DL prediction models (CRISPRoff_seq/CRISPRoff_epi) instead.
- Only sequence information is available with no epigenetic annotations (methylation, expression, accessibility, TSS distance), since the feature ranking is meant to expose epigenetic contributions.
- The dataset is CRISPRa activation screening where feature-favorability directions may differ from the CRISPRoff repression findings being reproduced.

## Inputs

- sgRNA sequence + epigenetic feature matrix (nucleosome positioning/distance to TSS, ATAC-seq chromatin accessibility, DNA methylation, RNA expression) for the CRISPRoff training dataset
- Trained XGBoost gradient-boosted tree model for CRISPRoff sgRNA activity
- Min-max normalized epigenetic annotations (e.g., from ATACdb ATAC-seq data)

## Outputs

- Ranked table of top 30 features with feature names and mean |SHAP| values (table_s4_shap_feature_importance.csv)
- Identification of top favorable epigenetic features (nucleosome positioning, RNA expression, chromatin accessibility) for CRISPRoff editing

## How to apply

Assemble the sgRNA sequence and epigenetic feature matrix (nucleosome positioning/distance to TSS, chromatin accessibility from ATAC-seq, DNA methylation, RNA expression) extracted with the in-house Python 3.6 pybedtools scripts, min-max normalizing epigenetic features before model input. Fit an XGBoost gradient-boosted tree model on the 90% training split of the CRISPRoff dataset, then compute Shapley Additive exPlanations values with the TreeExplainer interpreter, calculating mean |SHAP| values (the average impact on model output magnitude) on the training dataset. Rank all features by mean |SHAP| value and export the top 30 with their SHAP values to a table (e.g., table_s4_shap_feature_importance.csv); interpret top-ranked features as those favoring efficient epigenome editing, cross-checking directional expectations (e.g., methylated target sites and highly expressed genes show higher repression efficiency).

## Related tools

- **XGBoost** (Gradient-boosted tree algorithm trained on the CRISPRoff feature matrix; its tree structure enables exact SHAP computation) — https://github.com/yangqianq/EpiCas-DL
- **TreeExplainer** (SHAP interpreter that computes mean |SHAP| values on the training dataset for feature importance ranking) — https://github.com/yangqianq/EpiCas-DL
- **pybedtools** (Used in Python 3.6 custom scripts to assemble sgRNA and epigenetic feature matrices (e.g., distance to TSS, overlapping annotations)) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
import shap; explainer = shap.TreeExplainer(xgb_model); sv = explainer.shap_values(X_train); mean_abs = pd.Series(np.abs(sv).mean(axis=0), index=feature_names).sort_values(ascending=False).head(30)
```

## Evaluation signals

- Output CSV exists and contains a feature-name column and a mean |SHAP| value column with exactly 30 ranked rows.
- Top-ranked features correspond to the epigenetic features reported in the paper: nucleosome positioning (distance to TSS), RNA expression, and chromatin accessibility (ATAC-seq).
- SHAP values are computed on the training dataset via TreeExplainer, matching the paper's stated protocol (mean |SHAP| = average impact on model output magnitude).
- Directional sanity check: methylated target sites and highly expressed genes should align with higher gene repression efficiency in the CRISPRoff datasets.
- Feature matrix is min-max normalized before model input, as the article specifies.

## Limitations

- Only four epigenetic feature types (nucleosome positioning, chromatin accessibility, DNA methylation, RNA expression) were considered; incomplete epigenetic feature information limited model performance.
- CRISPRoff screening datasets are relatively small and mainly from HEK293T and K562 cell lines, increasing overfitting risk and limiting generalization of the ranked features.
- Differences between CRISPRoff and CRISPRi systems might affect interpretation, and screening datasets may contain noise in measured knock-down efficacy.

## Evidence

- [methods] Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations): "Feature identification was achieved using a recently developed method, Tree SHAP (Shapley Additive exPlanations)"
- [methods] Mean |SHAP| values interpreted on the training dataset by TreeExplainer: "Mean |SHAP| values, the average impact on model output magnitude, were interpreted on the training dataset by TreeExplainer"
- [methods] SHAP values associated with the XGBoost algorithm: "which associated the SHAP values with the XGBoost algorithm [26]"
- [readme] Four epigenetic feature types combined in EpiCas-DL: "combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
- [results] DNA methylation associated with higher repression efficiency in CRISPRoff datasets: "sites with DNA methylation showed higher gene repression efficiency in the two CRISPRoff datasets"
