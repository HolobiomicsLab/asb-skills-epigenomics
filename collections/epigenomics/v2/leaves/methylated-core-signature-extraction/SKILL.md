---
name: methylated-core-signature-extraction
description: Use when bisulfite amplicon-seq epihaplotype tables are available and
  the goal is to identify stable phased mCpG combinations (methylated cores) marking
  epiallele families, or to compute clonality/stability indices revealing dominant
  epiallele families in complex DNA populations.
when_to_use_negative:
- The analysis goal is single-CpG methylation quantification or abundance of individual
  epialleles without phased mCpG combinations — tools like EBA, ampliMethProfiler
  or methclone suffice, as MethCoresProfiler is designed for stable phased mCpG combinations.
- Input data are whole-genome or single-nucleotide-resolution methylation calls rather
  than single-molecule bisulfite amplicon epihaplotypes in binary format.
- Amplified (enriched) epialleles are expected at low abundance (~10%) that escape
  detection by Modules 1 and 2 under the default sampling scheme.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: MethCoresProfiler
  role: Core three-module R tool (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst)
    that extracts significant methylated cores and computes MethCores, Clonality,
    and Entanglement indices
  repo: https://github.com/84AP/MethCoresProfiler/
- name: PEAR
  role: Merges paired-end bisulfite amplicon FASTQ reads with a minimum 40 nucleotide
    overlapping region before epihaplotype calling
- name: PRINSEQ
  role: Converts quality-filtered merged reads to FASTA format prior to ampliMethProfiler
    processing
- name: ampliMethProfiler
  role: Processes FASTA reads under quality filters (length ±50%, ≥80% primer similarity,
    ≥98% bisulfite efficiency, ≥60% base alignment) to produce the binary epihaplotype
    tables that MethCoresProfiler consumes
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: R
  role: Language in which MethCoresProfiler is implemented; scripts require packages
    such as vegan, data.table, FunChisq, ComplexHeatmap, installable via Install_Dependencies.R
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:8cdc27f37cba456adb5b19e7c541497c064cdb6f553af183b71cd52353af3aea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  repo_url: https://github.com/84AP/MethCoresProfiler
  tools:
  - MethCoresProfiler
  - PEAR
  - PRINSEQ
  - R
  - ampliMethProfiler
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# methylated-core-signature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract and trace 'methylated cores' — stable combinations of phased methylated CpGs shared by epiallele families — from bisulfite amplicon data using MethCoresProfiler, quantifying clonality and stability (entanglement) indices to distinguish dominant clonally expanded epiallele families from stochastic methylation.

## When to use

Use when bisulfite amplicon-seq epihaplotype tables are available and the goal is to identify stable phased mCpG combinations (methylated cores) marking epiallele families, or to compute clonality/stability indices revealing dominant epiallele families in complex DNA populations.

## When NOT to use

- The analysis goal is single-CpG methylation quantification or abundance of individual epialleles without phased mCpG combinations — tools like EBA, ampliMethProfiler or methclone suffice, as MethCoresProfiler is designed for stable phased mCpG combinations.
- Input data are whole-genome or single-nucleotide-resolution methylation calls rather than single-molecule bisulfite amplicon epihaplotypes in binary format.
- Amplified (enriched) epialleles are expected at low abundance (~10%) that escape detection by Modules 1 and 2 under the default sampling scheme.

## Inputs

- Tab-delimited text file of EpiHaplotypes in BINARY format (e.g. ampliMethProfiler file.out per sample)
- Tab-delimited text file of CpG positions in the sequence/string (cgpos)
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID (meta.map)
- Bisulfite amplicon-seq paired-end FASTQ reads (e.g. ENA accession PRJEB28662) for upstream epihaplotype generation

## Outputs

- ComposCore table reporting structure and frequency of significant methylated cores
- Tab_Epialleles table of individual epialleles containing significant mCpG combinations
- MethCores Index, Clonality Index, and Stability (Entanglement) Index E per core per sample
- Heatmaps of epiallele structure and frequency, tetrachoric correlations, co-occurrence of two mCpGs, and Shannon/Renyi entropy statistics
- PCA of significant epialleles

## How to apply

First produce binary epihaplotype tables by merging paired-end FASTQ reads with PEAR (minimum 40 nt overlap), retaining reads with mean Phred >33 and 400–500 nt length, converting to FASTA with PRINSEQ, and processing with ampliMethProfiler under filters of length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment, removing reads with ambiguous CpG calls. Then run MethCoresProfiler's three modules: MethCores_Extractor (computes average depth b4, repeated sampling default y = 1000, and annotates significant two-mCpG combinations by chi-square independence testing at P-value ≤ 10−9), MethCores_Combinator (crosses significant 2-mCpG combinations, tests higher-order combinations against expected frequencies for independent events, and outputs ComposCore and Tab_Epialleles), and MethCores_Analyst (hierarchical clustering heatmaps, PCA, comparison of each epiallele to an automatically generated random control R with the same number of CpGs and depth, and extraction of the most frequent shared mCpGs on a decreasing scale starting at 0.9 with minimum 2 mCpGs). The minimum core contains at least two methylated cytosines with statistically significant frequency (chi-square adjusted P ≤ 10−9). Finally, interpret the three indices: MethCores Index (core frequency in the population), Clonality Index (core frequency normalized to average population methylation), and Stability/Entanglement Index E (rate of coupling of at least 2 mCpGs in the core) to identify dominant epiallele families, e.g. comparing P1 versus P90 gut samples.

## Related tools

- **MethCoresProfiler** (Core three-module R tool (MethCores_Extractor, MethCores_Combinator, MethCores_Analyst) that extracts significant methylated cores and computes MethCores, Clonality, and Entanglement indices) — https://github.com/84AP/MethCoresProfiler/
- **PEAR** (Merges paired-end bisulfite amplicon FASTQ reads with a minimum 40 nucleotide overlapping region before epihaplotype calling)
- **PRINSEQ** (Converts quality-filtered merged reads to FASTA format prior to ampliMethProfiler processing)
- **ampliMethProfiler** (Processes FASTA reads under quality filters (length ±50%, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment) to produce the binary epihaplotype tables that MethCoresProfiler consumes) — https://sourceforge.net/projects/amplimethprofiler/
- **R** (Language in which MethCoresProfiler is implemented; scripts require packages such as vegan, data.table, FunChisq, ComplexHeatmap, installable via Install_Dependencies.R) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # with MethCoresProfiler="/path-to-MethCoresProfiler-master/"; Data="testData/"; input="out/"
```

## Evaluation signals

- Results table exists and contains core structures together with clonality index values for each sample.
- All reported cores contain at least two methylated cytosines with chi-square adjusted P-value ≤ 10−9.
- For the DDO1 gut validation, the P90 sample shows a dominant high-clonality methylated core absent or much weaker at P1, reproducing Supplementary Figure S8.
- MethCores index and clonality index are computed for both conditions (e.g. P1 and P90) allowing direct comparison of dominant epiallele family emergence.
- ComposCore and Tab_Epialleles outputs are generated by Module 2, and Module 3 heatmaps mark individual significant epialleles versus the theoretical random control R.

## Limitations

- Amplified epialleles at ~10% abundance under certain conditions escape detection by Modules 1 and 2.
- If no significant epialleles are found in Module 3, only the epiallele with the highest frequency is annotated as the stable signature/core.
- Stable epialleles with phased methylated/non-methylated CpGs cannot be identified by current single-nucleotide techniques due to heterogeneity of 5'-3' ends and dilution by random unstable methylated CpGs — bisulfite amplicon data at single-molecule resolution is required.

## Evidence

- [methods] quality filters: [evidence span withheld — anchor status: not_found]
