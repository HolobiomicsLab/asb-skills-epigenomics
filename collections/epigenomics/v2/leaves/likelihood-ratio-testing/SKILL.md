---
name: likelihood-ratio-testing
description: Use when testing whether a candidate driver element's target gene is
  differentially expressed between samples mutated in that element and wild-type samples,
  while explicitly correcting for somatic copy-number state (SCNA) and tumour tissue
  label.
when_to_use_negative:
- Input is already a raw mutation burden feature table for DriverPower's model/infer
  subcommands rather than per-sample expression quantifications
- Only ~3.7% of the genome was studied; do not use this to assess driver activity
  for regulatory elements without expression or SCNA annotations
- Tumour types with fewer than 30 samples lack statistical power for reliable GLM-based
  expression testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the quasi-Poisson GLM fitting and likelihood ratio test on PCAWG
    expression/mutation/SCNA data
  repo: https://github.com/smshuai/DriverPower
- name: DriverPower
  role: Upstream tool that identifies the driver candidates (e.g. SGK1, GPR126) whose
    expression effects are then tested
  repo: https://github.com/smshuai/DriverPower
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-13929-1
    title: Combined burden and functional impact tests for cancer driver discovery
      using DriverPower
  source_content_hash: sha256:d28580333808b924a9d49c711df48bd44b5528d477be6eb78ba0cee15abf162d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smshuai/DriverPower
  tools:
  - DriverPower
  - Python
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# likelihood-ratio-testing

## Summary

Nested-model likelihood ratio testing to obtain copy-number-corrected differential expression p values for DriverPower-exclusive driver candidates, e.g. SGK1 and GPR126, from PCAWG FPKM-UQ expression data.

## When to use

Use when testing whether a candidate driver element's target gene is differentially expressed between samples mutated in that element and wild-type samples, while explicitly correcting for somatic copy-number state (SCNA) and tumour tissue label.

## When NOT to use

- Input is already a raw mutation burden feature table for DriverPower's model/infer subcommands rather than per-sample expression quantifications
- Only ~3.7% of the genome was studied; do not use this to assess driver activity for regulatory elements without expression or SCNA annotations
- Tumour types with fewer than 30 samples lack statistical power for reliable GLM-based expression testing

## Inputs

- PCAWG per-sample gene-expression quantification (FPKM-UQ), ~20K genes
- Somatic mutation status (MUT) per sample per target element
- Copy-number state (SCNA) per sample per gene
- Tissue/tumour-type labels per sample (e.g. Lymph-BNHL, Bladder-TCC)

## Outputs

- Per-gene copy-number-corrected p values from the likelihood ratio test (expression_glm_pvalues.tsv)
- Significance call per candidate gene, e.g. SGK1 p = 3e-13, GPR126 p = 0.012

## How to apply

Load PCAWG per-sample gene-expression quantification (FPKM-UQ), somatic mutation status (MUT), copy-number state (SCNA), and tissue labels into pandas DataFrames keyed by sample and gene (~20K genes). For each target gene, fit the full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue and the corresponding null model FPKM-UQ ~ SCNA + tissue on the same samples using statsmodels GLM with the QuasiPoisson family. Perform a likelihood ratio test between full and null models to obtain a copy-number-corrected p value per gene, then extract the p values for specific candidates (SGK1 mutated in Lymph-BNHL samples; GPR126 mutated in Bladder-TCC enhancer samples) and write the full per-gene table to expression_glm_pvalues.tsv. The rationale is that DriverPower-exclusive driver candidates must show functional impact beyond burden, so their expression effect must be demonstrated independent of copy-number confounding.

## Related tools

- **Python** (Implements the quasi-Poisson GLM fitting and likelihood ratio test on PCAWG expression/mutation/SCNA data) — https://github.com/smshuai/DriverPower
- **DriverPower** (Upstream tool that identifies the driver candidates (e.g. SGK1, GPR126) whose expression effects are then tested) — https://github.com/smshuai/DriverPower

## Examples

```
import statsmodels.api as sm; full = sm.GLM(y, X_full, family=sm.families.QuasiPoisson()).fit(); null = sm.GLM(y, X_null, family=sm.families.QuasiPoisson()).fit(); p = 1 - chi2.sf(2*(full.llf - null.llf), 1)
```

## Evaluation signals

- Reproduced SGK1 p value matches the reported copy-number-corrected p = 3e-13 within numerical tolerance
- Reproduced GPR126 p value matches the reported p = 0.012 (Fig. 2f, g) within numerical tolerance
- Full and null GLMs are fitted on identical sample sets so the LRT degrees of freedom equal the number of dropped terms (1: MUT)
- expression_glm_pvalues.tsv contains p values for all ~20K genes and can be filtered to the two candidate genes
- SGK1 shows significant overexpression in mutated Lymph-BNHL samples relative to non-mutated samples

## Limitations

- Many tumour types have fewer than 30 samples, limiting statistical power of the per-tissue expression test
- Functional impact score schemes are currently biased toward coding mutations, which can affect candidate prioritization feeding this test
- No driver discovery tool currently models the effect of negatively selected sites; DriverPower's BMR is trained on predominantly neutral regions, limiting recall of candidates entering this analysis

## Evidence

- [full_text] full quasi-Poisson GLM FPKM-UQ ~ MUT + SCNA + tissue using statsmodels GLM with the QuasiPoisson family: [evidence span withheld — anchor status: not_found]
- [full_text] likelihood ratio test between the full and null models to obtain a copy-number-corrected p value: [evidence span withheld — anchor status: not_found]
- [results] SGK1 is significantly overexpressed in mutated Lymph-BNHL samples relative to non-mutated samples: [evidence span withheld — anchor status: not_found]
- [methods] DriverPower requires Python >= 3.5 and some other computing packages.: "DriverPower requires Python >= 3.5 and some other computing packages."
- [readme] DriverPower is a tool used to discover potential coding and non-coding cancer driver elements: "DriverPower is a tool used to discover potential coding and non-coding cancer driver elements from tumour whole-genome or whole-exome somatic mutations."
