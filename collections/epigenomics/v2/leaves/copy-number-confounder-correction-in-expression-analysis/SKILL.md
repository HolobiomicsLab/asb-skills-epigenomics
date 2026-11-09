---
name: copy-number-confounder-correction-in-expression-analysis
description: Use when testing whether samples carrying mutations in a candidate driver
  element show differential expression of a target gene, and you must correct for
  copy-number state and tissue-of-origin confounding.
when_to_use_negative:
- Input is already a normalized expression matrix with copy-number effects removed
  — refitting the SCNA term would double-correct
- The goal is calling driver elements themselves (burden/functional impact testing)
  rather than validating downstream expression effects — use DriverPower model/infer
  instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: Python (statsmodels, pandas)
  role: Fits the full quasi-Poisson GLM (FPKM-UQ ~ MUT + SCNA + tissue) and null model,
    and computes the likelihood ratio test p values
- name: DriverPower
  role: Upstream method whose exclusive driver candidates (SGK1, GPR126) define the
    mutated samples tested in the expression analysis
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:d842f6d6e006165a7a53915336cf799bc4268268eb894b190f042073b4cf5624
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python (statsmodels, pandas)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# copy-number-confounder-correction-in-expression-analysis

## Summary

Reproduces the DriverPower paper's copy-number-corrected differential expression test, in which per-gene FPKM-UQ expression from PCAWG samples is modeled with a quasi-Poisson GLM including mutation status (MUT), copy-number state (SCNA), and tissue, and a likelihood ratio test against a null model without MUT yields a copy-number-corrected p value (e.g. SGK1 p = 3e-13, GPR126 p = 0.012).

## When to use

Use when testing whether samples carrying mutations in a candidate driver element show differential expression of a target gene, and you must correct for copy-number state and tissue-of-origin confounding.

## When NOT to use

- Input is already a normalized expression matrix with copy-number effects removed — refitting the SCNA term would double-correct
- The goal is calling driver elements themselves (burden/functional impact testing) rather than validating downstream expression effects — use DriverPower model/infer instead

## Inputs

- PCAWG per-sample gene-expression quantification table (FPKM-UQ), keyed by sample and gene (~20K genes)
- Per-sample somatic mutation status (MUT) for the candidate driver elements
- Per-sample copy-number state (SCNA) annotations
- Per-sample tissue/tumour-type labels (e.g. Lymph-BNHL, Bladder-TCC)

## Outputs

- Copy-number-corrected per-gene p values from likelihood ratio tests (expression_glm_pvalues.tsv)
- Validated p values for SGK1 (Lymph-BNHL) and GPR126 (Bladder-TCC enhancer) comparable to Fig. 2f, g

## How to apply

Load PCAWG per-sample gene-expression quantifications (FPKM-UQ), somatic mutation status (MUT), copy-number state (SCNA), and tissue labels into pandas DataFrames keyed by sample and gene (~20K genes). For each target gene, fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue with statsmodels GLM (QuasiPoisson family), then fit the null model FPKM-UQ ~ SCNA + tissue on the same samples. Perform a likelihood ratio test between the full and null models to obtain the copy-number-corrected p value; this removes SCNA-driven expression differences so that only the mutational effect is tested. Restrict each gene's test to the tumour type where the element was called (e.g. SGK1 in Lymph-BNHL, GPR126 enhancer in Bladder-TCC). Write the full per-gene table to expression_glm_pvalues.tsv and validate reproduced p values against Fig. 2f, g (SGK1 p = 3e-13; GPR126 p = 0.012).

## Related tools

- **Python (statsmodels, pandas)** (Fits the full quasi-Poisson GLM (FPKM-UQ ~ MUT + SCNA + tissue) and null model, and computes the likelihood ratio test p values)
- **DriverPower** (Upstream method whose exclusive driver candidates (SGK1, GPR126) define the mutated samples tested in the expression analysis) — https://github.com/smshuai/DriverPower

## Examples

```
import statsmodels.api as sm; full = sm.GLM(fpkm, sm.add_constant(X[['MUT','SCNA','tissue']]), family=sm.families.Poisson()).fit(); null = sm.GLM(fpkm, sm.add_constant(X[['SCNA','tissue']]), family=sm.families.Poisson()).fit(); p = 1 - sp.stats.chi2.cdf(2*(full.llf - null.llf), 1)
```

## Evaluation signals

- Reproduced SGK1 p value ≈ 3e-13 (copy-number-corrected likelihood ratio test, Fig. 2f)
- Reproduced GPR126 p value ≈ 0.012 (Fig. 2g), both within numerical tolerance
- SGK1 is overexpressed in mutated Lymph-BNHL samples relative to non-mutated samples
- Output file expression_glm_pvalues.tsv contains a complete per-gene p-value table for all ~20K genes
- Full and null GLMs are fitted on exactly the same sample sets, so the LRT degrees of freedom equal 1 (the dropped MUT term)

## Limitations

- DriverPower provides candidate driver elements, not functional validation — expression testing is a downstream corroboration step, not a driver call
- False-positive associations can arise from technical artefacts or biological processes not captured by covariates; DriverPower authors note variant-calling artefacts and processes like regional AID activity are not modeled
- Many PCAWG tumour types have fewer than 30 samples, limiting statistical power for within-type expression tests
- Functional impact scores are biased toward coding mutations, so non-coding candidates like the GPR126 enhancer rely heavily on such downstream evidence

## Evidence

- [methods] full quasi-Poisson GLM: [evidence span withheld — anchor status: not_found]
- [results] SGK1 expression finding: [evidence span withheld — anchor status: not_found]
- [readme] DriverPower tool scope: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
