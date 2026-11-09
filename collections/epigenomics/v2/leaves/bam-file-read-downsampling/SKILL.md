---
name: bam-file-read-downsampling
description: Use when you need to determine the minimum sequencing depth at which
  a ChIP-seq mark's genome-wide profile becomes visually distinguishable, by comparing
  BedGraph profiles generated from BAM files sub-sampled to controlled read counts.
when_to_use_negative:
- The goal is absolute cross-sample quantification rather than within-mark depth comparison
  — down-sampling discards data and SeqCode's depth-based normalization already makes
  profiles comparable
- The experiment uses spike-in material — SeqCode instead normalizes by reads mapped
  over the spike-in genome, and the spike-in factor must be manually introduced by
  users
- Input is raw FASTQ rather than mapped BAM — read mapping (e.g. Bowtie) with exclusion
  of unaligned reads must precede down-sampling
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SAMtools
  role: Used to down-sample each BAM file to the two target read depths (1M and 8M
    reads)
- name: Bowtie
  role: Maps raw reads over the mouse mm9 genome before down-sampling; unaligned reads
    must be excluded from the resulting BAM files
- name: SeqCode
  role: buildChIPprofile converts each down-sampled BAM (plus mm9 chromInfo) into
    a compressed BedGraph profile; its depth-based normalization makes 1M and 8M profiles
    comparable
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:c516e399f4d5e25f7348c53a77f02763dd208731717fc906914a9a529e3c4bf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - SAMtools
  - SeqCode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# bam-file-read-downsampling

## Summary

Down-sample mapped BAM files to defined read depths (e.g. 1,000,000 and 8,000,000 reads) with SAMtools to test sequencing-depth saturation of ChIP-seq signals before profile generation with SeqCode. It underpins the paper's demonstration that sharp H3K4me3 promoter patterning is resolved with 1M reads while broad H3K36me3 domains require 8M reads.

## When to use

Use when you need to determine the minimum sequencing depth at which a ChIP-seq mark's genome-wide profile becomes visually distinguishable, by comparing BedGraph profiles generated from BAM files sub-sampled to controlled read counts.

## When NOT to use

- The goal is absolute cross-sample quantification rather than within-mark depth comparison — down-sampling discards data and SeqCode's depth-based normalization already makes profiles comparable
- The experiment uses spike-in material — SeqCode instead normalizes by reads mapped over the spike-in genome, and the spike-in factor must be manually introduced by users
- Input is raw FASTQ rather than mapped BAM — read mapping (e.g. Bowtie) with exclusion of unaligned reads must precede down-sampling

## Inputs

- Mapped ChIP-seq BAM files (unaligned reads excluded), e.g. mESC H3K4me3 and H3K36me3 mapped to mm9
- Chromosome-size file for the target assembly (e.g. mm9 chromInfo.txt from UCSC genome browser downloads)
- Target down-sampling depths (e.g. 1,000,000 and 8,000,000 reads) or sampling fractions for SAMtools

## Outputs

- Down-sampled BAM files at each target read depth per histone mark
- Compressed BedGraph genome-wide profiles produced by SeqCode buildChIPprofile for each depth
- Depth-saturation comparison showing how many reads are needed to resolve sharp (H3K4me3) vs broad (H3K36me3) profiles

## How to apply

First map raw reads (e.g. mESC H3K4me3 from SRX367147, Tee et al., and H3K36me3 from GSM1019769, Ballare et al.) to the reference genome (mouse mm9) with Bowtie, and exclude unaligned reads from the resulting BAM files as required by SeqCode commands. Down-sample each BAM to two depths (1,000,000 and 8,000,000 reads) using SAMtools, choosing the sampling fraction from the total mapped-read count. Run SeqCode buildChIPprofile on each down-sampled BAM together with the mm9 chromosome-size file (chromInfo.txt from UCSC), with default settings: 100 bp window resolution, 150 bp average fragment size, single-end reads. SeqCode normalizes values by sequencing depth (total reads in the BAM), so the 1M and 8M profiles are directly comparable within each histone mark; the depth at which the pattern first resolves defines the saturation threshold for that mark.

## Related tools

- **SAMtools** (Used to down-sample each BAM file to the two target read depths (1M and 8M reads))
- **Bowtie** (Maps raw reads over the mouse mm9 genome before down-sampling; unaligned reads must be excluded from the resulting BAM files)
- **SeqCode** (buildChIPprofile converts each down-sampled BAM (plus mm9 chromInfo) into a compressed BedGraph profile; its depth-based normalization makes 1M and 8M profiles comparable) — https://github.com/eblancoga/seqcode

## Examples

```
samtools view -s 0.125 -b H3K36me3_mm9.bam > H3K36me3_1M.bam && bin/buildChIPprofile chromInfo.txt H3K36me3_1M.bam H3K36me3_1M
```

## Evaluation signals

- The 1M-read H3K4me3 BedGraph profile already resolves the sharp promoter pattern, matching Supplementary Figure 5
- The 1M-read H3K36me3 profile fails to distinguish the broad domains, while the 8M-read profile does — reproducing the differential saturation finding
- Within each histone mark, 1M and 8M profiles are comparable in scale because SeqCode normalizes by total reads in the BAM
- Each down-sampled BAM contains the intended number of reads (1,000,000 or 8,000,000), and unaligned reads are excluded before SeqCode processing
- All four BedGraph profiles were generated with identical buildChIPprofile settings (100 bp windows, 150 bp average fragment size, single-end reads)

## Limitations

- Unaligned reads must be excluded from BAM files before SeqCode commands, otherwise processing fails or misbehaves
- Findings are depth-saturation statements for these marks (sharp H3K4me3 vs broad H3K36me3 in mESC on mm9); thresholds do not necessarily transfer to other marks, cell types, or assemblies
- SeqCode normalization relies on the total number of reads in the BAM file, so any untracked change in BAM content between depths distorts comparability

## Evidence

- [other] Down-sample each BAM file to two depths (1,000,000 and 8,000,000 reads) using SAMtools.: [evidence span withheld — anchor status: not_found]
- [methods] SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] Commands working with sequencing data require the user to provide the mapped reads of the experiment (BAM or SAM format).: "Commands working with sequencing data require the user to provide the mapped reads of the experiment (BAM or SAM format)."
- [methods] Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3).: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
