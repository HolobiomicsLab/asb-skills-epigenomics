---
name: phenotype-stratified-results-tabulation
description: Use when POSTRE has been run in batch (POSTRE_multiSV or Multiple SV
  Submission) over an SV set with attached phenotype lists and you must tally pathogenic
  vs non-pathogenic calls per phenotype association and compute non-pathogenic percentages
  per mode.
when_to_use_negative:
- The SV coordinates are not in hg19 — POSTRE requires hg19 breakpoints, so tabulated
  calls would be invalid until lifted over
- The goal is evaluating recessive phenotypes — the article states POSTRE is oriented
  towards dominant disorders and its capacity for recessive phenotypes has not been
  specifically evaluated
- The input involves phenotype classes outside the supported list (head_neck, limbs,
  neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: POSTRE_multiSV()
  role: R-function batch mode used to generate the 'Results per SV and phenotype'
    table from the SVs data frame
  repo: https://github.com/vicsanga/Postre
- name: Shiny
  role: Framework underlying the Multiple SV Submission mode used as the second running
    mode for cross-mode comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:d52ca382e5e35e545fff427dadfc4f6c92a6b1ccbe5c02248ff5542242df464f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  repo_url: https://github.com/vicsanga/Postre
  tools:
  - POSTRE
  - POSTRE_multiSV()
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# phenotype-stratified-results-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulates POSTRE pathogenicity predictions per SV–phenotype association from the 'Results per SV and phenotype' output tables and computes the non-pathogenic percentage per phenotype group, SV set, and running mode (Standard vs High Specificity), enabling Type-1 error (false-positive) control assessment against healthy-individual SV benchmarks.

## When to use

Use when POSTRE has been run in batch (POSTRE_multiSV or Multiple SV Submission) over an SV set with attached phenotype lists and you must tally pathogenic vs non-pathogenic calls per phenotype association and compute non-pathogenic percentages per mode.

## When NOT to use

- The SV coordinates are not in hg19 — POSTRE requires hg19 breakpoints, so tabulated calls would be invalid until lifted over
- The goal is evaluating recessive phenotypes — the article states POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- The input involves phenotype classes outside the supported list (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)

## Inputs

- 7-column tab-separated plain text SV file (chromosome 1, coordinates 1 in hg19, chromosome 2, coordinates 2, SV type [Inversion/Translocation/Deletion/Duplication], comma-separated phenotype list, unique SV id)
- Healthy-individual (gnomAD) SV sets, including the 500 largest SVs, from the Postre GitHub testFiles folder
- POSTRE batch output: 'Results per SV and phenotype' txt table from each running mode

## Outputs

- Pathogenic vs non-pathogenic tally for every SV-phenotype association analyzed
- Non-pathogenic percentage per SV set and per running mode (Standard, High Specificity)
- Benchmark comparison table against reported values (96.8%/99.2% random 10,000-SV set; 89.5%/98.3% 500 largest SVs)

## How to apply

Run each SV set (e.g. the gnomAD healthy-individual SV sets and the 500 largest SVs from the Postre testFiles folder, in 7-column tab-separated hg19 format: chromosome 1, coordinates 1, chromosome 2, coordinates 2, SV type, comma-separated phenotype list, unique SV id) through both POSTRE running modes. Extract the downloadable txt 'Results per SV and phenotype' table, which contains one labelled pathogenicity prediction for each SV and its associated phenotypes. Tally pathogenic vs non-pathogenic calls for every SV-phenotype association, then compute the non-pathogenic percentage per SV set and per running mode (Standard vs High Specificity). The rationale is that SVs from healthy individuals should be overwhelmingly non-pathogenic, so the non-pathogenic percentage directly measures Type-1 error control; the reported benchmarks are 96.8% Standard / 99.2% High Specificity for the random 10,000-SV set and 89.5% / 98.3% for the 500 largest SVs. Optionally cross-check the aggregated 'Results per gene and phenotype' table, which stratifies predictions per gene, phenotype and pathogenic mechanism (coding vs long-range).

## Related tools

- **POSTRE_multiSV()** (R-function batch mode used to generate the 'Results per SV and phenotype' table from the SVs data frame) — https://github.com/vicsanga/Postre
- **Shiny** (Framework underlying the Multiple SV Submission mode used as the second running mode for cross-mode comparison)

## Examples

```
source('/PathTo/Postre_app/POSTRE_multiSV.R'); results <- POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Reproduced non-pathogenic percentages match reported benchmarks: 96.8% (Standard) and 99.2% (High Specificity) for the random 10,000-SV gnomAD set; 89.5% and 98.3% for the 500 largest SVs
- Every SV-phenotype association in the input set appears in the 'Results per SV and phenotype' table with a labelled pathogenicity prediction (no missing calls)
- Each input SV row parses as exactly 7 tab-separated columns with valid SV types (Inversion, Translocation, Deletion, Duplication) and valid phenotype labels
- Total tallied associations equal the number of SV rows multiplied by the number of phenotypes listed per SV

## Limitations

- POSTRE is oriented towards prediction of SVs causing dominant disorders; recessive phenotypes have not been specifically evaluated
- The prediction of long-range pathomechanisms is limited by the incomplete characterization of enhancers across different human cell types
- Databases used to train SV classifiers are biased towards coding deleterious effects due to the scarcity of reported long-range pathogenic SVs, which affects specificity estimates

## Evidence
