---
name: hic-fastq-adapter-trimming
description: Use when paired-end Hi-C (or Micro-C/ChIA-PET) FASTQ files must be adapter-
  and quality-trimmed before alignment, e.g. at the start of a Microcket run on MuSC
  Hi-C libraries from GSE189841.
when_to_use_negative:
- Input reads are not 3D-genomics (Hi-C/Micro-C/ChIA-PET) paired-end data, e.g. RNA-seq
  data where STAR-based workflows are expected instead
- Reads are already adapter-trimmed and quality-filtered, so Ktrim trimming would
  be redundant
- Input is not FASTQ (e.g. already-aligned BAM files or processed .pairs files), which
  should skip trimming and go straight to filtering
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Ktrim
  role: 'Adapter- and quality-trimmer invoked by Microcket on the Hi-C paired-end
    FASTQ reads, with sequencing kit set (default: illumina) for adaptor-trimming'
  repo: https://github.com/hellosunking/Ktrim
- name: Microcket
  role: Pipeline wrapper that orchestrates trimming (Ktrim), read stitching (FLASH),
    and BWA alignment for 3D genomics data (version 1.4, Apr 2025)
  repo: https://github.com/hellosunking/Microcket
- name: FLASH
  role: Stitches trimmed read pairs prior to alignment in the Microcket workflow
- name: BWA
  role: Aligns the trimmed/stitched reads against the mm10 (or hg38) genome index
    after Ktrim trimming
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2025.103962
    title: Protocol for investigating chromatin architecture in adult skeletal muscle
      stem cells using high-throughput chromosome conformation capture
  source_content_hash: sha256:9ec77a0c3f29a47aeeac2c03b30951ed6d87f7a52eaa61becf2cc8fe32fc24e9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/hellosunking/Microcket
  repo_url: https://github.com/hellosunking/Ktrim
  tools:
  - BWA
  - FLASH
  - JuicerTools
  - Ktrim
  - Microcket
  - Samtools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hic-fastq-adapter-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs adapter and quality trimming of paired-end Hi-C FASTQ reads as the first step of the Microcket pipeline, using Ktrim with the sequencing-kit-appropriate adapter settings so that only clean reads proceed to stitching (FLASH) and alignment (BWA).

## When to use

Use when paired-end Hi-C (or Micro-C/ChIA-PET) FASTQ files must be adapter- and quality-trimmed before alignment, e.g. at the start of a Microcket run on MuSC Hi-C libraries from GSE189841.

## When NOT to use

- Input reads are not 3D-genomics (Hi-C/Micro-C/ChIA-PET) paired-end data, e.g. RNA-seq data where STAR-based workflows are expected instead
- Reads are already adapter-trimmed and quality-filtered, so Ktrim trimming would be redundant
- Input is not FASTQ (e.g. already-aligned BAM files or processed .pairs files), which should skip trimming and go straight to filtering

## Inputs

- Paired-end Hi-C FASTQ files (read1/read2, one lane per line in an fq.list file), e.g. QSC library SRR17068908/SRR17068909/SRR17068910 and FISC library SRR17068893/SRR17068894 from GSE189841
- Sequencing kit specification (default: illumina) for Ktrim adapter-trimming

## Outputs

- Adapter- and quality-trimmed paired FASTQ reads passed to downstream stitching (FLASH) and BWA alignment
- Per-library trimming counts ('Ktrim' rows) recorded in the Microcket.<sid>.final.stat QC statistics file

## How to apply

Run Ktrim, Microcket's built-in trimming engine, on the read1/read2 FASTQ files listed in the fq.list file, setting the sequencing kit for adapter trimming (default: illumina). Ktrim removes 3' adapters and low-quality bases and reports per-lane statistics (e.g. the 'Ktrim' count in the Microcket.<sid>.final.stat QC file) so trimmed reads can be distinguished from the Total read count. After trimming, Microcket proceeds to read stitching with FLASH and alignment with BWA against a pre-built mm10 (or hg38.p13) index, applying a minimum mapping quality of 10 (-Q) and a minimum alignable proportion of 0.5 (-e). Rationale: untrimmed adapters and poor-quality tails reduce mapping rates; the protocol expects >50% uniquely mapped reads for high-quality Hi-C libraries, and the Ktrim fraction in the final.stat output lets you verify trimming was applied before alignment.

## Related tools

- **Ktrim** (Adapter- and quality-trimmer invoked by Microcket on the Hi-C paired-end FASTQ reads, with sequencing kit set (default: illumina) for adaptor-trimming) — https://github.com/hellosunking/Ktrim
- **Microcket** (Pipeline wrapper that orchestrates trimming (Ktrim), read stitching (FLASH), and BWA alignment for 3D genomics data (version 1.4, Apr 2025)) — https://github.com/hellosunking/Microcket
- **FLASH** (Stitches trimmed read pairs prior to alignment in the Microcket workflow)
- **BWA** (Aligns the trimmed/stitched reads against the mm10 (or hg38) genome index after Ktrim trimming) — https://github.com/lh3/bwa

## Examples

```
bin/bwa index -a bwtsw mm10.fa; microcket -i fq.list -o sample_id
```

## Evaluation signals

- The Microcket.<sid>.final.stat QC file shows a non-zero 'Ktrim' count/fraction, confirming the trimming step executed on the FASTQ input
- 'Total' read counts in the final.stat file match the raw FASTQ read counts of the input lanes before trimming
- Downstream 'Unique' mapping fraction exceeds the expected >50% uniquely-mapped threshold (e.g. 72.5% for QSC GSM5708470 and 70% for FISC GSM5708476 in Table 1)
- No adapter-contaminated reads persist: incomplete-mapping and unpairable fractions remain low relative to total reads (e.g. Incomplete-mapping 1.6%, Unpairable 0.3% in the example stats)

## Limitations

- Ktrim/Microcket is distributed under GPLv3 for personal and academic usage only; commercial use is not permitted
- Microcket has pre-packaged annotation files only for hg38 and mm10; other reference builds require preparing .info and .sam.header files and building the BWA index (which may take ~1 hour with 16 threads)
- On the test dataset (~14 million reads) Microcket finishes in ~10 minutes with 8 threads, but very large libraries may need substantially more time

## Evidence

- [methods] Set sequencing kit for adaptor-trimming. Default: illumina.: "Set sequencing kit for adaptor-trimming. Default: illumina."
- [methods] Microcket depends on Ktrim: "`Microcket` depends on the following tools:
- [Ktrim](https://github.com/hellosunking/Ktrim "Ktrim")"
- [readme] Microcket is an extra-fast and versatile tool for 3D genomics data: "Microcket: an extra-fast and vesatile tool for analysis of 3D genomics data (Hi-C, Micro-C, ChIA-PET, and derivant protocols)"
- [discussion] Table 1 provides alignment statistics as reported by Microcket: "Table 1 provides the alignment statistics for a single replicate (QSC: GSM5708470; FISC: GSM5708476), as reported by Microcket."
- [readme] Ktrim is an extra-fast and accurate adapter- and quality-trimmer: [evidence span withheld — anchor status: external_doc]
