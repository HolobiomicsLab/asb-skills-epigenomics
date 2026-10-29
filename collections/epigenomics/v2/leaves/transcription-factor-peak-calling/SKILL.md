---
name: transcription-factor-peak-calling
description: Use when ChIP-seq FASTQ data for a DNA-binding factor (e.g., LuxR or
  H-NS in V. harveyi) needs trimming, reference alignment, and macs2 peak calling
  to count and compare binding peaks across conditions.
when_to_use_negative:
- Input is RNA-seq differential expression data rather than ChIP-seq immunoprecipitation
  reads (use the RNA-seq analysis path; the paper deposited RNA-seq separately under
  GSE136191)
- The question is promoter co-occupancy or overlap analysis of already-called peak
  sets rather than peak calling itself — use findOverlaps.pl / findClosest2.pl on
  existing macs2 peak files instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: Trimmomatic
  role: Trims adapter sequences and low-quality bases from ChIP-seq FASTQ files before
    alignment
- name: bwa
  role: Indexes the V. harveyi ATCC BAA-1116 genome (bwa index) and aligns trimmed
    reads (bwa mem) to produce BAM files
- name: macs2
  role: Calls transcription factor binding peaks (narrowPeak output) for each ChIP-seq
    sample and condition
- name: getPeakAnno.pl
  role: Perl script from the paper's repository that adds annotation information to
    macs2 peaks, enabling peak-to-ORF (≤500 bp) assignment
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Genome browser session used to inspect the uploaded sequences and peak tracks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:c773ac1e0a3679d5e6f3ec6eec71a48c83c3fb90f6ba6a0cc118643fb4279d68
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
  - Trimmomatic
  - bwa
  - getPeakAnno.pl
  - macs2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# transcription-factor-peak-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end ChIP-seq transcription factor binding analysis: trim ChIP-seq FASTQ reads, align them to the V. harveyi ATCC BAA-1116 reference genome with bwa, and call transcription factor binding peaks with macs2 to quantify LuxR and H-NS occupancy under wild-type and mutant conditions.

## When to use

Use when ChIP-seq FASTQ data for a DNA-binding factor (e.g., LuxR or H-NS in V. harveyi) needs trimming, reference alignment, and macs2 peak calling to count and compare binding peaks across conditions.

## When NOT to use

- Input is RNA-seq differential expression data rather than ChIP-seq immunoprecipitation reads (use the RNA-seq analysis path; the paper deposited RNA-seq separately under GSE136191)
- The question is promoter co-occupancy or overlap analysis of already-called peak sets rather than peak calling itself — use findOverlaps.pl / findClosest2.pl on existing macs2 peak files instead

## Inputs

- ChIP-seq FASTQ files from NCBI GEO accession GSE136050 (LuxR and H-NS immunoprecipitation samples)
- V. harveyi ATCC BAA-1116 reference genome (multi-fasta, cleaned/reformatted with reformatFasta.pl)
- Indexed reference genome (bwa index output)

## Outputs

- Sorted, indexed BAM alignment files per sample
- macs2 narrowPeak/BED peak coordinate files per factor and condition
- Per-sample peak count table (chipseq_peak_counts.csv): 506 LuxR peaks, 237 H-NS wild-type peaks, 302 H-NS ΔluxR peaks
- Annotated peak-to-gene tables from getPeakAnno.pl / getGenePeakAnno2.pl (peaks proximal to an ORF ≤500 bp)

## How to apply

Download ChIP-seq FASTQ files for the immunoprecipitation samples from NCBI GEO accession GSE136050 using the SRA toolkit. Trim adapters and low-quality bases from each FASTQ with Trimmomatic, then align trimmed reads to the V. harveyi ATCC BAA-1116 reference genome using bwa index followed by bwa mem, sorting and indexing the resulting BAM files. Call transcription factor binding peaks for each sample with macs2 (narrowPeak output). Count peaks per factor and condition and compare against published values: 506 LuxR peaks and 237 H-NS peaks in wild-type, rising to 302 H-NS peaks in a ΔluxR background. For biological interpretation, annotate peaks with the repository's Perl utilities (getPeakAnno.pl, findOverlaps.pl) — in the paper, 88 peaks overlapped between LuxR and H-NS and were proximal to an ORF (≤500 bp).

## Related tools

- **Trimmomatic** (Trims adapter sequences and low-quality bases from ChIP-seq FASTQ files before alignment)
- **bwa** (Indexes the V. harveyi ATCC BAA-1116 genome (bwa index) and aligns trimmed reads (bwa mem) to produce BAM files)
- **macs2** (Calls transcription factor binding peaks (narrowPeak output) for each ChIP-seq sample and condition)
- **getPeakAnno.pl** (Perl script from the paper's repository that adds annotation information to macs2 peaks, enabling peak-to-ORF (≤500 bp) assignment) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Genome browser session used to inspect the uploaded sequences and peak tracks)

## Examples

```
macs2 callpeak -t H-NS_WT_IP.bam -c INPUT_WT.bam -f BAM -n H-NS_WT
```

## Evaluation signals

- Reproduced peak counts match the reported values: 506 LuxR peaks, 237 H-NS peaks in wild-type, and 302 H-NS peaks in the ΔluxR background
- Every peak file (narrowPeak/BED) exists and is non-empty for each sample and condition
- Overlap analysis reproduces the reported 88 overlapping LuxR/H-NS peaks proximal to an ORF (≤500 bp)
- H-NS peak count correctly increases in the ΔluxR mutant background (302 vs 237 in wild-type), reproducing the direction of the published comparison
- ChIP-seq results are consistent with the paper's orthogonal validation: ChIP-seq was validated by qPCR

## Limitations

- Peak counts are condition- and strain-specific: H-NS binding increases from 237 to 302 peaks when ChIP-seq is performed in a ΔluxR background, so counts cannot be transferred between conditions
- Only 88 of the 506 LuxR and 302 H-NS peaks overlap and are proximal to an ORF (≤500 bp); many peaks are intergenic or non-overlapping and require separate annotation care
- Peak annotation and overlap logic depends on the repository's Perl helper scripts (getPeakAnno.pl, findOverlaps.pl, findClosest2.pl) and a cleanly reformatted multi-fasta reference (reformatFasta.pl)

## Evidence
