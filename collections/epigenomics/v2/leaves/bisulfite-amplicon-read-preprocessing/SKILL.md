---
name: bisulfite-amplicon-read-preprocessing
description: Use when raw paired-end FASTQ bisulfite amplicon reads must be converted
  into binary single-molecule epihaplotype tables before downstream methylation-core
  or epiallele-family analysis.
when_to_use_negative:
- Input is already a binary epihaplotype table or MethCoresProfiler output — preprocessing
  would only destroy per-molecule CpG calls
- Data are whole-genome bisulfite sequencing reads rather than targeted amplicons;
  the length (400–500 nt), primer-similarity (≥80%), and amplicon-overlap (40 nt)
  filters assume a single amplified locus
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: PEAR
  role: Merges paired-end FASTQ amplicon reads with a minimum 40-nucleotide overlapping
    region
- name: PRINSEQ
  role: Applies quality filtering (Phred >33, length 400–500 nt) and converts merged
    reads to FASTA format
- name: ampliMethProfiler
  role: Extracts per-molecule mCpG configurations from FASTA reads under quality filters,
    producing binary epihaplotype tables for MethCoresProfiler
  repo: https://sourceforge.net/projects/amplimethprofiler/
- name: MethCoresProfiler
  role: Downstream consumer of the binary epihaplotype, CpG-position, and metadata
    files produced by this preprocessing workflow
  repo: https://github.com/84AP/MethCoresProfiler/
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa096
    title: Tracing and tracking epiallele families in complex DNA populations
  source_content_hash: sha256:dc422b76cc0ea524b9fadd8f2a3719367288fc1d6394588154457aaaa6194e72
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bisulfite-amplicon-read-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing pipeline for bisulfite amplicon sequencing reads: merge paired-end FASTQ with PEAR, quality-filter, convert to FASTA with PRINSEQ, and extract per-molecule CpG methylation configurations (binary epihaplotypes) with ampliMethProfiler, producing the input tables required by MethCoresProfiler.

## When to use

Use when raw paired-end FASTQ bisulfite amplicon reads must be converted into binary single-molecule epihaplotype tables before downstream methylation-core or epiallele-family analysis.

## When NOT to use

- Input is already a binary epihaplotype table or MethCoresProfiler output — preprocessing would only destroy per-molecule CpG calls
- Data are whole-genome bisulfite sequencing reads rather than targeted amplicons; the length (400–500 nt), primer-similarity (≥80%), and amplicon-overlap (40 nt) filters assume a single amplified locus

## Inputs

- Paired-end FASTQ reads from bisulfite amplicon sequencing libraries (e.g. ENA accession PRJEB28662)
- Reference amplicon sequence with primer sequences for the target locus (e.g. DDO1/DAO regions)

## Outputs

- Tab-delimited text file of epihaplotypes in BINARY format (ampliMethProfiler file.out, one per sample, placed in an 'out/' folder)
- Tab-delimited text file of CpG positions in the amplicon sequence (cgpos)
- Sample metadata file with columns #SampleID, Tissue, Description, Group, Rep, ID (meta.map)
- Quality-filtered FASTA reads ready for ampliMethProfiler

## How to apply

Start from paired-end FASTQ files (e.g. from ENA accession PRJEB28662) and merge them with PEAR, setting a minimum of 40 nucleotides as the overlapping region. Retain only reads with mean Phred quality score >33 and length between 400 and 500 nucleotides, then convert them to FASTA format with PRINSEQ. Process the FASTA reads with ampliMethProfiler, keeping reads with length ±50% of the reference, ≥80% primer sequence similarity, ≥98% bisulfite efficiency, and ≥60% base alignment to the reference, and removing reads with ambiguous CpG calls (gaps, A, or G at CpG dinucleotides). The rationale is that MethCoresProfiler's Modules 1–3 require a tab-delimited binary epihaplotype file, a CpG-position file, and a sample-metadata file (#SampleID, Tissue, Description, Group, Rep, ID), and poor-quality or ambiguously called reads would corrupt mCpG co-occurrence statistics.

## Related tools

- **PEAR** (Merges paired-end FASTQ amplicon reads with a minimum 40-nucleotide overlapping region)
- **PRINSEQ** (Applies quality filtering (Phred >33, length 400–500 nt) and converts merged reads to FASTA format)
- **ampliMethProfiler** (Extracts per-molecule mCpG configurations from FASTA reads under quality filters, producing binary epihaplotype tables for MethCoresProfiler) — https://sourceforge.net/projects/amplimethprofiler/
- **MethCoresProfiler** (Downstream consumer of the binary epihaplotype, CpG-position, and metadata files produced by this preprocessing workflow) — https://github.com/84AP/MethCoresProfiler/

## Examples

```
pear -f sample_R1.fastq -r sample_R2.fastq -o sample_merged; prinseq-lite.pl -fasta sample_merged.fastq -min_len 400 -max_len 500 -out_good sample_filtered; Rscript ampliMethProfiler.R --input sample_filtered.fasta
```

## Evaluation signals

- All retained reads fall within 400–500 nucleotides with mean Phred score >33 after PEAR/PRINSEQ filtering
- ampliMethProfiler output passes its filters: read length within ±50% of the reference, ≥80% primer similarity, ≥98% bisulfite efficiency, ≥60% of bases aligned to the reference, and no ambiguous CpG calls (gaps/A/G at CpGs)
- The resulting binary epihaplotype files are tab-delimited, one file.out per sample in the input 'out/' folder, and are accepted by MethCoresProfiler's MethCores_Extractor together with cgpos and metadata files
- Bisulfite conversion efficiency per sample reaches ≥98%, indicating reliable C→T conversion before CpG calling
- The number of epihaplotypes per sample is consistent with the average read depth (b4) computed by MethCores_Extractor in subsequent analysis

## Limitations

- Reads with ambiguous calls (including gaps or A or G) at CpG dinucleotides must be removed, so heavily damaged or poorly converted templates are discarded and may bias depth
- The 400–500 nt length window and 40 nt minimum overlap are tailored to the DDO1/DAO amplicons in the paper; other loci with different amplicon sizes require re-tuned thresholds
- The 10% amplified epialleles in Cond 3 and Cond 4 escape detection by downstream Modules 1 and 2, indicating sensitivity limits for rare epialleles at low fractions

## Evidence
