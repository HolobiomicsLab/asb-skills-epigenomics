---
name: tumour-cohort-burden-stratification
description: Use when GLM and GBM BMR models have been trained on the same train_feature.hdf5/train_y.tsv
  data and you must decide which algorithm to report or apply per tumour cohort, especially
  across cohorts spanning high vs low mutational burden.
when_to_use_negative:
- Only one tumour cohort is available, so there is no mutational-burden stratification
  to compare against
- The goal is driver candidate calling from test data ('driverpower infer') rather
  than choosing between BMR modelling algorithms
- Input is raw somatic mutations that have not yet been converted into DriverPower
  training features and response tables
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
tools:
- name: DriverPower
  role: Provides the 'model' sub-command used to train both the GLM (randomized lasso
    feature selection, --featImpCut) and GBM (--gbmFold) BMR models and report per-model
    training metrics
  repo: https://github.com/smshuai/DriverPower
- name: XGBoost
  role: Engine behind the GBM BMR model; trained as k-fold cross-validated models
    with poisson deviance objective using default parameters from xgb_param_default.py
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- name: conda
  role: Used to install DriverPower from the smshuai channel under Python >= 3.5 before
    model training
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:8dd8dd39cb063b95b1ca079d733c89e3a44f6fdd321ddc3720ab4d6f55021199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# tumour-cohort-burden-stratification

## Summary

Compare DriverPower's two background mutation rate (BMR) modelling algorithms — randomized lasso + binomial GLM and k-fold cross-validated XGBoost GBM — by stratifying per-model training metrics (r2, variance explained, Pearson correlation) by tumour cohort mutational burden.

## When to use

Use when GLM and GBM BMR models have been trained on the same train_feature.hdf5/train_y.tsv data and you must decide which algorithm to report or apply per tumour cohort, especially across cohorts spanning high vs low mutational burden.

## When NOT to use

- Only one tumour cohort is available, so there is no mutational-burden stratification to compare against
- The goal is driver candidate calling from test data ('driverpower infer') rather than choosing between BMR modelling algorithms
- Input is raw somatic mutations that have not yet been converted into DriverPower training features and response tables

## Inputs

- train_feature.hdf5 (merged from train_feature.hdf5.part1/2/3, md5 cb4af6bc7979efa087955f94909dd065)
- train_y.tsv (training response table of per-element mutation counts)
- fitted GLM model file [name].GLM.model.pkl with feature importance table
- fitted GBM fold models [name].GBM.model.fold[1..5]

## Outputs

- model_comparison.tsv — GLM vs GBM metrics (r2, variance explained, Pearson correlation) side by side and stratified by cohort mutational burden

## How to apply

Fit both BMR models on identical inputs: the GLM via 'driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GLM', which runs randomized lasso feature selection keeping features with importance score >= the default --featImpCut of 0.5, and the GBM via '--method GBM --gbmFold 5', which trains 5-fold cross-validated XGBoost models with poisson deviance objective using xgb_param_default.py parameters. Collect the per-model training metrics DriverPower reports (r2, variance explained, Pearson correlation; e.g. the tutorial GBM reached r2=0.63). Tabulate GLM vs GBM metrics side by side and stratify rows by cohort mutational burden, saving the table as model_comparison.tsv. The rationale is that model choice is burden-dependent: the two algorithms perform essentially equivalently in high mutational burden cohorts, whereas GBM consistently outperforms GLM in low mutational burden cohorts such as Myeloid-MPN and CNS-PiloAstro, a pattern reported in Supplementary Figs. 4-6.

## Related tools

- **DriverPower** (Provides the 'model' sub-command used to train both the GLM (randomized lasso feature selection, --featImpCut) and GBM (--gbmFold) BMR models and report per-model training metrics) — https://github.com/smshuai/DriverPower
- **XGBoost** (Engine behind the GBM BMR model; trained as k-fold cross-validated models with poisson deviance objective using default parameters from xgb_param_default.py) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- **conda** (Used to install DriverPower from the smshuai channel under Python >= 3.5 before model training) — https://github.com/smshuai/DriverPower

## Examples

```
driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GBM --gbmFold 5
```

## Evaluation signals

- Merged train_feature.hdf5 matches the published md5 checksum cb4af6bc7979efa087955f94909dd065
- GLM run emits a feature importance table in which only features with importance score >= 0.5 (default --featImpCut) are retained in [name].GLM.model.pkl
- GBM run produces exactly --gbmFold fold models ([name].GBM.model.fold[1..5] for --gbmFold 5)
- Per-model training metrics include r2, variance explained and Pearson correlation, with tutorial-scale values such as GBM r2=0.63
- Reproduced comparison table shows GLM and GBM performing similarly on high-burden cohorts while GBM outperforms GLM on low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro (Supplementary Figs. 4-6)

## Limitations

- The default --gbmFold is 3 and must be an integer >= 2; fold choice affects GBM cross-validated model outputs
- Many tumour types have fewer than 30 samples, limiting statistical power for per-cohort stratified comparisons
- The BMR background model is trained on predominantly neutral regions and no tool currently models negatively selected sites, which limits recall
- DriverPower version matters: v1.0.0 was used in the manuscript, with v1.0.1 and v1.0.2 released afterwards containing bug fixes and minor modifications

## Evidence

- [methods] Train GLM background model with randomized lasso feature selection: "Train a generalized linear model with feature selection by randomized lasso"
- [methods] GBM fold parameter: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] Feature importance cutoff for GLM: "``--featImpCut``: [*optional*] cutoff for feature importance score. Features with score >= cutoff will be used in the model. Only used for GLM."
- [abstract] BMR model explanatory power motivating the comparison: "Using a total of 1373 genomic features derived from public sources, DriverPower's background mutation model explains up to 93% of the regional variance in the mutation rate across multiple tumour"
- [supplementary] Validation pattern for burden-stratified GLM vs GBM comparison: [evidence span withheld — anchor status: not_found]
- [readme] Tool scope: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
