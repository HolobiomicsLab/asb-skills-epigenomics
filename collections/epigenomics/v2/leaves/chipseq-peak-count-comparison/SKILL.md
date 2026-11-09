---
name: chipseq-peak-count-comparison
description: Use when raw ChIP-seq FASTQ files (e.g. GEO GSE136050) need trimming,
  alignment to the V. harveyi reference, macs2 peak calling, and validation of per-factor
  peak counts against published figures.
when_to_use_negative:
- Input is RNA-seq rather than ChIP-seq data — the same pipeline does not apply without
  a peak-calling step and control design
- Only processed peak coordinate files from GSE136191 are available and the goal is
  overlap/doublet analysis rather than reproducing raw-read peak calling — skip trimming/alignment
  and start from the deposited peak intervals
- The reference genome is not V. harveyi ATCC BAA-1116 — published count benchmarks
  (506/237/302) would not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Trimmomatic
  role: Trims adapter sequences and low-quality bases from ChIP-seq FASTQ files before
    alignment
- name: bwa
  role: Indexes the V. harveyi ATCC BAA-1116 genome (bwa index) and aligns trimmed
    reads (bwa mem) to produce BAM files
- name: macs2
  role: Calls transcription factor binding peaks per ChIP-seq sample, producing narrowPeak
    outputs counted and compared to published figures
- name: Perl
  role: Runs the authors' peak-annotation and overlap scripts (getPeakAnno.pl, findOverlaps.pl,
    findClosest2.pl) deposited on GitHub
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Genome browser used to inspect the deposited sequences and peak tracks
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:9a1e4a11a792116b1cfb73a6067a01df73aeca76f2163b123bf38047e6f87508
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
  - Perl
  - Trimmomatic
  - bwa
  - macs2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chipseq-peak-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces bacterial TF ChIP-seq processing (Trimmomatic → bwa → macs2) for LuxR and H-NS in V. harveyi ATCC BAA-1116 and benchmarks the resulting peak counts and overlap tallies against published values (506 LuxR peaks, 237 H-NS peaks in wild-type, 302 H-NS peaks in ΔluxR, 88 overlapping peaks proximal to an ORF ≤500 bp).

## When to use

Use when raw ChIP-seq FASTQ files (e.g. GEO GSE136050) need trimming, alignment to the V. harveyi reference, macs2 peak calling, and validation of per-factor peak counts against published figures.

## When NOT to use

- Input is RNA-seq rather than ChIP-seq data — the same pipeline does not apply without a peak-calling step and control design
- Only processed peak coordinate files from GSE136191 are available and the goal is overlap/doublet analysis rather than reproducing raw-read peak calling — skip trimming/alignment and start from the deposited peak intervals
- The reference genome is not V. harveyi ATCC BAA-1116 — published count benchmarks (506/237/302) would not apply

## Inputs

- ChIP-seq FASTQ files for LuxR and H-NS immunoprecipitation samples (NCBI GEO GSE136050)
- V. harveyi ATCC BAA-1116 reference genome FASTA for bwa indexing
- Processed LuxR and H-NS MACS2 peak interval files for wild-type and ΔluxR backgrounds (GSE136191)
- V. harveyi ORF annotation
- Authors' Perl scripts from github.com/Juliacvk/LuxR_H-NS (FileUtil.pm, getPeakAnno.pl, findOverlaps.pl, findClosest2.pl)

## Outputs

- Sorted, indexed BAM alignment files per sample
- macs2 peak calls in narrowPeak/BED format per factor and condition
- chipseq_peak_counts.csv with per-sample peak counts (506 LuxR, 237 H-NS WT, 302 H-NS ΔluxR)
- peak_overlap_counts.csv with per-strain LuxR/H-NS overlap tallies (68 wild-type, 88 ΔluxR proximal to ORF ≤500 bp)

## How to apply

Download the LuxR and H-NS ChIP-seq FASTQ files from NCBI GEO accession GSE136050 with the SRA toolkit, trim adapters and low-quality bases with Trimmomatic, then align to the V. harveyi ATCC BAA-1116 genome using bwa index plus bwa mem, sorting and indexing the resulting BAMs. Call TF binding peaks per sample with macs2 (narrowPeak output) and count peaks per factor/condition. The authors' Perl scripts in github.com/Juliacvk/LuxR_H-NS (getPeakAnno.pl, getGenePeakAnno2.pl, findOverlaps.pl, findClosest2.pl) annotate macs2 peaks and intersect LuxR and H-NS peak sets; keep only overlaps proximal to an ORF (≤500 bp) when tallying co-occupied loci. Validation is count-based: LuxR should yield 506 peaks and H-NS 302 peaks (237 in wild-type) in the reproduced analysis, with 88 overlapping peaks proximal to an ORF; each narrowPeak/BED file must exist and be non-empty.

## Related tools

- **Trimmomatic** (Trims adapter sequences and low-quality bases from ChIP-seq FASTQ files before alignment)
- **bwa** (Indexes the V. harveyi ATCC BAA-1116 genome (bwa index) and aligns trimmed reads (bwa mem) to produce BAM files)
- **macs2** (Calls transcription factor binding peaks per ChIP-seq sample, producing narrowPeak outputs counted and compared to published figures)
- **Perl** (Runs the authors' peak-annotation and overlap scripts (getPeakAnno.pl, findOverlaps.pl, findClosest2.pl) deposited on GitHub) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Genome browser used to inspect the deposited sequences and peak tracks)

## Examples

```
macs2 callpeak -t HNS_IP.trimmed.bam -c input.bam -f BAM -n HNS_WT --format BAM --outdir peaks/
```

## Evaluation signals

- Reproduced peak counts match published values: 506 LuxR peaks, 237 H-NS peaks in wild-type, and 302 H-NS peaks in the ΔluxR background
- LuxR/H-NS overlap tally reproduces 68 overlapping regions in wild-type and 88 co-occupied loci (ΔluxR) filtered to peaks proximal to an ORF (≤500 bp)
- Every narrowPeak/BED peak file exists and is non-empty
- Per-sample counts recorded in chipseq_peak_counts.csv and peak_overlap_counts.csv are consistent with the narrowPeak files they summarize
- ChIP-seq signal is corroborated by qPCR validation as reported in the paper ('ChIP-seq was validated by qPCR')

## Limitations

- The H-NS count context is strain-dependent: H-NS peaks rise from 237 (wild-type) to 302 (ΔluxR), so counts cannot be compared without matching the correct background
- Overlap counts depend on the ≤500 bp ORF-proximity filter; unfiltered overlap counts will not match the published 88
- Whether LuxR also functions alongside H-NS to repress gene expression is unresolved and needs additional experimentation, so count comparisons address co-occupancy, not co-repression
- Peak-calling parameters other than the authors' macs2 settings may shift counts away from the published benchmarks

## Evidence

- [discussion] Within the 506 and 302 peaks identified for LuxR and H-NS, respectively, 88 peaks overlapped and were proximal to an ORF (≤500 bp).: "Within the 506 and 302 peaks identified for LuxR and H-NS, respectively, 88 peaks overlapped and were proximal to an ORF (≤500 bp)."
- [supplementary] Datasets for RNA-seq and ChIP-seq have been deposited into NCBI GEO: GSE136050 and GSE136191: "Datasets for RNA-seq and ChIP-seq have been deposited into NCBI GEO: GSE136050 and GSE136191"
- [supplementary] All Perl scripts have been deposited in Github: https://github.com/Juliacvk/LuxR_H-NS: "All Perl scripts have been deposited in Github: https://github.com/Juliacvk/LuxR_H-NS"
- [readme] findOverlaps.pl - perl script to identify sets of overlapping peaks for Venn diagram: "findOverlaps.pl - perl script to identify sets of overlapping peaks for Venn diagram"
- [readme] getPeakAnno.pl - perl script to add annotation information to Macs2 peaks: "getPeakAnno.pl - perl script to add annotation information to Macs2 peaks"
