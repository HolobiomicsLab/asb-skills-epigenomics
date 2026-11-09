---
name: mutation-expression-association-testing
description: Use when validating DriverPower-exclusive driver candidates (e.g. SGK1
  in Lymph-BNHL, GPR126 in Bladder-TCC enhancer) by asking whether mutated samples
  show differential gene expression after correcting for copy number and tissue.
when_to_use_negative:
- Input is already a DriverPower background-model feature table (train_feature.hdf5
  / test_feature.hdf5) rather than per-sample expression — this skill is for downstream
  expression validation, not driver discovery
- The goal is to call driver elements from somatic mutations alone (use the driverpower
  model/infer sub-commands instead)
- Samples lack copy-number state (SCNA) or tissue labels, since the GLM requires both
  covariates for the copy-number-corrected test
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DriverPower
  role: Produces the driver candidates (e.g. SGK1, GPR126) whose mutation–expression
    association is being tested; burden and functional impact tests on PCAWG WGS data
  repo: https://github.com/smshuai/DriverPower
- name: Python
  role: Implements the quasi-Poisson GLMs, likelihood ratio tests, and pandas data
    handling (statsmodels; Python >= 3.5 required by DriverPower)
- name: XGBoost
  role: GBM background-model component of DriverPower used upstream to infer the driver
    candidates being validated by expression testing
  repo: https://github.com/dmlc/xgboost
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:63706b75b0cae1d119e05f73aac1a2db5ae4fde9489933e3725cdf6e2ebfce22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  - XGBoost
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mutation-expression-association-testing

## Summary

Tests whether samples carrying mutations in DriverPower-exclusive candidate elements show copy-number-corrected differential expression, by fitting quasi-Poisson GLMs on PCAWG per-sample FPKM-UQ gene expression with MUT and SCNA covariates and comparing full vs. null models via a likelihood ratio test.

## When to use

Use when validating DriverPower-exclusive driver candidates (e.g. SGK1 in Lymph-BNHL, GPR126 in Bladder-TCC enhancer) by asking whether mutated samples show differential gene expression after correcting for copy number and tissue.

## When NOT to use

- Input is already a DriverPower background-model feature table (train_feature.hdf5 / test_feature.hdf5) rather than per-sample expression — this skill is for downstream expression validation, not driver discovery
- The goal is to call driver elements from somatic mutations alone (use the driverpower model/infer sub-commands instead)
- Samples lack copy-number state (SCNA) or tissue labels, since the GLM requires both covariates for the copy-number-corrected test

## Inputs

- PCAWG per-sample gene-expression quantification (FPKM-UQ)
- Somatic mutation status per sample/element (MUT, e.g. from PCAWG WGS, ~2583 genomes across 38 tumour types)
- Copy-number state per sample/gene (SCNA)
- Tissue/tumour-type labels per sample
- DriverPower-exclusive candidate elements and target genes (SGK1, GPR126)

## Outputs

- Copy-number-corrected p value per gene from the likelihood ratio test (full vs. null GLM)
- expression_glm_pvalues.tsv (full per-gene p-value table)

## How to apply

Load the PCAWG per-sample gene-expression quantification (FPKM-UQ), somatic mutation status (MUT), copy-number state (SCNA), and tissue labels into pandas DataFrames keyed by sample and gene (~20K genes). For each target gene, fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue with statsmodels GLM using the QuasiPoisson family, then fit the null model FPKM-UQ ~ SCNA + tissue on the same samples. Perform a likelihood ratio test between full and null models to obtain a copy-number-corrected p value per gene. Restrict the mutation covariate to samples in the relevant tumour type (e.g. Lymph-BNHL for SGK1, Bladder-TCC enhancer for GPR126), and write the full per-gene table to expression_glm_pvalues.tsv. Rationale: this controls the expression signal for copy-number state so the association reflects driver-mutation status rather than CNV-driven expression, following the DriverPower validation against Fig. 2f, g.

## Related tools

- **DriverPower** (Produces the driver candidates (e.g. SGK1, GPR126) whose mutation–expression association is being tested; burden and functional impact tests on PCAWG WGS data) — https://github.com/smshuai/DriverPower
- **Python** (Implements the quasi-Poisson GLMs, likelihood ratio tests, and pandas data handling (statsmodels; Python >= 3.5 required by DriverPower))
- **XGBoost** (GBM background-model component of DriverPower used upstream to infer the driver candidates being validated by expression testing) — https://github.com/dmlc/xgboost

## Examples

```
import statsmodels.api as sm; res_full = sm.GLM(fpkm_uq, X_full, family=sm.families.QuasiPoisson()).fit(); res_null = sm.GLM(fpkm_uq, X_null, family=sm.families.QuasiPoisson()).fit()
```

## Evaluation signals

- Reproduced SGK1 p value ≈ 3e-13 (copy-number-corrected LRT, Lymph-BNHL mutated vs. wild-type) matches the reported Fig. 2f value within numerical tolerance
- Reproduced GPR126 p value ≈ 0.012 (Bladder-TCC enhancer) matches the reported Fig. 2g value within numerical tolerance
- Null and full GLM models are fit on the same sample set with identical SCNA and tissue covariates, differing only by the MUT term
- expression_glm_pvalues.tsv is produced and contains p values for all ~20K genes, not just the two validation targets
- Direction of effect is consistent with the reported finding: SGK1 is overexpressed in mutated samples

## Limitations

- Many tumour types have fewer than 30 PCAWG samples, limiting statistical power for the expression association test
- False-positive driver calls may arise from variant-calling artefacts or biological processes (e.g. regional AID activity) not captured by the BMR model, which would confound the expression test
- Systematic undercalling of somatic variants in non-coding regions means some driver mutations (e.g. ~216 estimated TERT promoter mutations missed in PCAWG) will be absent from the MUT covariate

## Evidence

- [results] SGK1 is significantly overexpressed in mutated Lymph-BNHL samples relative to non-mutated samples, with a copy number corrected p = 3e-13 from a likelihood ratio test (Fig. 2f).: [evidence span withheld — anchor status: not_found]
- [methods] fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue using statsmodels GLM with the QuasiPoisson family: [evidence span withheld — anchor status: not_found]
- [intro] we combined the two mutation significance testing methods to develop DriverPower (Fig. 1a), a framework for identification of coding and non-coding cancer drivers: "we combined the two mutation significance testing methods to develop DriverPower (Fig. 1a), a framework for identification of coding and non-coding cancer drivers"
- [readme] DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations.: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
