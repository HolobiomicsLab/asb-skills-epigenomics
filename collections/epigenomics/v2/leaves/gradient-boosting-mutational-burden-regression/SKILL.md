---
name: gradient-boosting-mutational-burden-regression
description: Use when somatic mutation counts (nMut, nSample, length) per genomic
  element must be modelled as a function of genomic features to estimate expected
  background mutation burden before driver significance testing.
when_to_use_negative:
- Input lacks the training features/responses and the goal is only to call drivers
  from a pre-trained model — use the 'driverpower infer' sub-command directly instead.
- The hypothesis requires an interpretable linear model with feature selection; DriverPower's
  GLM option with randomized lasso is the alternative to GBM.
- Data are not pan-cancer/cancer WGS somatic mutations on hg19 elements — the BMR
  model is trained on predominantly neutral regions of this specific study design.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Provides the 'driverpower model' sub-command that trains GBM (XGBoost) BMR
    models and the 'infer' sub-command that applies them to test elements
  repo: https://github.com/smshuai/DriverPower
- name: dmlc/xgboost
  role: Underlying gradient boosting library used by DriverPower's GBM method for
    k-fold regression of mutation burden
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- name: bedtools
  role: Converts bed12 test element files to bed6 (bed12tobed6) before response-table
    generation
- name: prepare.py
  role: Builds per-element response tables (nMut, nSample, length) for training and
    test elements
  repo: https://raw.githubusercontent.com/smshuai/DriverPower/master/script/prepare.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:8d757b0780e1712a2263980a58fe4e6f0ddec9a6ca6d2f125e3ed78133ef1b5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python >=
  - bedtools
  - dmlc/xgboost
  - prepare.py
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gradient-boosting-mutational-burden-regression

## Summary

Train a gradient boosting machine (XGBoost) background mutation rate (BMR) model in DriverPower that predicts per-element somatic mutation burden from 1373 genomic features, explaining up to 93% of regional mutation-rate variance for the pan-cancer cohort and 83% on held-out 1-Mbp test elements.

## When to use

Use when somatic mutation counts (nMut, nSample, length) per genomic element must be modelled as a function of genomic features to estimate expected background mutation burden before driver significance testing.

## When NOT to use

- Input lacks the training features/responses and the goal is only to call drivers from a pre-trained model — use the 'driverpower infer' sub-command directly instead.
- The hypothesis requires an interpretable linear model with feature selection; DriverPower's GLM option with randomized lasso is the alternative to GBM.
- Data are not pan-cancer/cancer WGS somatic mutations on hg19 elements — the BMR model is trained on predominantly neutral regions of this specific study design.

## Inputs

- train_feature.hdf5 (merged from three split parts; 1373 genomic features)
- train_y.tsv per-element response table (nMut, nSample, length) for 867,266 training elements
- test_feature.hdf5 for 2521 non-overlapping 1-Mbp autosomal test elements
- test_y.tsv per-element response table
- somatic mutation file (e.g. random_mutations.tsv.gz) and callable regions BED

## Outputs

- ./output/tutorial.GBM.model.pkl trained GBM model
- feature_importance.tsv
- model_info.pkl
- result.tsv with predicted expected mutation burden on test elements
- training log metrics (e.g. r2=0.63, Variance explained=0.63)

## How to apply

Generate per-element response tables (nMut, nSample, length) with prepare.py from a somatic mutation file, element BEDs, and callable regions, then concatenate the three split train_feature.hdf5 parts (md5 cb4af6bc7979efa087955f94909dd065). Train with 'driverpower model' using --method GBM and --gbmFold 3, which trains 3-fold XGBoost gradient boosting machines where two folds train and one validates each fold model; this contrasts with the GLM option that uses randomized lasso feature selection. Inspect the training log for reported metrics (the tutorial GBM run reports r2=0.63, Variance explained=0.63 on the training set) and keep the default outputs feature_importance.tsv and model_info.pkl. Apply the trained .GBM.model.pkl via 'driverpower infer' to the test feature table (2521 non-overlapping 1-Mbp autosomal elements) to obtain predicted expected mutation burden in result.tsv, benchmarking against the paper's 93% training / 83% held-out variance explained.

## Related tools

- **DriverPower** (Provides the 'driverpower model' sub-command that trains GBM (XGBoost) BMR models and the 'infer' sub-command that applies them to test elements) — https://github.com/smshuai/DriverPower
- **dmlc/xgboost** (Underlying gradient boosting library used by DriverPower's GBM method for k-fold regression of mutation burden) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- **bedtools** (Converts bed12 test element files to bed6 (bed12tobed6) before response-table generation)
- **prepare.py** (Builds per-element response tables (nMut, nSample, length) for training and test elements) — https://raw.githubusercontent.com/smshuai/DriverPower/master/script/prepare.py

## Examples

```
driverpower model --feature train_feature.hdf5 --response train_y.tsv --method GBM --gbmFold 3 --name tutorial --modelDir ./output
```

## Evaluation signals

- Training log reports 'Model metrics for training set: r2=0.63, Variance explained=0.63' for the tutorial GBM run (paper reports up to 93% variance explained for the pan-cancer cohort).
- Merged train_feature.hdf5 md5sum matches cb4af6bc7979efa087955f94909dd065.
- Infer on test elements yields result.tsv covering the 2521 non-overlapping 1-Mbp autosomal test elements with predicted expected mutation burden.
- Held-out variance explained is consistent with the paper's 83% on the 1-Mbp test element set.
- Default output files feature_importance.tsv, model_info.pkl, and the .GBM.model.pkl all exist in --modelDir.

## Limitations

- The BMR is trained on predominantly neutral regions and no driver discovery tool currently models the effect of negatively selected sites, limiting recall.
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity); additional BMR features are proposed but not yet incorporated.
- Blacklist regions should be removed to reduce variant artefact effects, as done in the manuscript.
- GBM fold count must be an integer >= 2 (default --gbmFold 3); the tutorial run's r2=0.63 on the training set is lower than the paper's reported model fit, so replicate-level variance depends on data and settings.

## Evidence

- [abstract] DriverPower's background mutation model, built from 1373 genomic features, explains up to 93% of the regional variance in mutation rate across multiple tumour types: "Using a total of 1373 genomic features derived from public sources, DriverPower's background mutation model explains up to 93% of the regional variance in the mutation rate"
- [methods] GBM training uses k-fold cross-validation with XGBoost: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] Train a GLM with randomized lasso feature selection as the alternative method: "Train a generalized linear model with feature selection by randomized lasso"
- [methods] Model sub-command trains the BMR from training features and responses: "DriverPower sub-command ``model`` is used to train BMR models. To build the BMR model, training features (X) and responses (y) are required."
- [methods] DriverPower requires Python >= 3.5: "DriverPower requires Python >= 3.5 and some other computing packages."
- [readme] DriverPower is a tool for cancer driver discovery from WGS/WES somatic mutations (README): "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
