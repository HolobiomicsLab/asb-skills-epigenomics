---
name: quasi-poisson-glm-regression
description: Use when testing whether a candidate driver element's mutation status
  (MUT) is associated with gene expression while correcting for copy number (SCNA)
  and tumour-type covariates, from per-sample quantification tables.
when_to_use_negative:
- The task is driver-element identification itself — use DriverPower's model/infer
  subcommands on the burden/functional-feature tables instead; this skill only validates
  driver candidates at the expression level
- Expression data are raw counts appropriate to an exact Poisson model without overdispersion,
  or already normalized for a method requiring a different error family
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: Python
  role: Used with pandas DataFrames and statsmodels GLM (QuasiPoisson family) to fit
    full and null models and run the likelihood ratio test on per-sample FPKM-UQ expression
    data
- name: DriverPower
  role: Upstream tool that produced the DriverPower-exclusive driver candidates (SGK1,
    GPR126) whose differential expression this GLM analysis tests
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:52a76ff8ba17ae327bdfbad2d85b8ab69627320704193698ac88ea93de50aa41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# quasi-poisson-glm-regression

## Summary

Fit quasi-Poisson generalized linear models to overdispersed per-sample gene-expression counts (PCAWG FPKM-UQ) and compare full vs. null models via likelihood ratio testing, as used to test whether DriverPower-exclusive driver candidates (SGK1 in Lymph-BNHL, GPR126 in Bladder-TCC enhancer) are differentially expressed after copy-number correction.

## When to use

Use when testing whether a candidate driver element's mutation status (MUT) is associated with gene expression while correcting for copy number (SCNA) and tumour-type covariates, from per-sample quantification tables.

## When NOT to use

- The task is driver-element identification itself — use DriverPower's model/infer subcommands on the burden/functional-feature tables instead; this skill only validates driver candidates at the expression level
- Expression data are raw counts appropriate to an exact Poisson model without overdispersion, or already normalized for a method requiring a different error family

## Inputs

- PCAWG per-sample gene-expression quantification matrix (FPKM-UQ) for ~20K genes
- Per-sample somatic mutation status (MUT) for candidate driver elements
- Per-sample copy-number state (SCNA)
- Tissue/cancer-type labels per sample

## Outputs

- expression_glm_pvalues.tsv containing the full per-gene table of likelihood-ratio-test p values
- Copy-number-corrected p values for target genes (e.g. SGK1 p = 3e-13, GPR126 p = 0.012)

## How to apply

Load PCAWG per-sample gene-expression quantification (FPKM-UQ), somatic mutation status (MUT), copy-number state (SCNA), and tissue labels into pandas DataFrames keyed by sample and gene (~20K genes). For each target gene, fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue with statsmodels GLM and the QuasiPoisson family, which handles overdispersion in expression-scale data. Fit the null model FPKM-UQ ~ SCNA + tissue on the same samples and perform a likelihood ratio test between full and null fits to obtain a copy-number-corrected p value per gene. Extract p values for driver-gene pairs of interest (e.g. SGK1 mutated in Lymph-BNHL, GPR126 mutated in Bladder-TCC enhancer) and write the full per-gene table to expression_glm_pvalues.tsv.

## Related tools

- **Python** (Used with pandas DataFrames and statsmodels GLM (QuasiPoisson family) to fit full and null models and run the likelihood ratio test on per-sample FPKM-UQ expression data)
- **DriverPower** (Upstream tool that produced the DriverPower-exclusive driver candidates (SGK1, GPR126) whose differential expression this GLM analysis tests) — https://github.com/smshuai/DriverPower

## Examples

```
import statsmodels.api as sm; full = sm.GLM(y, X, family=sm.families.QuasiPoisson()).fit(); null = sm.GLM(y, X_null, family=sm.families.QuasiPoisson()).fit(); p = 1 - chi2.cdf(2*(full.llf - null.llf), df=1)
```

## Evaluation signals

- Reproduced SGK1 p value matches the reported copy-number-corrected p = 3e-13 (Fig. 2f) within numerical tolerance
- Reproduced GPR126 p value matches the reported p = 0.012 (Fig. 2g) within numerical tolerance
- Full and null models are fitted on identical sample sets, with the null containing exactly SCNA + tissue terms
- expression_glm_pvalues.tsv contains one p value per gene across the ~20K-gene expression matrix

## Limitations

- Validation depends on access to the original PCAWG per-sample FPKM-UQ expression, MUT, and SCNA tables; discrepancy may arise from sample filtering or tissue-covariate coding differences
- This expression-level test validates only a subset of candidates; the skill inherits the paper's limitations such as tumour types with fewer than 30 samples limiting statistical power

## Evidence

- [supplementary] quasi-Poisson GLM with statsmodels: [evidence span withheld — anchor status: not_found]
- [supplementary] LRT protocol: [evidence span withheld — anchor status: not_found]
- [results] reported finding: [evidence span withheld — anchor status: not_found]
- [abstract] data source: "Testing across a collection of 2583 cancer genomes from the PCAWG project, DriverPower identifies 217 coding and 95 non-coding driver candidates."
