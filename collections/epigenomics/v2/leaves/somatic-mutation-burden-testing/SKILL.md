---
name: somatic-mutation-burden-testing
description: Use when calling candidate driver elements (coding or non-coding) from
  tumour somatic mutation callsets (WGS or WES, e.g. PCAWG or TCGA) by testing whether
  per-element mutational burden exceeds a modelled background mutation rate.
when_to_use_negative:
- Input is already a driver result table or a precomputed feature table — this skill
  only covers building BMR models and inferring drivers from raw somatic mutation
  calls plus element/feature tables
- Analysis targets negatively selected sites or requires explicit modelling of negative
  selection — the BMR is trained on predominantly neutral regions and does not model
  this effect
- Tumour type has very few samples (<30), where statistical power for burden testing
  is limited per the paper's discussion
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: DriverPower
  role: 'Primary package: ''driverpower model'' trains the BMR model and ''driverpower
    infer'' calls driver elements with burden plus optional CADD functional adjustment'
  repo: https://github.com/smshuai/DriverPower
- name: Python
  role: Runtime environment (>= 3.5) and language for the prepare.py response-table
    helper
  repo: https://github.com/smshuai/DriverPower
- name: bedtools
  role: Interval manipulation for building response tables (bed12tobed6), intersecting
    mutations with CADD scores, and grouping mean CADD PHRED per element
- name: XGBoost
  role: Gradient boosting backend for the GBM background mutation rate model (poisson
    objective, default 3-fold cross-validation)
  repo: https://github.com/dmlc/xgboost
- name: conda / pip
  role: Installation channels for DriverPower (conda install -c smshuai driverpower
    or pip install driverpower)
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:ac449cbc33219227176024c1e81bba4a862a0ebfe13646d1895f371adebf6b3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  - XGBoost
  - bedtools
  - conda / pip
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# somatic-mutation-burden-testing

## Summary

Combined burden and functional impact testing for cancer driver discovery using DriverPower, which models regional background mutation rates with genomic features (GBM/XGBoost poisson models from 1373 features) and calls coding or non-coding driver elements by testing excess mutational burden adjusted for functional impact scores such as CADD.

## When to use

Use when calling candidate driver elements (coding or non-coding) from tumour somatic mutation callsets (WGS or WES, e.g. PCAWG or TCGA) by testing whether per-element mutational burden exceeds a modelled background mutation rate.

## When NOT to use

- Input is already a driver result table or a precomputed feature table — this skill only covers building BMR models and inferring drivers from raw somatic mutation calls plus element/feature tables
- Analysis targets negatively selected sites or requires explicit modelling of negative selection — the BMR is trained on predominantly neutral regions and does not model this effect
- Tumour type has very few samples (<30), where statistical power for burden testing is limited per the paper's discussion

## Inputs

- Somatic mutation callsets (e.g. PCAWG whole-genome or GDC MuTect2 whole-exome VCFs, hg19)
- Genomic element tables: train_elements.tsv.gz, test_elements.bed12.gz (BED12)
- Training/test feature tables as HDF5 (train_feature.hdf5, test_feature.hdf5; 1373 genomic features)
- Callable regions (callable.bed.gz) and blacklist regions
- Per-mutation functional impact scores (CADD PHRED, retrieved in ≤100K variant batches)
- Pre-trained BMR model (XGBoost poisson GBM, e.g. trained on PCAWG Liver-HCC or Panc-AdenoCA)

## Outputs

- Driver candidate table per cohort: '[outDir]/[name].result.tsv' with elements significant at the q-value threshold
- Trained BMR model (GLM or GBM) from 'driverpower model'
- Response tables (train_y.tsv / test response) with nMut, nSample and effective length per element

## How to apply

Prepare training and test feature/response tables (nMut, nSample, effective length per element) with the prepare.py helper and bedtools, removing blacklist regions to reduce variant artefacts. Retrieve per-mutation CADD scores via the CADD web interface (up to 100K variants per query) and aggregate mean CADD PHRED per element with bedtools intersect/groupby. Train a background mutation rate model with 'driverpower model' — either a GLM with randomized lasso feature selection (--featImpCut) or an XGBoost poisson GBM with 3-fold cross-validation (--gbmFold). Then run 'driverpower infer' on test data with pre-trained models, optionally adding functional adjustment via --funcScore and --funcScoreCut 'CADD:0.01'; keep --method at auto (dispersion-test-based choice of binomial vs negative binomial) and --geoMean True (geometric mean of nMut and nSample). Collect driver candidates significant at the q-value threshold from '[outDir]/[name].result.tsv'.

## Related tools

- **DriverPower** (Primary package: 'driverpower model' trains the BMR model and 'driverpower infer' calls driver elements with burden plus optional CADD functional adjustment) — https://github.com/smshuai/DriverPower
- **Python** (Runtime environment (>= 3.5) and language for the prepare.py response-table helper) — https://github.com/smshuai/DriverPower
- **bedtools** (Interval manipulation for building response tables (bed12tobed6), intersecting mutations with CADD scores, and grouping mean CADD PHRED per element)
- **XGBoost** (Gradient boosting backend for the GBM background mutation rate model (poisson objective, default 3-fold cross-validation)) — https://github.com/dmlc/xgboost
- **conda / pip** (Installation channels for DriverPower (conda install -c smshuai driverpower or pip install driverpower)) — https://github.com/smshuai/DriverPower

## Examples

```
driverpower infer --testElement test_elements.bed --testInput test_feature.hdf5 --testResponse test_y.tsv --model gbm.model --funcScore cadd.tsv --funcScoreCut "CADD:0.01" --outDir out/
```

## Evaluation signals

- Reproducing the PCAWG-scale result: 217 coding and 95 non-coding driver candidates across 2583 cancer genomes
- Transfer validation on WES: 14 coding drivers in 364 TCGA-LIHC samples and 6 coding drivers in 180 TCGA-PAAD samples using Liver-HCC/Panc-AdenoCA-trained models, with called genes matching known driver gene sets
- Response tables contain nMut, nSample and effective length per coding element for each cohort
- Model fit check: the BMR model built from 1373 genomic features should explain up to 93% of the regional variance in mutation rate across multiple tumour types
- Dispersion test of training data selects binomial vs negative binomial when --method=auto; --geoMean defaults to True

## Limitations

- The BMR is trained on predominantly neutral regions and does not model negatively selected sites, limiting recall
- False positives may arise from variant-calling artefacts or biological processes not captured by the BMR (e.g. regional AID activity); additional BMR features are proposed but not incorporated
- Systematic undercalling of somatic variants in non-coding regions (e.g. ~216 TERT promoter mutations estimated missed in PCAWG) reduces sensitivity
- Many tumour types have fewer than 30 samples, limiting statistical power
- Only ~3.7% of the genome was studied; functional impact score schemes are biased toward coding mutations, hampering non-coding driver discovery

## Evidence

- [abstract] DriverPower's background mutation model, built from 1373 genomic features, explains up to 93% of the regional variance in mutation rate across multiple tumour types: "Using a total of 1373 genomic features derived from public sources, DriverPower's background mutation model explains up to 93% of the regional variance in the mutation rate across multiple tumour"
- [methods] GBM training with k-fold cross-validation: "``--gbmFold``: [*optional*] Number of model fold to train for GBM. Fold must be an integer >= 2. Default value is 3."
- [methods] functional adjustment with CADD cutoff: "``--funcScoreCut``: [*optional*] Cutoff of each functional impact scores. The format of this parameter is a string in "NAME1:CUTOFF1;NAME2:CUTOFF2...""
- [methods] blacklist region filtering: "In our manuscript, we also removed all blacklist regions to reduce the effect of variant artifacts."
- [discussion] no modelling of negative selection: "To our knowledge, no driver discovery tool currently models the effect of negatively selected sites; future work aims to take this mechanism into account."
- [readme] DriverPower scope: coding and non-coding drivers from WGS or WES: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
