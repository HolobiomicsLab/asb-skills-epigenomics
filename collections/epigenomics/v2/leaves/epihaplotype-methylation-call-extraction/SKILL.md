---
name: epihaplotype-methylation-call-extraction
description: Use when bisulfite amplicon-seq paired-end FASTQ reads (e.g. ENA PRJEB28662)
  must be converted into binary epihaplotype tables of phased per-molecule mCpG calls
  as input for epiallele or methylated-core analysis.
when_to_use_negative:
- Data are whole-genome or reduced-representation bisulfite sequencing reads rather
  than amplicon-scale reads, since ampliMethProfiler and core extraction assume a
  fixed amplicon reference with known CpG positions
- Single-nucleotide (non-phased) methylation quantification is the goal — the article
  notes stable phased epialleles 'cannot be identified' by such techniques, and this
  skill targets phased per-molecule calls
- Input is already a binary epihaplotype table ready for MethCoresProfiler Modules
  1–3; re-running the extraction steps would be redundant
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: PEAR
  role: Merges paired-end FASTQ reads with a minimum of 40 nucleotides as the overlapping
    region
- name: PRINSEQ
  role: Converts quality-filtered merged reads to FASTA format for ampliMethProfiler
    input
- name: ampliMethProfiler
  role: Extracts mCpG configurations in single DNA molecules from FASTA reads under
    quality filters, producing binary epihaplotype tables
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: MethCoresProfiler
  role: 'Downstream consumer: requires the binary epihaplotype table plus CpG position
    and metadata files as inputs to Module 1 (MethCores_Extractor)'
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:596fb187fb538db6f5acf92aa263391741675e42479a7e460fe6539e95e5fd8f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/84AP/MethCoresProfiler
  tools:
  - MethCoresProfiler
  - PEAR
  - PRINSEQ
  - R
  - ampliMethProfiler
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# epihaplotype-methylation-call-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-molecule binary mCpG configurations (epihaplotypes) from bisulfite amplicon-seq reads by merging paired-end FASTQs with PEAR, filtering on quality and length, converting to FASTA with PRINSEQ, and calling methylation states with ampliMethProfiler, producing the binary epihaplotype tables consumed by MethCoresProfiler.

## When to use

Use when bisulfite amplicon-seq paired-end FASTQ reads (e.g. ENA PRJEB28662) must be converted into binary epihaplotype tables of phased per-molecule mCpG calls as input for epiallele or methylated-core analysis.

## When NOT to use

- Data are whole-genome or reduced-representation bisulfite sequencing reads rather than amplicon-scale reads, since ampliMethProfiler and core extraction assume a fixed amplicon reference with known CpG positions
- Single-nucleotide (non-phased) methylation quantification is the goal — the article notes stable phased epialleles 'cannot be identified' by such techniques, and this skill targets phased per-molecule calls
- Input is already a binary epihaplotype table ready for MethCoresProfiler Modules 1–3; re-running the extraction steps would be redundant

## Inputs

- Paired-end bisulfite amplicon-seq FASTQ reads (e.g. from ENA accession PRJEB28662)
- Reference amplicon sequence with primer sequences for the target gene region (e.g. DDO1, DAO)
- CpG position file for the amplicon (tab-delimited, for downstream MethCoresProfiler modules)

## Outputs

- Binary epihaplotype tables (tab-delimited text, one row per molecule, 1/0 mCpG calls per CpG position; the 'file.out' format consumed by MethCores_Extractor)
- Quality-filtered merged reads in FASTA format
- Per-sample methylation call summaries from ampliMethProfiler

## How to apply

Download paired-end bisulfite amplicon FASTQ reads and merge them with PEAR using a minimum of 40 nucleotides as the overlapping region, then retain only reads with mean Phred quality >33 and length between 400 and 500 nucleotides. Convert the filtered reads to FASTA format with PRINSEQ, since ampliMethProfiler consumes FASTA input. Process the FASTA reads with ampliMethProfiler applying its quality filters: read length within ±50% of the reference, ≥80% primer similarity, ≥98% bisulfite efficiency, and ≥60% base alignment with the reference. Remove reads with ambiguous CpG calls (gaps or A or G at CpG dinucleotides) so that every remaining molecule has a fully determined methylation state at each CpG. The rationale is that downstream tools such as MethCoresProfiler require unambiguous binary epihaplotype tables, so strict filtering here prevents stochastic or unphased methylation calls from contaminating the epiallele families.

## Related tools

- **PEAR** (Merges paired-end FASTQ reads with a minimum of 40 nucleotides as the overlapping region)
- **PRINSEQ** (Converts quality-filtered merged reads to FASTA format for ampliMethProfiler input)
- **ampliMethProfiler** (Extracts mCpG configurations in single DNA molecules from FASTA reads under quality filters, producing binary epihaplotype tables) — https://sourceforge.net/projects/amplimethprofiler/
- **MethCoresProfiler** (Downstream consumer: requires the binary epihaplotype table plus CpG position and metadata files as inputs to Module 1 (MethCores_Extractor)) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
Rscript Run_MethCoresProfiler_setup.R  # after PEAR merge, Phred>33/400-500nt filtering, PRINSEQ FASTA conversion, and ampliMethProfiler binary epihaplotype generation (Data="testData/", input="out/")
```

## Evaluation signals

- All retained reads have mean Phred >33 and lengths within 400–500 nucleotides (consistent with the amplicon size)
- ampliMethProfiler filters pass: read length within ±50% of reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% base alignment
- Output epihaplotype table contains no ambiguous CpG calls (no gaps, A, or G at CpG dinucleotides) and binary 1/0 values at each expected CpG position
- Downstream MethCores_Extractor accepts the table without format errors when run with the CpG position and metadata files (#SampleID, Tissue, Description, Group, Rep, ID)
- CpG methylation frequencies in the table are consistent across P1 vs later samples when computed as the single-mCpG profile output of Module 1

## Limitations

- The 10% amplified epialleles in certain synthetic conditions escape detection by MethCoresProfiler Modules 1 and 2 if upstream filtering or depth reduces their representation
- Reads with ambiguous CpG calls must be discarded entirely, which lowers effective depth on molecules with poor conversion or sequencing quality
- The approach assumes amplicon-scale reads (400–500 nt) with known CpG positions and is not applicable to short-read or single-nucleotide methylation analysis

## Evidence
