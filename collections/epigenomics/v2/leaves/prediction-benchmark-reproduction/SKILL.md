---
name: prediction-benchmark-reproduction
description: Use when a paper reports pathogenicity-prediction success rates for structural
  variants (e.g. 95% vs 66% across two POSTRE modes) and the goal is to rerun the
  tool on its released patient SV files to verify the numbers and per-gene calls.
when_to_use_negative:
- SVs are implicated in recessive phenotypes — POSTRE is oriented towards dominant
  disorders and its capacity for recessive phenotypes has not been specifically evaluated
- SV coordinates are not in hg19 or the phenotype does not fall among the seven supported
  categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system,
  pituitary)
- A quick single-SV query is needed and the online Shiny app (postre-tool.csic.es)
  suffices, since the online version supports only Single SV submissions
tools:
- name: POSTRE_multiSV()
  role: R function run sequentially on the 41-patient SV manifest in Standard and
    High-Specificity modes to generate per-SV and per-gene pathogenicity prediction
    tables
  repo: https://github.com/vicsanga/Postre
- name: POSTRE
  role: Underlying Shiny-based tool whose cell type/tissue-specific coding and long-range
    SV predictions are being benchmarked
  repo: https://github.com/vicsanga/Postre
- name: R
  role: Runtime (>=3.5.0) in which POSTRE_multiSV.R is sourced and the SV manifest
    is loaded with read.delim
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:99d7f499fb5d7c8f6aeedafa653e9b153088f897cc5f0c73ce58de026f1e3cfa
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# prediction-benchmark-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces a published benchmark of SV pathogenicity predictions by re-running POSTRE_multiSV() on patient SV test files in both Standard and High-Specificity modes and comparing per-SV and per-gene outputs against the paper's reported success rates (95% Standard, 66% High-Specificity for 41 patients with long-range pathomechanisms).

## When to use

Use when a paper reports pathogenicity-prediction success rates for structural variants (e.g. 95% vs 66% across two POSTRE modes) and the goal is to rerun the tool on its released patient SV files to verify the numbers and per-gene calls.

## When NOT to use

- SVs are implicated in recessive phenotypes — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- SV coordinates are not in hg19 or the phenotype does not fall among the seven supported categories (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
- A quick single-SV query is needed and the online Shiny app (postre-tool.csic.es) suffices, since the online version supports only Single SV submissions

## Inputs

- Patient SV manifest as a plain text file with 7 tab-separated columns (chromosome 1, hg19 coordinates 1, chromosome 2, hg19 coordinates 2, SV type, phenotype list, unique SV id)
- Postre repository testFiles (e.g. ExampleMultipleSubmission.tsv)
- R environment (>=3.5.0) with POSTRE_multiSV.R sourced and pathTo_Postre_app_Folder set

## Outputs

- pathogenicityPrediction_per_SV: per-SV and per-phenotype pathogenic prediction table
- geneStats_recurrencyAndPathomech: per-gene/per-phenotype/per-mechanism (coding vs long-range) aggregation of predictions
- Per-mode sensitivity comparison against reported benchmarks (95% Standard, 66% High-Specificity) and list of SV ids in the errors element

## How to apply

Download the patient SV test files from the Postre GitHub testFiles folder, using ExampleMultipleSubmission.tsv as the format reference; each SV is one tab-separated line of 7 columns: chromosome 1, hg19 coordinates 1 (comma-separated ranges allowed when not base-pair resolution), chromosome 2, coordinates 2, SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotype list (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary), and a unique SV id. In R (>=3.5.0), source POSTRE_multiSV.R from the Postre_app folder and load the manifest with read.delim(sep="\t", header=FALSE). Call POSTRE_multiSV(SVs, pathTo_Postre_app_Folder) twice — once in Standard mode and once in High-Specificity mode — collecting pathogenicityPrediction_per_SV and geneStats_recurrencyAndPathomech each time. Compute the fraction of SVs predicted pathogenic per phenotype under each mode and diff against the reported rates (95% Standard, 66% High-Specificity); the sensitivity drop in High-Specificity arises because some causative genes (MEF2C, SLC2A1) lack the broad polycomb domains that mode requires.

## Related tools

- **POSTRE_multiSV()** (R function run sequentially on the 41-patient SV manifest in Standard and High-Specificity modes to generate per-SV and per-gene pathogenicity prediction tables) — https://github.com/vicsanga/Postre
- **POSTRE** (Underlying Shiny-based tool whose cell type/tissue-specific coding and long-range SV predictions are being benchmarked) — https://github.com/vicsanga/Postre
- **R** (Runtime (>=3.5.0) in which POSTRE_multiSV.R is sourced and the SV manifest is loaded with read.delim)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); SVs <- read.delim("ExampleMultipleSubmission.tsv", sep="\t", header=FALSE); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Fraction of the 41 SVs predicted pathogenic matches ~95% in Standard mode and ~66% in High-Specificity mode
- geneStats_recurrencyAndPathomech contains the previously proposed causative genes (e.g. SATB2, MEF2C, FOXG1, SLC2A1) with their LOF/GOF coding vs long-range mechanisms
- Per-SV table flags exactly the expected error cases (e.g. MEF2C, SLC2A1 SVs failing High-Specificity due to missing broad polycomb domains)
- Input manifest parses cleanly with read.delim(sep="\t", header=FALSE) yielding a 7-column data frame with valid SV types and phenotype tokens
- errors element of the POSTRE_multiSV() output records only SV ids expected to fail, not formatting failures

## Limitations

- High-Specificity mode misses causative genes such as MEF2C and SLC2A1 because they lack broad polycomb domains, a requirement of that mode
- POSTRE's capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated
- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms; bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case
- Only SVs with two breakpoints and one of the four supported types (Inversion, Translocation, Deletion, Duplication) can be handled; for translocations breakpoint order does not matter, but for other SVs breakpoint 1 must have the smaller coordinate

## Evidence
