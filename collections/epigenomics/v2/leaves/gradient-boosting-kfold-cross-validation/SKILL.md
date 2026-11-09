---
name: gradient-boosting-kfold-cross-validation
description: Use when building a DriverPower background mutation rate model from genomic
  features and wanting a flexible non-linear learner (XGBoost GBM) whose performance
  should be compared against, or substituted for, the GLM default — especially for
  low mutational burden cohorts.
when_to_use_negative:
- The cohort already has high mutational burden and a simple interpretable GLM is
  sufficient, since GBM shows essentially no advantage there.
- The input is not the DriverPower-format feature/response tables (train_feature.hdf5
  + train_y.tsv) — the model sub-command requires both X and y prepared via the DriverPower
  data pipeline.
- 'Only one fold / very small training data is available: --gbmFold must be an integer
  >= 2, so k-fold cross-validation cannot be run with a single split.'
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: DriverPower
  role: Provides the 'model' sub-command that trains the GBM BMR background model
    with --gbmFold k-fold cross-validation and reports per-model training metrics.
  repo: https://github.com/smshuai/DriverPower
- name: XGBoost
  role: Gradient boosting (tree boosting) library underlying DriverPower's GBM method,
    run with poisson deviance objective and defaults from xgb_param_default.py; --gbmFold
    controls the number of folds.
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- name: bedtools
  role: Used upstream to convert BED12 test elements (bedtools bed12tobed6) into the
    element tables that feed model training data preparation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:26981ceb3e522bb22412a68b8425c16f60d4131c9389da3d007d5226386fd6c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - bedtools
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Gradient boosting k-fold cross-validation

## Summary

Training of k-fold cross-validated XGBoost (GBM) background mutation rate (BMR) models in DriverPower, used as a non-linear alternative to the randomized-lasso + binomial GLM for predicting regional mutation burden across tumour cohorts.

## When to use

Use when building a DriverPower background mutation rate model from genomic features and wanting a flexible non-linear learner (XGBoost GBM) whose performance should be compared against, or substituted for, the GLM default — especially for low mutational burden cohorts.

## When NOT to use

- The cohort already has high mutational burden and a simple interpretable GLM is sufficient, since GBM shows essentially no advantage there.
- The input is not the DriverPower-format feature/response tables (train_feature.hdf5 + train_y.tsv) — the model sub-command requires both X and y prepared via the DriverPower data pipeline.
- Only one fold / very small training data is available: --gbmFold must be an integer >= 2, so k-fold cross-validation cannot be run with a single split.

## Inputs

- train_feature.hdf5 (merged from train_feature.hdf5.part1/2/3, md5 cb4af6bc7979efa087955f94909dd065)
- train_y.tsv (per-element mutation counts / response table built with prepare.py from random_mutations.tsv.gz, train_elements.tsv.gz and callable.bed.gz)

## Outputs

- [name].GBM.model.fold[1..5] (per-fold pickled XGBoost models)
- per-model training metrics (r2, variance explained, Pearson correlation)
- model_comparison.tsv (GLM vs GBM metrics stratified by cohort mutational burden)

## How to apply

Merge the split training features (cat train_feature.hdf5.part1 part2 part3 > train_feature.hdf5, md5 cb4af6bc7979efa087955f94909dd065) and run 'driverpower model' with --method GBM and --gbmFold set to the desired fold count (integer >= 2, default 3; the tutorial used 5), which trains per-fold XGBoost models with the poisson deviance objective using default parameters from xgb_param_default.py and outputs [name].GBM.model.fold[1..5]. Collect the per-model training metrics DriverPower reports (r2, variance explained, Pearson correlation — the tutorial GBM reached r2=0.63) and tabulate them side by side against the GLM metrics, where the GLM is fitted after randomized lasso feature selection with --featImpCut 0.5 retaining features with importance >= 0.5. Stratify the GLM vs GBM comparison by cohort mutational burden, because the algorithms' relative performance depends on burden: they are near-equivalent in high-burden cohorts, whereas GBM consistently outperforms GLM in low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro.

## Related tools

- **DriverPower** (Provides the 'model' sub-command that trains the GBM BMR background model with --gbmFold k-fold cross-validation and reports per-model training metrics.) — https://github.com/smshuai/DriverPower
- **XGBoost** (Gradient boosting (tree boosting) library underlying DriverPower's GBM method, run with poisson deviance objective and defaults from xgb_param_default.py; --gbmFold controls the number of folds.) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- **bedtools** (Used upstream to convert BED12 test elements (bedtools bed12tobed6) into the element tables that feed model training data preparation.)

## Examples

```
driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GBM --gbmFold 5
```

## Evaluation signals

- md5 checksum of the merged train_feature.hdf5 equals cb4af6bc7979efa087955f94909dd065.
- Exactly --gbmFold model files are produced, named [name].GBM.model.fold[1..k], with k >= 2.
- Reported GBM training metrics (r2, variance explained, Pearson correlation) are in a plausible range — e.g. the tutorial GBM reached r2=0.63.
- In the GLM-vs-GBM comparison table, GBM matches GLM on high-burden cohorts but consistently outperforms GLM on low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro (Supplementary Figs. 4-6 pattern).

## Limitations

- GBM fold count must be an integer >= 2 (default 3); no single-split training is possible via --gbmFold.
- Comparison conclusions are cohort-dependent: GBM's advantage over the randomized-lasso + binomial GLM is only consistent in low mutational burden tumour cohorts.
- Many tumour types in the PCAWG data set have fewer than 30 samples, limiting statistical power for model comparison.
- DriverPower's BMR is trained on predominantly neutral regions and no tool models negatively selected sites, capping recall regardless of learner choice.

## Evidence

- [methods] ``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3.: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] Train a generalized linear model with feature selection by randomized lasso: "Train a generalized linear model with feature selection by randomized lasso"
- [methods] DriverPower sub-command ``model`` is used to train BMR models. To build the BMR model, training features (X) and responses (y) are required.: "DriverPower sub-command ``model`` is used to train BMR models. To build the BMR model, training features (X) and responses (y) are required."
- [readme] DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations.: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
- [supplementary] GLM and GBM perform similarly on high-burden cohorts while GBM consistently outperforms GLM on low-burden cohorts such as Myeloid-MPN and CNS-PiloAstro: [evidence span withheld — anchor status: not_found]
