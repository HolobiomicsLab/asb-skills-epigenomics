---
name: methylation-core-significance-testing-chi-square
description: Use when bisulfite amplicon data (paired-end FASTQ, e.g. ENA PRJEB28662)
  has been reduced to binary epihaplotype tables of mCpG configurations and the goal
  is to detect stable phased mCpG combinations and trace epiallele families in complex
  DNA populations.
when_to_use_negative:
- When the goal is only per-CpG methylation quantification or counting individual
  epialleles — the paper notes existing tools (EBA, ampliMethProfiler, methclone)
  handle single-CpG or single-epiallele quantification but cannot distinguish stable
  phased cores (e.g. MethCoresProfiler distinguishes Cond 4 from R1/R2 where they
  fail)
- When reads contain ambiguous CpG calls (gaps, A or G at the CpG dinucleotide) that
  have not been filtered out, or fail ampliMethProfiler quality filters (length ±50%
  of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment)
- When the input is not phased single-molecule bisulfite data (e.g. array methylation
  or unphased consensus methylation levels)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: MethCoresProfiler
  role: R-based three-module tool (Extractor, Combinator, Analyst) that annotates
    significant mCpG combinations, builds core and epiallele tables, and computes
    MethCores, clonality and E indices
  repo: https://github.com/84AP/MethCoresProfiler/
- name: ampliMethProfiler
  role: Extracts per-molecule mCpG configurations from FASTA reads under quality filters
    to produce the binary epihaplotype tables consumed by MethCoresProfiler
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: PEAR
  role: Merges paired-end bisulfite amplicon FASTQ reads with a minimum 40-nucleotide
    overlapping region
- name: PRINSEQ
  role: Converts quality-filtered merged reads to FASTA format for ampliMethProfiler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:461646da0241a699d30fae26fc52458285757515afb86c78d64fa744ad43f119
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# methylation-core-significance-testing-chi-square

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify significant combinations of methylated phased CpGs (methylated cores) shared by epiallele families by chi-square independence testing, as implemented in the three modules of MethCoresProfiler 2.0, and quantify them with MethCores, clonality and entanglement (E) indices.

## When to use

Use when bisulfite amplicon data (paired-end FASTQ, e.g. ENA PRJEB28662) has been reduced to binary epihaplotype tables of mCpG configurations and the goal is to detect stable phased mCpG combinations and trace epiallele families in complex DNA populations.

## When NOT to use

- When the goal is only per-CpG methylation quantification or counting individual epialleles — the paper notes existing tools (EBA, ampliMethProfiler, methclone) handle single-CpG or single-epiallele quantification but cannot distinguish stable phased cores (e.g. MethCoresProfiler distinguishes Cond 4 from R1/R2 where they fail)
- When reads contain ambiguous CpG calls (gaps, A or G at the CpG dinucleotide) that have not been filtered out, or fail ampliMethProfiler quality filters (length ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment)
- When the input is not phased single-molecule bisulfite data (e.g. array methylation or unphased consensus methylation levels)

## Inputs

- Tab-delimited text file of EpiHaplotypes in BINARY format (ampliMethProfiler file.out)
- Tab-delimited text file of CpG positions in the sequence/string (cgpos)
- Sample metadata table with columns #SampleID, Tissue, Description, Group, Rep, ID (meta.map)
- Paired-end bisulfite amplicon FASTQ reads (ENA PRJEB28662) if starting from raw data

## Outputs

- Annotated significant two-mCpG and higher-order mCpG combinations (cores)
- ComposCore and Tab_Epialleles tables
- Heatmaps of epiallele structure and frequency with significant epialleles marked (red = high, blue = low, white = neutral)
- PCA of significant epialleles
- Per-sample MethCores index, Clonality index and Entanglement (E) index
- Shannon entropy and tetrachoric correlations of CpGs from Module 1

## How to apply

First convert reads to binary epihaplotype tables via ampliMethProfiler after merging with PEAR (min 40 nt overlap, Phred >33, 400–500 nt) and PRINSEQ FASTA conversion. Run Module 1 (MethCores_Extractor) with repeated iterative sampling (default y = 1000) at average depth (b4) to annotate significant two-mCpG combinations by chi-square independence testing at P ≤ 1e-9. Run Module 2 (MethCores_Combinator) to cross the significant two-mCpG combinations, compare higher-order (3+ mCpG) combination frequencies to the expected frequency for independent events (chi-square, P ≤ 1e-9), and report ComposCore and Tab_Epialleles tables. Run Module 3 (MethCores_Analyst) to cluster epialleles, compare each epiallele's frequency to an automatically generated random control (R) of the same CpG number and depth, extract the most frequent mCpGs shared by significant epialleles on a 0.9-decreasing percentage scale (minimum two mCpGs), and compute the MethCores index, clonality index and entanglement (E) index. The minimum core must contain at least two methylated cytosines with adjusted chi-square P ≤ 1e-9; install dependencies via Install_Dependencies.R (vegan, Hmisc, FunChisq, entropy, ComplexHeatmap, etc.).

## Related tools

- **MethCoresProfiler** (R-based three-module tool (Extractor, Combinator, Analyst) that annotates significant mCpG combinations, builds core and epiallele tables, and computes MethCores, clonality and E indices) — https://github.com/84AP/MethCoresProfiler/
- **ampliMethProfiler** (Extracts per-molecule mCpG configurations from FASTA reads under quality filters to produce the binary epihaplotype tables consumed by MethCoresProfiler) — https://sourceforge.net/projects/amplimethprofiler/
- **PEAR** (Merges paired-end bisulfite amplicon FASTQ reads with a minimum 40-nucleotide overlapping region)
- **PRINSEQ** (Converts quality-filtered merged reads to FASTA format for ampliMethProfiler)

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after setting MethCoresProfiler=('/path-to-MethCoresProfiler-master/') and Data='testData/', input='out/'
```

## Evaluation signals

- All annotated cores satisfy the minimum-core rule: at least two methylated cytosines with chi-square adjusted P ≤ 1e-9
- Higher-order (3+ mCpG) combination frequencies exceed the expected frequency for independent events (chi-square, P ≤ 1e-9) before annotation
- ComposCore and Tab_Epialleles tables are produced by Module 2, and E/clonality/MethCores indices are produced per sample by Module 3
- On the DDO1/DAO postnatal mouse brain data, reproduce expected patterns such as the stable '−336|−330' association in cerebellum and hippocampus and the cortex switch from '−242|−175' to '−336|−318|−242', consistent with Supplementary Table S3 E-index values
- MethCoresProfiler should separate Cond 4 from R1 and R2 synthetic epialleles where EBA, ampliMethProfiler and methclone do not

## Limitations

- Low-frequency amplified epialleles can escape detection: in the paper's conditions, the 10% amplified epialleles in Cond 3 and Cond 4 escaped detection by Modules 1 and 2
- If no significant epialleles are found, Module 3 falls back to annotating only the highest-frequency epiallele as the stable core
- The statistical threshold is very stringent (P ≤ 10−9); rare cores may be missed, and the pipeline has extensive R dependencies (handled via Install_Dependencies.R, pacman or docker)

## Evidence
