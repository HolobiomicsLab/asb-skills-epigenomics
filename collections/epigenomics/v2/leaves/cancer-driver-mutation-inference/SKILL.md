---
name: cancer-driver-mutation-inference
description: Use when you have processed somatic mutation response tables (nMut, nSample,
  effective length per element) for coding or non-coding test elements and a pre-trained
  BMR model, and need to call driver candidates at q < 0.1.
when_to_use_negative:
- You need to train the background mutation model from scratch — use the 'driverpower
  model' sub-command workflow instead of 'infer'.
- Your somatic mutations are not in hg19 or are not processed per-element response
  tables (the example data and tutorial are hg19-based).
- You expect modelling of negatively selected sites or burden-only calls without functional
  scores — DriverPower's BMR is trained on predominantly neutral regions and does
  not model negative selection.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DriverPower
  role: Main framework; its 'infer' sub-command calls driver candidates from test
    data with pre-trained BMR models, with --funcScore, --funcScoreCut, --method and
    --geoMean options.
  repo: github.com/smshuai/DriverPower
- name: bedtools
  role: Intersects per-mutation CADD score files with test elements and aggregates
    mean scores per element (bedtools intersect, bedtools groupby); converts BED12
    to BED6 for element preparation.
- name: XGBoost
  role: Provides the GBM background mutation rate model used by DriverPower (poisson
    objective, default 3-fold cross-validation).
  repo: https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- name: prepare.py
  role: Helper script that builds response tables from somatic mutation callsets and
    element files, recording nMut, nSample and effective length per element.
  repo: https://raw.githubusercontent.com/smshuai/DriverPower/master/script/prepare.py
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:d278c76838d129730b9b462fee8d43fb0e233221225b68eeb8fc5992adb8715e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DriverPower
  - Python >=
  - XGBoost
  - bedtools
  - prepare.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cancer-driver-mutation-inference

## Summary

Run DriverPower's 'infer' sub-command to identify coding and non-coding cancer driver elements by combining mutational burden tests (binomial/negative binomial) with functional impact scores (e.g. CADD), using a pre-trained background mutation rate (BMR) model. Applied to PCAWG whole-genome data it yields 217 coding and 95 non-coding driver candidates across 2583 genomes.

## When to use

Use when you have processed somatic mutation response tables (nMut, nSample, effective length per element) for coding or non-coding test elements and a pre-trained BMR model, and need to call driver candidates at q < 0.1.

## When NOT to use

- You need to train the background mutation model from scratch — use the 'driverpower model' sub-command workflow instead of 'infer'.
- Your somatic mutations are not in hg19 or are not processed per-element response tables (the example data and tutorial are hg19-based).
- You expect modelling of negatively selected sites or burden-only calls without functional scores — DriverPower's BMR is trained on predominantly neutral regions and does not model negative selection.

## Inputs

- test feature table (test_feature.hdf5)
- test response table with nMut, nSample and effective length per coding element (e.g. GDC MuTect2 whole-exome callsets via prepare.py)
- pre-trained BMR model information (GBM, XGBoost poisson objective, 3-fold)
- per-element functional impact score table (e.g. CADD_per_ele_score.tsv)
- test element BED files (e.g. test_elements.bed12.gz, hg19)

## Outputs

- [outDir]/[name].result.tsv listing driver candidates with q-values (BH-corrected, threshold q < 0.1)
- aggregated coding and non-coding driver candidate lists (e.g. DriverPower.result.tsv)

## How to apply

Prepare per-element test response tables (nMut, nSample, effective length) with the prepare.py helper and bedtools, and retrieve per-mutation CADD PHRED scores via the CADD web interface, aggregating per-element means with bedtools intersect and bedtools groupby. For each test element set (CDS, splice site, 5'-UTR, 3'-UTR, promoter, enhancer, lncRNA, lncRNA promoter), run 'driverpower infer' with test_feature.hdf5, the test response table, and the model information from the pre-trained BMR model (GBM with XGBoost poisson objective), supplying --funcScore CADD_per_ele_score.tsv and --funcScoreCut 'CADD:0.01'. Keep the default --method auto, which selects between binomial and negative binomial p-values via a dispersion test of the training data, and --geoMean True (geometric mean of nMut and nSample). Apply Benjamini-Hochberg correction to p-values and retain driver candidates with q < 0.1, writing results to [outDir]/[name].result.tsv.

## Related tools

- **DriverPower** (Main framework; its 'infer' sub-command calls driver candidates from test data with pre-trained BMR models, with --funcScore, --funcScoreCut, --method and --geoMean options.) — github.com/smshuai/DriverPower
- **bedtools** (Intersects per-mutation CADD score files with test elements and aggregates mean scores per element (bedtools intersect, bedtools groupby); converts BED12 to BED6 for element preparation.)
- **XGBoost** (Provides the GBM background mutation rate model used by DriverPower (poisson objective, default 3-fold cross-validation).) — https://github.com/dmlc/xgboost/blob/master/doc/parameter.md
- **prepare.py** (Helper script that builds response tables from somatic mutation callsets and element files, recording nMut, nSample and effective length per element.) — https://raw.githubusercontent.com/smshuai/DriverPower/master/script/prepare.py

## Examples

```
python ./prepare.py random_mutations.tsv.gz train_elements.tsv.gz callable.bed.gz train_y.tsv
```

## Evaluation signals

- Reproduce the reported DriverPower result of 217 coding and 95 non-coding driver candidates across the 2583 PCAWG cancer genomes.
- For TCGA WES: the LIHC run reproduces 14 coding drivers in 364 samples and the PAAD run reproduces 6 coding drivers in 180 samples.
- Called genes match the known driver gene sets reported in the paper (all PAAD drivers and all but one LIHC driver were known driver genes).
- DriverPower achieves the highest F1 score for both coding and non-coding driver discovery compared to six published methods used by the PCAWG Drivers and Functional Interpretation Working Group.
- Check that the result file [outDir]/[name].result.tsv exists and contains BH-corrected q-values below the 0.1 threshold for retained candidates.

## Limitations

- DriverPower's BMR is trained on predominantly neutral regions, so the effect of negatively selected sites is not modelled, which limits recall.
- False-positive driver calls may arise from variant-calling artefacts or biological processes not captured by the BMR model (e.g. regional AID activity).
- Systematic undercalling of somatic variants in non-coding regions (e.g. ~216 TERT promoter mutations estimated missed in PCAWG) and many tumour types with fewer than 30 samples limit statistical power.
- Only ~3.7% of the genome was studied, and functional impact score schemes are biased toward coding mutations.

## Evidence

- [abstract] DriverPower identifies 217 coding and 95 non-coding driver candidates across 2583 PCAWG cancer genomes: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
- [methods] infer sub-command and dispersion-based auto method: "The ``infer`` sub-command is used to call drivers from test data with pre-trained models."
- [methods] CADD functional score cutoff: "--funcScoreCut "CADD:0.01""
- [methods] auto method selection: "Default is auto, which means decision will be made automatically based on the dispersion test of training data."
- [methods] geometric mean option: "``--geoMean``: [*optional*] Whether to use geometric mean of nMut and nSample in test. Default is ``True``."
- [readme] WES robustness finding: [evidence span withheld — anchor status: not_found]
- [discussion] BMR limitation: "To our knowledge, no driver discovery tool currently models the effect of negatively selected sites; future work aims to take this mechanism into account."
