---
name: gene-level-p-value-reproduction
description: Use when validating DriverPower-exclusive non-coding driver candidates
  by testing whether genes linked to mutated candidate elements show differential
  expression after correcting for copy number (SCNA) and tissue type.
when_to_use_negative:
- Input is already a background mutation model output (train_feature.hdf5 / response
  table) from the DriverPower model/infer pipeline — this skill models expression,
  not mutation burden
- The goal is to call new driver elements with burden tests rather than to validate
  expression effects of already-called candidates
- Only RNA-seq data without matched somatic MUT and SCNA calls are available, so the
  copy-number-corrected model cannot be fit
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: DriverPower
  role: Source of the DriverPower-exclusive driver candidates (SGK1 in Lymph-BNHL,
    GPR126 in Bladder-TCC enhancer) whose expression effects are tested
  repo: https://github.com/smshuai/DriverPower
- name: Python
  role: Language for the pandas data loading, statsmodels quasi-Poisson GLM fitting,
    and likelihood ratio testing workflow (requires Python >= 3.5)
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:a0ba7ffd696d1dbae77c06003aaa1e09f456e2c0250b29acf27c549561a865e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-level-p-value-reproduction

## Summary

Reproduces the copy-number-corrected differential expression p values reported in the DriverPower paper (e.g. SGK1 p = 3e-13 in Lymph-BNHL, GPR126 p = 0.012 in Bladder-TCC enhancer) by fitting quasi-Poisson GLMs to PCAWG per-sample FPKM-UQ expression with mutation (MUT), copy-number (SCNA) and tissue covariates and performing likelihood ratio tests.

## When to use

Use when validating DriverPower-exclusive non-coding driver candidates by testing whether genes linked to mutated candidate elements show differential expression after correcting for copy number (SCNA) and tissue type.

## When NOT to use

- Input is already a background mutation model output (train_feature.hdf5 / response table) from the DriverPower model/infer pipeline — this skill models expression, not mutation burden
- The goal is to call new driver elements with burden tests rather than to validate expression effects of already-called candidates
- Only RNA-seq data without matched somatic MUT and SCNA calls are available, so the copy-number-corrected model cannot be fit

## Inputs

- PCAWG per-sample gene-expression quantification table (FPKM-UQ), keyed by sample and gene (~20K genes)
- Somatic mutation status per sample (MUT) for candidate driver elements
- Copy-number state per sample (SCNA)
- Tumour tissue labels per sample (e.g. Lymph-BNHL, Bladder-TCC)

## Outputs

- expression_glm_pvalues.tsv — full per-gene table of copy-number-corrected LRT p values
- Target gene p values for SGK1 and GPR126 for comparison against Fig. 2f, g

## How to apply

Load PCAWG per-sample gene-expression quantifications (FPKM-UQ), somatic mutation status (MUT), copy-number state (SCNA), and tissue labels into pandas DataFrames keyed by sample and gene (~20K genes). For each target gene, fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue using statsmodels GLM with the QuasiPoisson family, and fit the null model FPKM-UQ ~ SCNA + tissue on the same samples. Perform a likelihood ratio test between full and null models to obtain the copy-number-corrected p value per gene. Extract p values for SGK1 (mutated in Lymph-BNHL samples) and GPR126 (mutated in the Bladder-TCC enhancer element), writing the full per-gene table to expression_glm_pvalues.tsv. The rationale is that differential expression of the linked gene, after SCNA correction, provides functional support for DriverPower-exclusive candidates that burden tests alone cannot confirm.

## Related tools

- **DriverPower** (Source of the DriverPower-exclusive driver candidates (SGK1 in Lymph-BNHL, GPR126 in Bladder-TCC enhancer) whose expression effects are tested) — https://github.com/smshuai/DriverPower
- **Python** (Language for the pandas data loading, statsmodels quasi-Poisson GLM fitting, and likelihood ratio testing workflow (requires Python >= 3.5)) — https://github.com/smshuai/DriverPower

## Examples

```
import statsmodels.api as sm; full = sm.GLM(y, X[['MUT','SCNA','tissue']], family=sm.families.QuasiPoisson()).fit(); null = sm.GLM(y, X[['SCNA','tissue']], family=sm.families.QuasiPoisson()).fit()
```

## Evaluation signals

- Reproduced SGK1 p value matches the reported copy-number-corrected p = 3e-13 within numerical tolerance
- Reproduced GPR126 p value matches the reported p = 0.012 within numerical tolerance
- SGK1 is significantly overexpressed in mutated Lymph-BNHL samples relative to non-mutated samples, consistent with Fig. 2f
- The full GLM includes both MUT and SCNA covariates; the null model omits MUT only, so the LRT has 1 degree of freedom per gene
- Output file expression_glm_pvalues.tsv contains one row per tested gene with finite p values

## Limitations

- Many tumour types have fewer than 30 samples, limiting statistical power for the expression tests in small cohorts
- Systematic undercalling of somatic variants in non-coding regions (e.g. ~216 TERT promoter mutations estimated missed in PCAWG) can misclassify MUT status feeding the model
- Validation depends on expert review against the published Fig. 2f, g values; discrepancies may arise from sample subsetting or normalisation differences

## Evidence

- [other] For each target gene, fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue using statsmodels GLM with the QuasiPoisson family.: [evidence span withheld — anchor status: not_found]
- [other] SGK1 is significantly overexpressed in mutated Lymph-BNHL samples relative to non-mutated samples, with a copy number corrected p = 3e-13 from a likelihood ratio test (Fig. 2f).: [evidence span withheld — anchor status: not_found]
- [methods] DriverPower requires Python >= 3.5 and some other computing packages.: "DriverPower requires Python >= 3.5 and some other computing packages."
- [readme] DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations.: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
