---
name: single-epiallele-significance-testing-vs-random-control
description: 'Use when benchmarking bisulfite-seq epiallele analysis: you need per-epiallele
  frequency/entropy values and chi-square significance against a theoretical random
  (R) control to show whether frequent individual epialleles are deterministic or
  stochastic.'
when_to_use_negative:
- When the goal is to isolate stable methylated cores or families of related epialleles
  — single-epiallele frequency testing is exactly the approach shown to fail here;
  use MethCoresProfiler instead
- When reads have not passed bisulfite quality filtering (98% bisulfite efficiency,
  ambiguous CpG calls removed) — unfiltered reads will inflate spurious epiallele
  frequencies
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: ampliMethProfiler
  role: Extracts single-molecule mCpG configurations and epiallele frequencies from
    FASTA reads under its quality filters
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: EBA
  role: Reference single-epiallele frequency method whose results are reproduced for
    the comparison
- name: methclone
  role: Reference single-epiallele abundance tool included in the comparative benchmark
- name: MethCoresProfiler
  role: Provides the core annotations (Tab_Epialleles, ComposCore) joined to the single-epiallele
    results and generates the random control R and chi-square tests in Modules 1/3
  repo: https://github.com/84AP/MethCoresProfiler/
- name: R
  role: Runtime environment for MethCoresProfiler modules and downstream comparison
    scripts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:2e5d5046dac9a322adeed6f12ffd1d5994285bc1a42da2507cc4218dfa63585d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  tools:
  - EBA
  - MethCoresProfiler
  - R
  - ampliMethProfiler
  - methclone
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-epiallele-significance-testing-vs-random-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A comparison/validation skill that computes single-epiallele frequency distributions (EBA/ampliMethProfiler/methclone-style) and tests each individual epiallele's frequency against a random control with the same number of CpGs and depth, demonstrating why single-epiallele methods cannot separate deterministic epialleles from stochastic background.

## When to use

Use when benchmarking bisulfite-seq epiallele analysis: you need per-epiallele frequency/entropy values and chi-square significance against a theoretical random (R) control to show whether frequent individual epialleles are deterministic or stochastic.

## When NOT to use

- When the goal is to isolate stable methylated cores or families of related epialleles — single-epiallele frequency testing is exactly the approach shown to fail here; use MethCoresProfiler instead
- When reads have not passed bisulfite quality filtering (98% bisulfite efficiency, ambiguous CpG calls removed) — unfiltered reads will inflate spurious epiallele frequencies

## Inputs

- Processed FASTA bisulfite sequencing reads (e.g. PRJEB28662 reads merged with PEAR, converted with PRINSEQ)
- Sample metadata (e.g. DDO1 and DAO regions with 6 and 4 CpGs respectively)
- MethCoresProfiler core annotations (Tab_Epialleles, ComposCore) for side-by-side comparison

## Outputs

- Per-sample single-epiallele frequency distribution table
- Chi-square significance calls (P ≤ 10−9) for each individual epiallele versus the random control
- Shannon and Renyi entropy values per epiallele population
- Side-by-side comparison table flagging individually significant epialleles that are indistinguishable from stochastic background

## How to apply

Load processed FASTA bisulfite reads (e.g. DDO1 −488/−44 bp, 6 CpGs; DAO +7/+334 bp, 4 CpGs) and extract single-molecule methylation configurations with ampliMethProfiler under its quality filters: read length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, and removal of reads with ambiguous calls (gaps, A or G) at CpG dinucleotides. Compute per-sample single-epiallele frequency distributions and Shannon/Renyi entropy, then test each individual epiallele's frequency (chi-square test) against a random control (R) generated automatically with the same number of CpGs and depth, applying P ≤ 10−9. Join the significant single-epiallele results with MethCoresProfiler core annotations (Tab_Epialleles and ComposCore) into a side-by-side table. The rationale: stochastic mCpGs (e.g. hydroxymethylated cytosines) dilute stable methylated molecules, so single-epiallele frequencies alone flag R1/R2 random epialleles and deterministic Cond 4 epialleles with comparable frequency/entropy, while only core-based annotation separates them.

## Related tools

- **ampliMethProfiler** (Extracts single-molecule mCpG configurations and epiallele frequencies from FASTA reads under its quality filters) — https://sourceforge.net/projects/amplimethprofiler/
- **EBA** (Reference single-epiallele frequency method whose results are reproduced for the comparison)
- **methclone** (Reference single-epiallele abundance tool included in the comparative benchmark)
- **MethCoresProfiler** (Provides the core annotations (Tab_Epialleles, ComposCore) joined to the single-epiallele results and generates the random control R and chi-square tests in Modules 1/3) — https://github.com/84AP/MethCoresProfiler/
- **R** (Runtime environment for MethCoresProfiler modules and downstream comparison scripts)

## Examples

```
Rscript Run_MethCoresProfiler_setup.R
```

## Evaluation signals

- Each epiallele significance call uses a chi-square test against a random control with the same number of CpGs and depth, at P ≤ 10−9
- Reproduced behavior: R1/R2 random epialleles and deterministic Cond 4 epialleles share comparable frequency/entropy values in the single-epiallele columns (Supplementary Figure S5C behavior)
- The same epialleles are separated by the MethCoresProfiler core columns, confirming the contrast is methodological, not data-driven
- Only reads passing ampliMethProfiler filters (±50% length, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% alignment, no ambiguous CpG calls) contribute to frequency tables
- Entropy metrics (Shannon and Renyi) are computable for every sample's epiallele population

## Limitations

- Single-epiallele frequency distributions cannot distinguish random R1/R2 epialleles from deterministic Cond 4 epialleles — stochastic mCpGs dilute stable methylated molecules under selection
- Frequent individual epialleles passing the P ≤ 10−9 chi-square threshold may still be indistinguishable from stochastic background without core-based annotation
- 10% amplified epialleles in Cond 3 and Cond 4 escape detection by MethCoresProfiler Modules 1 and 2

## Evidence
