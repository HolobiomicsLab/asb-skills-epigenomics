---
name: randomized-lasso-feature-selection
description: Use when training a GLM BMR model in DriverPower from high-dimensional
  genomic feature tables (e.g. train_feature.hdf5 with 1373 features) and a mutation-count
  response table, to discard uninformative features before fitting.
when_to_use_negative:
- The goal is to fit the GBM background model — GBM training uses 5-fold cross-validated
  XGBoost models with poisson deviance objective and does not use randomized lasso
  feature selection
- You are at the 'driverpower infer' stage with an already-trained model; feature
  selection applies only to the 'model' sub-command
- The input is raw somatic mutations rather than a processed training feature table;
  run prepare.py and build the feature/response tables first
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Provides the 'driverpower model' sub-command that performs randomized lasso
    feature selection for GLM BMR training, with the --featImpCut parameter
  repo: https://github.com/smshuai/DriverPower
- name: XGBoost
  role: Underlies the GBM alternative BMR model against which the GLM/randomized-lasso
    model is compared, using poisson deviance objective and default parameters from
    xgb_param_default.py
  repo: https://github.com/dmlc/xgboost
- name: conda
  role: Environment manager for installing DriverPower (conda install -c smshuai driverpower)
    under Python >= 3.5
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:4b83b2a06c4ceacd0db92c0d67c35f9ce3208d4dc48f7b8fd32b866e228e23e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - conda
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# randomized-lasso-feature-selection

## Summary

Feature-selection step within DriverPower's GLM background mutation rate (BMR) modelling, where randomized lasso ranks genomic features and only those with importance score >= the --featImpCut cutoff (default 0.5) are retained in the generalized linear model.

## When to use

Use when training a GLM BMR model in DriverPower from high-dimensional genomic feature tables (e.g. train_feature.hdf5 with 1373 features) and a mutation-count response table, to discard uninformative features before fitting.

## When NOT to use

- The goal is to fit the GBM background model — GBM training uses 5-fold cross-validated XGBoost models with poisson deviance objective and does not use randomized lasso feature selection
- You are at the 'driverpower infer' stage with an already-trained model; feature selection applies only to the 'model' sub-command
- The input is raw somatic mutations rather than a processed training feature table; run prepare.py and build the feature/response tables first

## Inputs

- train_feature.hdf5 (merged from 3 split parts, md5 cb4af6bc7979efa087955f94909dd065)
- train_y.tsv response table of mutation counts per element (built with prepare.py from random_mutations.tsv.gz and train_elements.tsv.gz)
- DriverPower environment (Python >= 3.5, conda install -c smshuai driverpower)

## Outputs

- Feature importance table from randomized lasso
- [name].GLM.model.pkl fitted GLM background model using retained features
- Training metrics (r2, variance explained, Pearson correlation) for model comparison

## How to apply

Fit the GLM background model with 'driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GLM'; this runs randomized lasso for feature selection and writes a feature importance table plus [name].GLM.model.pkl. Retain features whose importance score meets the --featImpCut cutoff (default 0.5); lower the cutoff only with justification since it directly controls model sparsity. Use Python >= 3.5 with DriverPower installed from the smshuai conda channel, and verify the merged training feature file (cat of the three .part hdf5 files) has md5 cb4af6bc7979efa087955f94909dd065 before modelling. After fitting, collect the reported r2, variance explained and Pearson correlation to compare against the GBM alternative, keeping in mind the published pattern that GLM and GBM perform similarly on high-burden cohorts while GBM outperforms GLM in low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro.

## Related tools

- **DriverPower** (Provides the 'driverpower model' sub-command that performs randomized lasso feature selection for GLM BMR training, with the --featImpCut parameter) — https://github.com/smshuai/DriverPower
- **XGBoost** (Underlies the GBM alternative BMR model against which the GLM/randomized-lasso model is compared, using poisson deviance objective and default parameters from xgb_param_default.py) — https://github.com/dmlc/xgboost
- **conda** (Environment manager for installing DriverPower (conda install -c smshuai driverpower) under Python >= 3.5)

## Examples

```
driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GLM
```

## Evaluation signals

- The merged train_feature.hdf5 checksum matches md5 cb4af6bc7979efa087955f94909dd065 before running the model command
- A feature importance table and [name].GLM.model.pkl are produced by the 'driverpower model --method GLM' run
- Retained features all have randomized-lasso importance scores >= the --featImpCut cutoff (default 0.5)
- Reported training metrics (r2, variance explained, Pearson correlation) are collected and the tutorial-scale GBM r2=0.63 provides an order-of-magnitude reference
- Side-by-side GLM vs GBM metrics reproduce the published pattern: similar performance on high-burden cohorts, GBM better on low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro, as in Supplementary Figs. 4-6

## Limitations

- DriverPower's BMR is trained on predominantly neutral regions and does not model negatively selected sites, limiting recall
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity)
- Many tumour types have fewer than 30 samples, limiting statistical power of the background model
- GLM underperforms GBM in low mutational burden tumour cohorts, so the GLM/randomized-lasso route is not uniformly optimal

## Evidence

- [methods] randomized lasso feature selection (--featImpCut cutoff on feature importance): "``--featImpCut``: [*optional*] cutoff for feature importance score. Features with score >= cutoff will be used in the model. Only used for GLM."
- [methods] GLM training with randomized lasso: "Train a generalized linear model with feature selection by randomized lasso"
- [methods] GBM alternative with k-fold cross-validation: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] DriverPower installation requirements: "DriverPower requires Python >= 3.5 and some other computing packages."
- [readme] DriverPower scope from README: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
