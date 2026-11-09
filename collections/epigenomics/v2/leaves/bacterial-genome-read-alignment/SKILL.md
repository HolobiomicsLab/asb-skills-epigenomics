---
name: bacterial-genome-read-alignment
description: Use when the input is adapter/quality-trimmed ChIP-seq FASTQ from a bacterial
  sample and the goal is genome-mapped reads (BAM) as the substrate for transcription
  factor peak calling with macs2.
when_to_use_negative:
- Input is RNA-seq rather than ChIP-seq data — this skill targets ChIP-seq read mapping
  as the front end of macs2 peak calling
- The reference genome is not V. harveyi ATCC BAA-1116 and strain-specific parameters/indexing
  would need to be re-derived
- Reads are raw and untrimmed; Trimmomatic adapter/quality trimming should precede
  alignment
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: bwa
  role: Indexes the V. harveyi ATCC BAA-1116 reference genome (bwa index) and aligns
    trimmed ChIP-seq reads (bwa mem) to produce BAM files
- name: Trimmomatic
  role: Upstream trimming of adapter sequences and low-quality bases from ChIP-seq
    FASTQ before alignment
- name: macs2
  role: Downstream peak caller consuming the sorted/indexed BAM alignments to identify
    LuxR and H-NS binding peaks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:bf712a3e45bb2b6407b8d04f4219fa2f6eb94b9123f7ad290d1831ed93d442fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - JBrowse
  - Trimmomatic
  - bwa
  - macs2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bacterial-genome-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns trimmed ChIP-seq reads to a compact bacterial reference genome (V. harveyi ATCC BAA-1116) using bwa index/bwa mem, then sorts and indexes the BAMs so downstream macs2 peak calling can quantify LuxR and H-NS binding peaks.

## When to use

Use when the input is adapter/quality-trimmed ChIP-seq FASTQ from a bacterial sample and the goal is genome-mapped reads (BAM) as the substrate for transcription factor peak calling with macs2.

## When NOT to use

- Input is RNA-seq rather than ChIP-seq data — this skill targets ChIP-seq read mapping as the front end of macs2 peak calling
- The reference genome is not V. harveyi ATCC BAA-1116 and strain-specific parameters/indexing would need to be re-derived
- Reads are raw and untrimmed; Trimmomatic adapter/quality trimming should precede alignment

## Inputs

- Trimmomatic-trimmed ChIP-seq FASTQ files for LuxR and H-NS immunoprecipitation samples (GEO accession GSE136050)
- V. harveyi ATCC BAA-1116 reference genome sequence (FASTA)

## Outputs

- Sorted and indexed BAM files per ChIP-seq sample and condition (wild-type and ΔluxR)
- bwa index files for the V. harveyi ATCC BAA-1116 reference genome
- Mapped reads feeding macs2 peak calling for LuxR/H-NS binding peaks

## How to apply

First build a reference index for the V. harveyi ATCC BAA-1116 genome with `bwa index`, since small bacterial genomes index quickly and bwa mem handles short Illumina reads well. Align each Trimmomatic-trimmed FASTQ from the LuxR and H-NS immunoprecipitation samples (GEO GSE136050) with `bwa mem` against the indexed reference. Sort and index the resulting BAM files so macs2 can access them efficiently. Retain the sorted/indexed BAMs per sample and condition (wild-type and ΔluxR background) because the subsequent macs2 step depends on condition-specific mapping; validate by checking read counts and that peak calling downstream reproduces 506 LuxR and 237/302 H-NS peaks.

## Related tools

- **bwa** (Indexes the V. harveyi ATCC BAA-1116 reference genome (bwa index) and aligns trimmed ChIP-seq reads (bwa mem) to produce BAM files)
- **Trimmomatic** (Upstream trimming of adapter sequences and low-quality bases from ChIP-seq FASTQ before alignment)
- **macs2** (Downstream peak caller consuming the sorted/indexed BAM alignments to identify LuxR and H-NS binding peaks)

## Examples

```
bwa index Vharveyi_BAA1116.fa && bwa mem Vharveyi_BAA1116.fa trimmed_LuxR_IP.fq | samtools sort -o LuxR_IP.sorted.bam && samtools index LuxR_IP.sorted.bam
```

## Evaluation signals

- Each sorted BAM file exists, is indexed (BAI present), and is non-empty for every LuxR and H-NS sample
- Alignment is against the correct reference (V. harveyi ATCC BAA-1116), confirmed by the bwa index build step
- Downstream macs2 peak counts match the reported values: 506 LuxR peaks, 237 H-NS peaks in wild-type, 302 H-NS peaks in the ΔluxR background
- Peak coordinate files (narrowPeak/BED) corresponding to each BAM are produced and non-empty
- Per-sample peak counts recorded in chipseq_peak_counts.csv agree with the published discussion values

## Limitations

- Reported peak counts are specific to the GSE136050 dataset and this pipeline (Trimmomatic, bwa mem, macs2); re-running on other datasets or parameters will not necessarily reproduce 506 LuxR / 237–302 H-NS peaks
- Only 88 of the 506 LuxR and 302 H-NS peaks overlapped and were proximal to an ORF (≤500 bp), so alignment alone does not imply functional promoter occupancy
- The ΔluxR H-NS ChIP-seq interpretation depends on the LuxR/H-NS competition model; mapping quality alone cannot validate the biological condition

## Evidence

- [results] The ChIP-seq analysis (Trimmomatic trimming, bwa mapping to the V. harveyi ATCC BAA-1116 genome, macs2 peak calling on GSE136050 data): [evidence span withheld — anchor status: not_found]
- [supplementary] Align the trimmed reads to the V. harveyi ATCC BAA-1116 reference genome using bwa (bwa index, then bwa mem), and sort and index the resulting BAM files.: [evidence span withheld — anchor status: not_found]
- [discussion] Within the 506 and 302 peaks identified for LuxR and H-NS, respectively, 88 peaks overlapped and were proximal to an ORF (≤500 bp).: "Within the 506 and 302 peaks identified for LuxR and H-NS, respectively, 88 peaks overlapped and were proximal to an ORF (≤500 bp)."
