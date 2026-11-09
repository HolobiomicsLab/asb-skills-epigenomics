---
name: gbm-model-prediction-genomic-elements
description: Use when you have a pre-trained DriverPower GBM BMR model and need to
  predict background mutation rates for new test elements (e.g., TCGA WES data sharing
  no samples with the PCAWG training set).
when_to_use_negative:
- You have no pre-trained model and must first fit a BMR model to your own training
  features (use the 'driverpower model' subcommand instead).
- Input data already consists of predicted mutation rates or driver result tables
  rather than raw somatic mutation callsets and feature tables.
- The tumour cohort has fewer than ~30 samples, limiting statistical power for driver
  detection.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: DriverPower
  role: Provides the 'infer' subcommand that applies pre-trained GBM/boosted-tree
    BMR models to test data and calls driver elements using burden and functional-impact
    tests
  repo: github.com/smshuai/DriverPower
- name: XGBoost
  role: Implements the gradient boosting (GBM) background mutation rate model with
    poisson objective and k-fold cross-validation that is loaded and applied for prediction
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- name: bedtools
  role: Builds per-element response tables and aggregates per-mutation CADD scores
    into element-level mean scores (bedtools intersect, bedtools groupby, bedtools
    bed12tobed6)
- name: prepare.py
  role: Helper script that constructs test response tables (nMut, nSample, effective
    length) from somatic mutation VCFs and element files
  repo: https://raw.githubusercontent.com/smshuai/DriverPower/master/script/prepare.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:d475b68369e07cfbfdddf6c5daea64bebc5c77bf842b329a3b31d8dc8b05d724
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DriverPower
  - XGBoost
  - bedtools
  - prepare.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# GBM model prediction on genomic elements

## Summary

Apply a pre-trained gradient boosting machine (GBM) background mutation rate model (XGBoost, poisson objective) from DriverPower to predict expected mutation burden on new genomic elements, enabling transfer of PCAWG whole-genome-trained models to independently processed WES cohorts such as TCGA-LIHC and TCGA-PAAD.

## When to use

Use when you have a pre-trained DriverPower GBM BMR model and need to predict background mutation rates for new test elements (e.g., TCGA WES data sharing no samples with the PCAWG training set).

## When NOT to use

- You have no pre-trained model and must first fit a BMR model to your own training features (use the 'driverpower model' subcommand instead).
- Input data already consists of predicted mutation rates or driver result tables rather than raw somatic mutation callsets and feature tables.
- The tumour cohort has fewer than ~30 samples, limiting statistical power for driver detection.

## Inputs

- Pre-trained GBM BMR model (XGBoost poisson, trained on PCAWG Liver-HCC or Panc-AdenoCA features)
- Test feature table (hdf5 format) of genomic features for the new cohort
- Test response table (nMut, nSample, effective length per coding element) built from GDC MuTect2 WES callsets via prepare.py
- Per-element mean CADD PHRED score table (--funcScore)
- Test elements BED/TSV files (e.g., test_elements.bed12.gz converted with bedtools bed12tobed6)

## Outputs

- Driver candidate results table '[outDir]/[name].result.tsv' with q-values per element
- Predicted background mutation rates for each test element
- Coding driver gene lists (e.g., 14 coding drivers in 364 TCGA-LIHC samples; 6 in 180 TCGA-PAAD samples)

## How to apply

First build test response tables from the GDC MuTect2 whole-exome callsets using the prepare.py helper with bedtools, recording nMut, nSample and effective length per coding element for each cohort (e.g., TCGA-LIHC 364 samples, TCGA-PAAD 180 samples). Enrich elements with per-element mean CADD PHRED scores, computed by retrieving per-mutation CADD scores via the CADD web interface (up to 100K variants per query) and aggregating with bedtools intersect and bedtools groupby. Load the pre-trained GBM model (XGBoost poisson objective, default 3-fold cross-validation) trained on PCAWG Liver-HCC or Panc-AdenoCA features, then run 'driverpower infer' with the test feature and response tables, passing --funcScore to the CADD table and --funcScoreCut "CADD:0.01", leaving --method at its default auto (dispersion-test-based selection between binomial and negative binomial) and --geoMean at its default True. Collect driver candidates significant at the q-value threshold from '[outDir]/[name].result.tsv'. Correctness is judged by whether called genes match known driver sets: 14 coding drivers for LIHC and 6 for PAAD, essentially all known driver genes.

## Related tools

- **DriverPower** (Provides the 'infer' subcommand that applies pre-trained GBM/boosted-tree BMR models to test data and calls driver elements using burden and functional-impact tests) — github.com/smshuai/DriverPower
- **XGBoost** (Implements the gradient boosting (GBM) background mutation rate model with poisson objective and k-fold cross-validation that is loaded and applied for prediction) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- **bedtools** (Builds per-element response tables and aggregates per-mutation CADD scores into element-level mean scores (bedtools intersect, bedtools groupby, bedtools bed12tobed6))
- **prepare.py** (Helper script that constructs test response tables (nMut, nSample, effective length) from somatic mutation VCFs and element files) — https://raw.githubusercontent.com/smshuai/DriverPower/master/script/prepare.py

## Examples

```
driverpower infer --testFeature test_feature.hdf5 --testResponse test_y.tsv --model trained_GBM_Liver-HCC.hdf5 --funcScore cadd_scores.tsv --funcScoreCut "CADD:0.01"
```

## Evaluation signals

- The LIHC run reproduces 14 coding drivers in 364 TCGA-LIHC samples and the PAAD run reproduces 6 coding drivers in 180 TCGA-PAAD samples.
- All (PAAD) or all but one (LIHC) called driver genes match known driver gene sets reported in the paper.
- The response table contains nMut, nSample and effective length per coding element for each cohort.
- Results are read from '[outDir]/[name].result.tsv' and only candidates significant at the q-value threshold are retained.
- The 'infer' run uses --funcScoreCut "CADD:0.01" with method=auto and geoMean=True as in the reference workflow.

## Limitations

- Many tumour types have fewer than 30 samples, limiting statistical power.
- False-positive driver calls may arise from variant-calling artefacts or biological processes (e.g., regional AID activity) not captured by the BMR model.
- DriverPower's BMR is trained on predominantly neutral regions and does not model negatively selected sites, limiting recall.

## Evidence

- [methods] GBM background mutation rate models (XGBoost poisson objective, default 3-fold): "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] Infer driver candidates with pre-trained models: "The ``infer`` sub-command is used to call drivers from test data with pre-trained models."
- [methods] Functional score cutoff for CADD: "--funcScoreCut "CADD:0.01""
- [methods] Dispersion-test-based auto method selection and geometric mean defaults: "Default is auto, which means decision will be made automatically based on the dispersion test of training data."
- [results] Robustness on WES cohorts with pre-trained models: [evidence span withheld — anchor status: not_found]
- [readme] DriverPower applicability to WGS and WES somatic mutations: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
