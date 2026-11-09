---
name: chip-seq-read-quality-trimming
description: Use when raw Illumina ChIP-seq FASTQ files (e.g., TF immunoprecipitation
  samples from GEO accession GSE136050) must be adapter- and quality-trimmed prior
  to genome alignment and macs2 peak calling.
when_to_use_negative:
- Input is already trimmed or provided as aligned BAM files — skip trimming and go
  directly to bwa mem mapping and macs2 peak calling.
- The goal is RNA-seq differential expression (GSE136191 in this study) rather than
  TF binding peak detection; the trimming rationale here targets ChIP-seq peak calling.
- Peak sets are already called and the task is only overlap/annotation analysis (e.g.,
  findOverlaps.pl, getPeakAnno.pl on existing macs2 output).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Trimmomatic
  role: Trims adapter sequences and low-quality bases from each ChIP-seq FASTQ file
    before genome mapping
- name: bwa
  role: Indexes the V. harveyi ATCC BAA-1116 genome and maps trimmed reads with bwa
    mem; produces BAMs for peak calling
- name: macs2
  role: Calls transcription factor binding peaks on each aligned ChIP-seq sample after
    trimming/mapping
- name: SRAs toolkit
  role: Downloads the GSE136050 ChIP-seq FASTQ files for LuxR and H-NS immunoprecipitation
    samples
- name: Perl scripts (Juliacvk/LuxR_H-NS)
  role: Post-trimming annotation of macs2 peaks (getPeakAnno.pl, getGenePeakAnno2.pl)
    and overlap detection (findOverlaps.pl)
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:85290580db1ddf95a81da9ac4599b036ae56502a92d1178248653af5a11b19e7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  repo_url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - JBrowse
  - Perl scripts (Juliacvk/LuxR_H-NS)
  - SRAs toolkit
  - Trimmomatic
  - bwa
  - macs2
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# chip-seq-read-quality-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trims adapters and low-quality bases from ChIP-seq FASTQ reads with Trimmomatic before aligning to the V. harveyi ATCC BAA-1116 genome (bwa) and calling TF binding peaks with macs2, the pipeline used to quantify LuxR and H-NS occupancy in GSE136050.

## When to use

Use when raw Illumina ChIP-seq FASTQ files (e.g., TF immunoprecipitation samples from GEO accession GSE136050) must be adapter- and quality-trimmed prior to genome alignment and macs2 peak calling.

## When NOT to use

- Input is already trimmed or provided as aligned BAM files — skip trimming and go directly to bwa mem mapping and macs2 peak calling.
- The goal is RNA-seq differential expression (GSE136191 in this study) rather than TF binding peak detection; the trimming rationale here targets ChIP-seq peak calling.
- Peak sets are already called and the task is only overlap/annotation analysis (e.g., findOverlaps.pl, getPeakAnno.pl on existing macs2 output).

## Inputs

- ChIP-seq FASTQ files for LuxR and H-NS immunoprecipitation samples from GEO accession GSE136050
- V. harveyi ATCC BAA-1116 reference genome FASTA (for the downstream bwa index)
- Illumina adapter sequences (Trimmomatic ILLUMINACLIP fasta)

## Outputs

- Adapter- and quality-trimmed FASTQ files per sample
- Sorted and indexed BAM alignments against V. harveyi ATCC BAA-1116
- macs2 peak calls in narrowPeak/BED format per factor and condition
- chipseq_peak_counts.csv with per-sample peak counts

## How to apply

Download the LuxR and H-NS ChIP-seq immunoprecipitation FASTQ files from NCBI GEO accession GSE136050 with the SRA toolkit, then run Trimmomatic on each file to remove adapter sequences and low-quality bases. The trimmed reads are the required input for the next stages: bwa index of the V. harveyi ATCC BAA-1116 reference followed by bwa mem alignment, then sorting/indexing of the BAMs and macs2 peak calling per sample and condition. Decision point: proceed to peak calling only after confirming each trimmed FASTQ is non-empty and the alignment produces sorted, indexed BAMs. Rationale: clean reads are necessary to recover the published binding landscape (506 LuxR peaks, 237 H-NS peaks in wild-type rising to 302 in the ΔluxR background) rather than adapter/quality artifacts. Peak calls can then be annotated with the repo's Perl scripts (getPeakAnno.pl, getGenePeakAnno2.pl) and cross-checked against qPCR validation performed by the authors.

## Related tools

- **Trimmomatic** (Trims adapter sequences and low-quality bases from each ChIP-seq FASTQ file before genome mapping)
- **bwa** (Indexes the V. harveyi ATCC BAA-1116 genome and maps trimmed reads with bwa mem; produces BAMs for peak calling)
- **macs2** (Calls transcription factor binding peaks on each aligned ChIP-seq sample after trimming/mapping)
- **SRAs toolkit** (Downloads the GSE136050 ChIP-seq FASTQ files for LuxR and H-NS immunoprecipitation samples)
- **Perl scripts (Juliacvk/LuxR_H-NS)** (Post-trimming annotation of macs2 peaks (getPeakAnno.pl, getGenePeakAnno2.pl) and overlap detection (findOverlaps.pl)) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
trimmomatic PE -phred33 luxR_IP_R1.fq.gz luxR_IP_R2.fq.gz trimmed_R1.fq.gz unpaired_R1.fq.gz trimmed_R2.fq.gz unpaired_R2.fq.gz ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

## Evaluation signals

- Reproduced peak counts match the reported values: 506 LuxR peaks, 237 H-NS peaks in wild-type, and 302 H-NS peaks in the ΔluxR background.
- Each peak coordinate file (narrowPeak/BED) exists and is non-empty for every sample.
- chipseq_peak_counts.csv contains a per-sample row for every trimmed, aligned ChIP-seq sample.
- BAM files are sorted and indexed after bwa mem alignment before macs2 is run.
- Within the 506 and 302 LuxR/H-NS peaks, 88 overlapping peaks are proximal to an ORF (≤500 bp), matching the published overlap count.

## Limitations

- Specific Trimmomatic parameter values (adapter fasta, quality cutoffs, MINLEN) are not stated in the article; only the trimming step itself is specified.
- The 88 overlapping LuxR/H-NS peaks were restricted to peaks proximal to an ORF (≤500 bp), so overlap counts are sensitive to this proximity threshold.
- ChIP-seq results were validated by qPCR rather than replicated peak calling; discrepancies in reproduced counts cannot be resolved from the published parameters alone.
- The manuscript carries no changelog, so versioning of scripts or parameters cannot be traced from the document.

## Evidence

- [other] Trim adapter sequences and low-quality bases from each FASTQ file with Trimmomatic.: [evidence span withheld — anchor status: not_found]
- [other] Align the trimmed reads to the V. harveyi ATCC BAA-1116 reference genome using bwa: [evidence span withheld — anchor status: not_found]
- [supplementary] ChIP-seq was validated by qPCR.: "ChIP-seq was validated by qPCR."
- [supplementary] All Perl scripts have been deposited in Github: https://github.com/Juliacvk/LuxR_H-NS: "All Perl scripts have been deposited in Github: https://github.com/Juliacvk/LuxR_H-NS"
- [readme] getPeakAnno.pl - perl script to add annotation information to Macs2 peaks: "getPeakAnno.pl - perl script to add annotation information to Macs2 peaks"
