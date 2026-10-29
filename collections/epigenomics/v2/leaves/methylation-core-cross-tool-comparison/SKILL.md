---
name: methylation-core-cross-tool-comparison
description: Use when bisulfite sequencing reads (e.g. FASTA epiallele tables from
  DDO1/DAO amplicons) have been processed with both single-epiallele tools and MethCoresProfiler,
  and the goal is to demonstrate that epiallele frequency distributions alone miss
  stable methylation cores.
when_to_use_negative:
- When the goal is to quantify single-CpG methylation levels rather than to compare
  epiallele-level versus core-level discriminative power
- When MethCoresProfiler core annotations (Tab_Epialleles, ComposCore) are unavailable
  and only single-epiallele frequencies can be computed — the comparison cannot be
  made
- When the input is raw FASTQ rather than quality-filtered, converted FASTA reads
  produced by the upstream PEAR/PRINSEQ/ampliMethProfiler workflow
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: ampliMethProfiler
  role: Extracts single-molecule mCpG configurations and epiallele frequencies from
    FASTA reads under quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite
    efficiency, ≥60% base alignment, no ambiguous CpG calls); represents the single-epiallele
    baseline that fails to separate R1/R2 from Cond 4
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: EBA
  role: Existing epiallele distribution tool whose individual-epiallele frequency
    output is used as the comparison baseline that misses stable cores
- name: MethCoresProfiler
  role: Provides the methylated-core annotations (via MethCores_Extractor, MethCores_Combinator
    and MethCores_Analyst modules; Tab_Epialleles and ComposCore tables) that successfully
    distinguish Cond 4 from R1 and R2 epialleles
  repo: https://github.com/84AP/MethCoresProfiler/
- name: R
  role: Language in which MethCoresProfiler is implemented and in which the comparison
    table and chi-square tests are assembled
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:11b8d958ba6c7894902d7faebf2d23cb057cd96f9fa74a7515c781bb553b0f26
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# methylation-core-cross-tool-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the side-by-side evaluation showing that single-epiallele frequency tools (EBA, ampliMethProfiler, methclone) identify frequent individual epialleles but cannot separate random (R1/R2) from deterministic (Cond 4) epialleles, whereas MethCoresProfiler's methylated cores distinguish them.

## When to use

Use when bisulfite sequencing reads (e.g. FASTA epiallele tables from DDO1/DAO amplicons) have been processed with both single-epiallele tools and MethCoresProfiler, and the goal is to demonstrate that epiallele frequency distributions alone miss stable methylation cores.

## When NOT to use

- When the goal is to quantify single-CpG methylation levels rather than to compare epiallele-level versus core-level discriminative power
- When MethCoresProfiler core annotations (Tab_Epialleles, ComposCore) are unavailable and only single-epiallele frequencies can be computed — the comparison cannot be made
- When the input is raw FASTQ rather than quality-filtered, converted FASTA reads produced by the upstream PEAR/PRINSEQ/ampliMethProfiler workflow

## Inputs

- Processed FASTA reads for DDO1 (−488 to −44 bp, 6 CpGs) and DAO (+7 to +334 bp, 4 CpGs) regions
- Tab-delimited EpiHaplotypes file in BINARY format, CpG position file, and sample metadata (#SampleID, Tissue, Description, Group, Rep, ID) for MethCoresProfiler
- MethCoresProfiler Module 2/3 outputs: Tab_Epialleles and ComposCore tables
- Random control (R) epiallele distributions with same CpG number and depth

## Outputs

- Per-sample single-epiallele frequency distribution table (EBA/methclone-style) for all DDO1 and DAO samples
- Chi-square significance calls for individual epialleles versus the random control (P ≤ 10−9)
- Side-by-side comparison table joining single-epiallele results with MethCoresProfiler core annotations, flagging epialleles that are individually significant yet indistinguishable from stochastic background

## How to apply

Load the processed FASTA reads for the DDO1 region (−488 to −44 bp from the TSS, 6 CpGs) and DAO region (+7 to +334 bp, 4 CpGs), then extract single-molecule methylation configurations with ampliMethProfiler applying its quality filters: read length ±50% of the reference, ≥80% primer sequence similarity, ≥98% bisulfite efficiency, ≥60% base alignment, and removal of reads with ambiguous calls (gaps, A or G) at CpG dinucleotides. Compute per-sample single-epiallele frequency distributions (EBA/methclone-style abundance) and test each epiallele's frequency against a random control (R) with the same number of CpGs and depth by chi-square test at P ≤ 10−9. Join these results with MethCoresProfiler's core annotations (Tab_Epialleles and ComposCore outputs from Modules 1–3) into a side-by-side comparison table. The rationale is that stochastic mCpGs (e.g. hydroxymethylated cytosines) dilute stable methylated molecules, so single-epiallele frequencies cannot isolate the deterministic cores that MethCoresProfiler's modules extract and annotate with clonality and stability indices.

## Related tools

- **ampliMethProfiler** (Extracts single-molecule mCpG configurations and epiallele frequencies from FASTA reads under quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, no ambiguous CpG calls); represents the single-epiallele baseline that fails to separate R1/R2 from Cond 4) — https://sourceforge.net/projects/amplimethprofiler/
- **EBA** (Existing epiallele distribution tool whose individual-epiallele frequency output is used as the comparison baseline that misses stable cores)
- **MethCoresProfiler** (Provides the methylated-core annotations (via MethCores_Extractor, MethCores_Combinator and MethCores_Analyst modules; Tab_Epialleles and ComposCore tables) that successfully distinguish Cond 4 from R1 and R2 epialleles) — https://github.com/84AP/MethCoresProfiler/
- **R** (Language in which MethCoresProfiler is implemented and in which the comparison table and chi-square tests are assembled) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after setting MethCoresProfiler="/path-to-MethCoresProfiler-master/" and Data="testData/" per README
```

## Evaluation signals

- The comparison table reproduces the paper's reported behavior: R1/R2 and Cond 4 epialleles share comparable frequency/entropy values in the single-epiallele columns while the core columns differ
- EBA, ampliMethProfiler and methclone identify frequent individual epialleles (matching Supplementary Figure S5C) but flag them as significant without separating random from deterministic ones
- Chi-square tests of individual epiallele frequencies use a random control with identical CpG number and depth and threshold P ≤ 10−9
- ampliMethProfiler filter compliance is verifiable: only reads with length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% aligned bases, and no gaps/A/G at CpG dinucleotides are retained
- MethCoresProfiler core outputs (Tab_Epialleles, ComposCore) contain cores with at least two methylated cytosines with statistically significant frequency (chi-square adjusted P ≤ 10−9)

## Limitations

- Single-epiallele methods dilute stable methylated molecules because stochastic mCpGs (e.g. hydroxymethylated cytosines) and the heterogeneity of 5'-3' ends of molecules obscure the cores under selection
- The 10% amplified epialleles in Cond 3 and Cond 4 escape detection by MethCoresProfiler Modules 1 and 2, so the comparison relies on Module 3 and the deterministic Cond 4 design
- Key comparison evidence (Supplementary Table S4, Figure S5C) resides in supplementary materials and is not reproduced in the main text sections

## Evidence
