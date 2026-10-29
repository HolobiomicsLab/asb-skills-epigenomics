---
name: glm-vs-gbm-predictive-metrics-comparison
description: Use when fitting DriverPower BMR background models and needing to decide
  between the GLM and GBM methods, or when reproducing the paper's finding that GBM
  outperforms GLM in low mutational burden tumour cohorts.
when_to_use_negative:
- The goal is calling driver candidates from test elements — use the 'driverpower
  infer' sub-command instead of comparing model-fit metrics
- Only one BMR algorithm will be used in the downstream analysis, so the head-to-head
  comparison adds no decision value
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Provides the 'model' sub-command used to fit both the GLM (randomized lasso
    + binomial/GLM) and GBM (XGBoost poisson deviance) background mutation rate models
    and report their training metrics
  repo: https://github.com/smshuai/DriverPower
- name: XGBoost
  role: Underlying gradient boosting library for the GBM BMR models (parallel tree
    boosting, 5-fold cross-validation, poisson deviance objective)
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- name: conda
  role: Installs DriverPower from the smshuai channel in a Python >= 3.5 environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:92314d1fede321a25a79cc8fa9380667dde236bbefc90c6332e7d5151c5e4d89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - conda
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# glm-vs-gbm-predictive-metrics-comparison

## Summary

Side-by-side comparison of DriverPower's two background mutation rate (BMR) modelling algorithms — generalized linear model with randomized lasso feature selection and 5-fold cross-validated XGBoost gradient boosting (GBM) — by tabulating their training metrics (r2, variance explained, Pearson correlation) stratified by cohort mutational burden.

## When to use

Use when fitting DriverPower BMR background models and needing to decide between the GLM and GBM methods, or when reproducing the paper's finding that GBM outperforms GLM in low mutational burden tumour cohorts.

## When NOT to use

- The goal is calling driver candidates from test elements — use the 'driverpower infer' sub-command instead of comparing model-fit metrics
- Only one BMR algorithm will be used in the downstream analysis, so the head-to-head comparison adds no decision value

## Inputs

- train_feature.hdf5 (merged from 3 split parts; 1373 genomic features)
- train_y.tsv response table (per-element mutation counts from prepare.py)

## Outputs

- [name].GLM.model.pkl and GLM feature importance table
- [name].GBM.model.fold[1..5] XGBoost fold models
- model_comparison.tsv with GLM vs GBM r2 / variance explained / Pearson correlation stratified by cohort mutational burden

## How to apply

Merge the 3-part train_feature.hdf5 archive (verifying md5 cb4af6bc7979efa087955f94909dd065), then fit the GLM with 'driverpower model --method GLM', which runs randomized lasso feature selection retaining features with importance >= the default --featImpCut of 0.5 and outputs [name].GLM.model.pkl. Fit the GBM with 'driverpower model --method GBM --gbmFold 5', which trains 5-fold cross-validated XGBoost models with a poisson deviance objective using default parameters from xgb_param_default.py and outputs [name].GBM.model.fold[1..5]. Collect the per-model training metrics DriverPower reports (r2, variance explained, Pearson correlation; the tutorial GBM reached r2=0.63), tabulate GLM vs GBM side by side, and stratify by cohort mutational burden. The expected reproduced pattern is essentially no difference in high-burden cohorts but consistent GBM superiority in low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro, as reported in Supplementary Figs. 4-6.

## Related tools

- **DriverPower** (Provides the 'model' sub-command used to fit both the GLM (randomized lasso + binomial/GLM) and GBM (XGBoost poisson deviance) background mutation rate models and report their training metrics) — https://github.com/smshuai/DriverPower
- **XGBoost** (Underlying gradient boosting library for the GBM BMR models (parallel tree boosting, 5-fold cross-validation, poisson deviance objective)) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- **conda** (Installs DriverPower from the smshuai channel in a Python >= 3.5 environment)

## Examples

```
driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GBM --gbmFold 5
```

## Evaluation signals

- Merged train_feature.hdf5 md5 checksum equals cb4af6bc7979efa087955f94909dd065
- GLM run retains only features with randomized-lasso importance score >= 0.5 (default --featImpCut) and produces [name].GLM.model.pkl
- GBM run produces exactly 5 fold model files ([name].GBM.model.fold[1..5]) matching --gbmFold 5
- Training metrics (r2, variance explained, Pearson correlation) reported for both algorithms; tutorial GBM benchmark is r2=0.63
- Reproduced pattern: GLM and GBM similar on high mutational burden cohorts, GBM consistently better on low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro (Supplementary Figs. 4-6)

## Limitations

- Fold must be an integer >= 2 for --gbmFold (default is 3), so fold counts below 2 are invalid
- The comparison is based on training metrics and cohort stratification; DriverPower's BMR is trained on predominantly neutral regions and does not model negatively selected sites, which limits recall
- Many tumour types have fewer than 30 samples, limiting statistical power of the per-cohort comparison

## Evidence

- [methods] Train a generalized linear model with feature selection by randomized lasso: "Train a generalized linear model with feature selection by randomized lasso"
- [methods] gbmFold parameter: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] featImpCut parameter: "``--featImpCut``: [*optional*] cutoff for feature importance score. Features with score >= cutoff will be used in the model. Only used for GLM."
- [readme] README citation: [evidence span withheld — anchor status: external_doc]
- [readme] XGBoost description: [evidence span withheld — anchor status: external_doc]
