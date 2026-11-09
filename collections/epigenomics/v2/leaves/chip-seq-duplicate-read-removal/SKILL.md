---
name: chip-seq-duplicate-read-removal
description: Use when aligned ChIP-seq BAM files are about to be binned and counted
  (e.g. with histoneHMM getBins/countBamInGRanges) and PCR/sequencing duplicates would
  inflate per-bin read counts; the article applied this to all H3K27me3/H3K9me3 datasets
  from rat, mouse and human cell lines.
when_to_use_negative:
- Input is not aligned ChIP-seq data but an already-binned or already-counted feature
  table — duplicates were removed upstream or the step is moot
- Peak-like (narrow) histone marks or transcription-factor ChIP-seq where methods
  designed for peaks are intended, since histoneHMM targets broad diffuse marks like
  H3K27me3 and H3K9me3
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: samtools
  role: Removes duplicated reads from each aligned ChIP-seq BAM file prior to binning
    and HMM fitting
- name: histoneHMM
  role: Downstream R package that consumes the duplicate-free BAMs via getBins/countBamInGRanges
    to build binned read counts for the bivariate HMM
  repo: github.com/matthiasheinig/histoneHMM
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:3f0992817220d63db0ca9f30e5f479ffbb78bdba55022625de85ab751671dede
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - BroadPeak
  - C++
  - MACS2
  - R
  - Rseg
  - SICER
  - Zinba
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# chip-seq-duplicate-read-removal

## Summary

Preprocessing step from the histoneHMM pipeline (Heinig et al., BMC Bioinformatics 2015) in which duplicated reads are removed from aligned ChIP-seq BAM files with samtools before genome-wide binning and HMM-based calling of modified or differentially modified regions.

## When to use

Use when aligned ChIP-seq BAM files are about to be binned and counted (e.g. with histoneHMM getBins/countBamInGRanges) and PCR/sequencing duplicates would inflate per-bin read counts; the article applied this to all H3K27me3/H3K9me3 datasets from rat, mouse and human cell lines.

## When NOT to use

- Input is not aligned ChIP-seq data but an already-binned or already-counted feature table — duplicates were removed upstream or the step is moot
- Peak-like (narrow) histone marks or transcription-factor ChIP-seq where methods designed for peaks are intended, since histoneHMM targets broad diffuse marks like H3K27me3 and H3K9me3

## Inputs

- Aligned ChIP-seq BAM file per sample (with index), e.g. hg19-aligned ENCODE samples for H1-hESC and K562
- Sample list / pairing of experimental and control ChIP-seq samples

## Outputs

- Duplicate-removed BAM file per sample, suitable for counting into 1000 bp genomic bins
- Per-bin bivariate read-count matrix (after downstream getBins/countBamInGRanges and 0.1-percentile truncation)

## How to apply

After aligning ChIP-seq reads (e.g. to hg19 for the ENCODE H1-hESC vs K562 comparison), run samtools on each sample BAM to remove duplicated reads, keeping only non-redundant fragments per position. This happens before partitioning the genome into 1000 bp bins and aggregating read counts with histoneHMM's getBins/countBamInGRanges. Duplicate removal works together with a second guard: read counts above the upper 0.1 percentile are truncated to that value so that extreme counts cannot cause numerical problems with very small emission probabilities in the bivariate HMM. Perform the step identically on both samples in a comparison so the bivariate read counts are symmetrically processed; use default samtools behavior as the article used the tool as-is without reported custom thresholds.

## Related tools

- **samtools** (Removes duplicated reads from each aligned ChIP-seq BAM file prior to binning and HMM fitting)
- **histoneHMM** (Downstream R package that consumes the duplicate-free BAMs via getBins/countBamInGRanges to build binned read counts for the bivariate HMM) — github.com/matthiasheinig/histoneHMM

## Examples

```
samtools rmdup sample.bam sample.rmdup.bam
```

## Evaluation signals

- Each input BAM has passed through samtools duplicate removal before any read counting; the read-count matrix is built from the deduplicated files only
- Residual per-bin count distributions show no extreme outliers, consistent with additionally truncating counts at the upper 0.1 percentile (maxq)
- Downstream histoneHMM posterior/viterbi calls succeed without numerical problems from very small emission probabilities, the failure mode duplicate removal plus truncation prevents
- Processing is applied symmetrically to both samples of a comparison (e.g. H1-hESC and K562) so bivariate read counts are comparable

## Limitations

- The article reports no samtools version number and used default behavior, so exact duplicate-detection parameters are not reproducible from the text alone
- Duplicate removal is documented only as part of the histoneHMM ChIP-seq preprocessing; its adequacy for other data types or library construction protocols is not assessed
- Truncation at the upper 0.1 percentile and duplicate removal together shape count distributions; skipping one changes the HMM input and downstream differentially modified region calls

## Evidence

- [discussion] Duplicate removal precedes binning and HMM fitting: "and removed duplicated reads using samtools"
- [discussion] Rationale for the accompanying count truncation guard: "we truncated read counts at the upper 0.1 percentile"
- [methods] Binning step that consumes the deduplicated BAMs: "countBamInGRanges(bam.file, granges, min.mapq, read.width)"
- [readme] Downstream context: broad-footprint histone mark analysis: "histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3."
