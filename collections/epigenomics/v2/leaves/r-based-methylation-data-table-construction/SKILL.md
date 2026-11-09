---
name: r-based-methylation-data-table-construction
description: Use when bisulfite sequencing epiallele tables (e.g. DDO1/DAO FASTA-derived
  EpiHaplotypes) must be compared between single-epiallele frequency tools and MethCoresProfiler
  core outputs to show single-epiallele methods fail to isolate stable methylation
  cores.
when_to_use_negative:
- Input is already core-annotated MethCoresProfiler output (ComposCore/Tab_Epialleles)
  where the question is core tracing, not single-epiallele vs core comparison
- The goal is quantitative methylation of single CpGs rather than structure of epiallele
  populations — the paper notes most programs were designed for single-CpG or single-epiallele
  abundance, not cores
- Reads fail the ampliMethProfiler quality filters (e.g. bisulfite efficiency <98%,
  ambiguous A/G or gaps at CpGs), so epiallele frequencies would be unreliable
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: ampliMethProfiler
  role: Extracts mCpG configurations in single DNA molecules from FASTA reads and
    applies the per-read quality filters used to build the single-epiallele frequency
    columns
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: MethCoresProfiler
  role: R tool providing the core annotations (Tab_Epialleles, ComposCore) that form
    the comparison columns of the table and separate Cond 4 from R1/R2 epialleles
  repo: https://github.com/84AP/MethCoresProfiler/
- name: EBA
  role: Reference single-epiallele frequency method whose output is included as a
    comparison column that fails to distinguish deterministic from random epialleles
- name: R
  role: Language in which the comparison table, chi-square tests and entropy computations
    are scripted; MethCoresProfiler is written in R
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:f0e948331bc2e1395332743b7e5034d9d7b06b0856119d6255f8bdeb4791bc1c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# r-based-methylation-data-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds an R-based side-by-side comparison table of single-epiallele frequency results (EBA/ampliMethProfiler/methclone-style) against MethCoresProfiler core annotations (Tab_Epialleles, ComposCore) on bisulfite reads from the DDO1 (−488 to −44 bp, 6 CpGs) and DAO (+7 to +334 bp, 4 CpGs) regions, to demonstrate that frequent individual epialleles alone cannot separate deterministic (Cond 4) from stochastic (R1/R2) methylated populations.

## When to use

Use when bisulfite sequencing epiallele tables (e.g. DDO1/DAO FASTA-derived EpiHaplotypes) must be compared between single-epiallele frequency tools and MethCoresProfiler core outputs to show single-epiallele methods fail to isolate stable methylation cores.

## When NOT to use

- Input is already core-annotated MethCoresProfiler output (ComposCore/Tab_Epialleles) where the question is core tracing, not single-epiallele vs core comparison
- The goal is quantitative methylation of single CpGs rather than structure of epiallele populations — the paper notes most programs were designed for single-CpG or single-epiallele abundance, not cores
- Reads fail the ampliMethProfiler quality filters (e.g. bisulfite efficiency <98%, ambiguous A/G or gaps at CpGs), so epiallele frequencies would be unreliable

## Inputs

- Processed FASTA bisulfite reads for the DDO1 region (−488 to −44 bp from TSS, 6 CpGs) and DAO region (+7 to +334 bp, 4 CpGs)
- Tab-delimited EpiHaplotypes in BINARY format (file.out) plus a CpG-position table and sample metadata (#SampleID, Tissue, Description, Group, Rep, ID) for MethCoresProfiler Module 1
- MethCoresProfiler Module 2 outputs: ComposCore (core structure/frequency) and Tab_Epialleles (individual epialleles)

## Outputs

- Per-sample single-epiallele frequency distribution table for every DDO1 and DAO sample
- Side-by-side comparison table joining single-epiallele columns with MethCoresProfiler core annotation columns (Tab_Epialleles and ComposCore)
- Flags marking epialleles that are individually significant (chi-square P ≤ 10−9) yet indistinguishable from stochastic R1/R2 background

## How to apply

Load the processed FASTA reads for the DDO1 and DAO regions produced upstream and extract single-molecule methylation configurations with ampliMethProfiler using its quality filters: read length ±50% of the reference, ≥80% primer sequence similarity, ≥98% bisulfite efficiency, ≥60% base alignment to the reference, and removal of reads with ambiguous calls (gaps, A or G) at CpG dinucleotides. Compute per-sample single-epiallele frequency distributions and test each epiallele frequency against a random control (R) with the same number of CpGs and depth using a chi-square test with P-value ≤ 10−9. Join these results with MethCoresProfiler outputs (Tab_Epialleles and ComposCore from MethCores_Extractor/MethCores_Combinator) into one side-by-side R data table, flagging epialleles that are individually significant yet indistinguishable from stochastic background. The rationale is that stochastic mCpGs (e.g. hydroxymethylated cytosines) dilute the stable methylated molecules, so per-epiallele abundance alone cannot isolate the methylated cores that MethCoresProfiler extracts.

## Related tools

- **ampliMethProfiler** (Extracts mCpG configurations in single DNA molecules from FASTA reads and applies the per-read quality filters used to build the single-epiallele frequency columns) — https://sourceforge.net/projects/amplimethprofiler/
- **MethCoresProfiler** (R tool providing the core annotations (Tab_Epialleles, ComposCore) that form the comparison columns of the table and separate Cond 4 from R1/R2 epialleles) — https://github.com/84AP/MethCoresProfiler/
- **EBA** (Reference single-epiallele frequency method whose output is included as a comparison column that fails to distinguish deterministic from random epialleles)
- **R** (Language in which the comparison table, chi-square tests and entropy computations are scripted; MethCoresProfiler is written in R) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after setting MethCoresProfiler="/path-to-MethCoresProfiler-master/" and Data="testData/" in the script
```

## Evaluation signals

- R1/R2 and Cond 4 epialleles share comparable frequency/entropy values in the single-epiallele columns while the MethCoresProfiler core columns differ (reproduces Supplementary Figure S5C behavior)
- Every epiallele frequency test uses a chi-square comparison to a random control of the same CpG number and depth with P-value ≤ 10−9
- All retained reads satisfy the ampliMethProfiler filters: length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, no ambiguous CpG calls
- The comparison table has matched rows across the single-epiallele and core columns for every DDO1 and DAO sample, with core columns traceable to ComposCore/Tab_Epialleles outputs
- Methylated cores appearing in the core columns contain at least two methylated cytosines with statistically significant frequency (adjusted P ≤ 10−9)

## Limitations

- Single-epiallele frequency distributions cannot isolate stable cores: stochastic mCpGs (e.g. hydroxymethylated cytosines) dilute the stable methylated molecules under selection
- Rare amplified epialleles can escape detection: the 10% amplified epiallele(s) in Cond 3 and Cond 4 escape detection by Modules 1 and 2
- MethCoresProfiler has a very heavy dependency list (dozens of R packages including vegan, FunChisq, ComplexHeatmap, mclust); use Install_Dependencies.R, pacman or docker as recommended in the README

## Evidence

- [methods] Extract mCpG configurations with ampliMethProfiler quality filters: [evidence span withheld — anchor status: not_found]
