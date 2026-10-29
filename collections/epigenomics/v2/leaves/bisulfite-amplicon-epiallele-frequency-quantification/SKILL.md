---
name: bisulfite-amplicon-epiallele-frequency-quantification
description: Use when bisulfite sequencing reads (e.g. FASTA from DDO1/DAO amplicons)
  must be reduced to per-molecule methylation configurations and single-epiallele
  frequency distributions, or when benchmarking these frequencies against MethCoresProfiler
  core annotations.
when_to_use_negative:
- When the goal is to isolate stable methylated cores shared by epiallele families
  in heterogeneous populations — single-epiallele frequency methods fail there; use
  MethCoresProfiler instead
- When reads have not passed bisulfite amplicon QC (e.g. <98% bisulfite conversion
  efficiency or ambiguous A/G calls at CpG dinucleotides), since results will be dominated
  by unreliable methylation calls
- When the input is whole-genome or single-CpG methylation data rather than phased
  amplicon reads covering a defined CpG string
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: ampliMethProfiler
  role: Extracts single-molecule mCpG configurations from FASTA bisulfite amplicon
    reads under quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite
    efficiency, ≥60% alignment, no ambiguous CpG calls)
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: EBA
  role: Reference method for computing epiallele abundance/frequency distributions
    used as the single-epiallele baseline being compared against
- name: MethCoresProfiler
  role: Provides the core annotations (Tab_Epialleles, ComposCore) against which single-epiallele
    frequencies are benchmarked to show stochastic and deterministic epialleles are
    indistinguishable by frequency alone
  repo: https://github.com/84AP/MethCoresProfiler/
- name: R
  role: Runtime environment for MethCoresProfiler modules and for assembling the chi-square-tested
    comparison tables
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:452a0e8d62026a8fb06e3df30cc125ea6d4efb7c9b29b774efb47db770e93ac3
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
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# bisulfite-amplicon-epiallele-frequency-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the abundance of individual epialleles (single-molecule mCpG configurations) from bisulfite amplicon reads using EBA/ampliMethProfiler/methclone-style frequency analysis, and demonstrates why such single-epiallele frequency distributions cannot isolate stable methylated cores in heterogeneous populations.

## When to use

Use when bisulfite sequencing reads (e.g. FASTA from DDO1/DAO amplicons) must be reduced to per-molecule methylation configurations and single-epiallele frequency distributions, or when benchmarking these frequencies against MethCoresProfiler core annotations.

## When NOT to use

- When the goal is to isolate stable methylated cores shared by epiallele families in heterogeneous populations — single-epiallele frequency methods fail there; use MethCoresProfiler instead
- When reads have not passed bisulfite amplicon QC (e.g. <98% bisulfite conversion efficiency or ambiguous A/G calls at CpG dinucleotides), since results will be dominated by unreliable methylation calls
- When the input is whole-genome or single-CpG methylation data rather than phased amplicon reads covering a defined CpG string

## Inputs

- Processed FASTA reads from bisulfite amplicon sequencing (e.g. DDO1 region −488 to −44 bp, 6 CpGs; DAO region +7 to +334 bp, 4 CpGs), produced upstream via PEAR merging (min 40 nt overlap) and PRINSEQ FASTA conversion
- Reference sequences with CpG positions and primer sequences for each amplicon region
- Sample metadata (e.g. SampleID, Tissue, Description, Group, Rep, ID)
- MethCoresProfiler core annotations (Tab_Epialleles, ComposCore) for the same samples, for the side-by-side comparison

## Outputs

- Per-sample single-molecule methylation configurations extracted by ampliMethProfiler
- Per-sample single-epiallele frequency distributions (abundance of individual epialleles)
- Chi-square significance calls for each epiallele frequency vs. a random control of the same CpG number and depth (P ≤ 10−9)
- Side-by-side comparison table of single-epiallele metrics vs. MethCoresProfiler core annotations, flagging individually significant epialleles that are indistinguishable from stochastic background

## How to apply

Load the processed FASTA reads for the target regions (e.g. DDO1, −488 to −44 bp from the TSS with 6 CpGs; DAO, +7 to +334 bp with 4 CpGs) and process them with ampliMethProfiler applying its quality filters: read length within ±50% of the reference length, ≥80% primer sequence similarity, ≥98% bisulfite efficiency, ≥60% base alignment to the reference, and removal of reads with ambiguous calls (gaps, A or G) at CpG dinucleotides. Extract single-molecule mCpG configurations and compute per-sample single-epiallele frequency distributions (EBA/methclone-style abundance of individual epialleles). Test each individual epiallele frequency against a random control with the same number of CpGs and depth (theoretical distribution) using a chi-square test with P-value ≤ 10−9. Finally, join these single-epiallele results with MethCoresProfiler core annotations (Tab_Epialleles and ComposCore outputs) into a side-by-side comparison table: R1/R2 (random) and Cond 4 (deterministic) epialleles share comparable frequency/entropy values in the single-epiallele columns while the core columns differ, because stochastic mCpGs (e.g. hydroxymethylated cytosines) dilute the stable methylated molecules under selection.

## Related tools

- **ampliMethProfiler** (Extracts single-molecule mCpG configurations from FASTA bisulfite amplicon reads under quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% alignment, no ambiguous CpG calls)) — https://sourceforge.net/projects/amplimethprofiler/
- **EBA** (Reference method for computing epiallele abundance/frequency distributions used as the single-epiallele baseline being compared against)
- **MethCoresProfiler** (Provides the core annotations (Tab_Epialleles, ComposCore) against which single-epiallele frequencies are benchmarked to show stochastic and deterministic epialleles are indistinguishable by frequency alone) — https://github.com/84AP/MethCoresProfiler/
- **R** (Runtime environment for MethCoresProfiler modules and for assembling the chi-square-tested comparison tables) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
source("Run_MethCoresProfiler_setup.R")  # after setting MethCoresProfiler=("/path-to-MethCoresProfiler-master/") and Data="testData/"; FASTA reads first processed with ampliMethProfiler
```

## Evaluation signals

- Reproduces the paper's reported behavior (Supplementary Figure S5C): EBA, ampliMethProfiler and methclone identify frequent individual epialleles but do not distinguish random R1/R2 epialleles from deterministic Cond 4 epialleles
- R1/R2 and Cond 4 epialleles show comparable frequency/entropy values in the single-epiallele columns of the comparison table while the MethCoresProfiler core columns differ
- All reported epiallele frequency significances were tested by chi-square against a random control with the same number of CpGs and depth at P-value ≤ 10−9
- Every read retained for epiallele calling satisfied all ampliMethProfiler filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% aligned bases, no gaps/A/G at CpG dinucleotides)
- Per-sample epiallele counts and frequencies are consistent with the input FASTA depth for each DDO1 and DAO sample

## Limitations

- Single-epiallele frequency distributions cannot distinguish random unstable methylated CpGs (including stochastic mCpGs such as hydroxymethylated cytosines) from stable methylated molecules under selection — the stable cores are diluted
- Stable epialleles with phased methylated/non-methylated CpGs cannot be identified by current single-nucleotide techniques due to heterogeneity of 5'-3' ends of molecules
- The 10% amplified epialleles in Cond 3 and Cond 4 escape detection by Modules 1 and 2 of MethCoresProfiler, so frequency-based significance alone misses rare deterministic subpopulations

## Evidence

- [methods] ampliMethProfiler quality filters: "reads in FASTA format were processed using ampliMethProfiler (14,19,20) (available at https://sourceforge.net/projects/amplimethprofiler/) applying several quality filters"
- [methods] ampliMethProfiler filter thresholds: "we retained only reads characterized by (i) length ±50% compared with the reference length"
- [methods] ambiguous CpG call removal: "Reads with ambiguous calls (including gaps or A or G) at the CpG dinucleotide were removed"
- [methods] chi-square vs random control: "the frequency of each epiallele in the sample is compared (chi-square test) to a random control (R), generated automatically by Module 1, with the same number of CpGs and depth"
- [methods] MethCoresProfiler outperforms single-epiallele methods: "MethCoresProfiler outperforms the other methods used to analyse epialleles because it distinguishes Cond 4 from R1 and R2 epialleles."
- [methods] core definition threshold: "The minimum core contains at least two methylated cytosines with statistically significant frequency (chi-square test adjusted P-value ≤10−9)."
