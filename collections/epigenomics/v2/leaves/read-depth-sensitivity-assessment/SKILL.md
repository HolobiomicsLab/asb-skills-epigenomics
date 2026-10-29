---
name: read-depth-sensitivity-assessment
description: Use when you need to determine the minimum sequencing depth required
  to visualize a ChIP-seq signal, especially to compare sharp histone marks like H3K4me3
  against broad domains like H3K36me3.
when_to_use_negative:
- Input is already a processed BedGraph/bigWig coverage track rather than mapped reads
  — depth re-normalization by SeqCode requires the total reads in the source BAM file
- The experiment uses spike-in material, where normalization should instead be driven
  by the manually supplied spike-in factor rather than sequencing depth
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Generates normalized genome-wide BedGraph profiles from down-sampled BAM files
    via buildChIPprofile using depth-based normalization
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Down-samples each mapped BAM file to the target read depths (1M and 8M reads)
- name: Bowtie
  role: Maps raw mESC ChIP-seq reads to the mouse mm9 genome
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:2b5f5c2bad4721f081ba1230b78676b9bfec99bfabe3547fb52b44a9bc41c4d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# read-depth-sensitivity-assessment

## Summary

Assesses how sequencing depth affects genome-wide ChIP-seq profile quality by down-sampling BAM files to controlled read counts and rebuilding profiles with SeqCode. This reveals the depth threshold at which sharp marks (H3K4me3) versus broad marks (H3K36me3) become visually resolvable.

## When to use

Use when you need to determine the minimum sequencing depth required to visualize a ChIP-seq signal, especially to compare sharp histone marks like H3K4me3 against broad domains like H3K36me3.

## When NOT to use

- Input is already a processed BedGraph/bigWig coverage track rather than mapped reads — depth re-normalization by SeqCode requires the total reads in the source BAM file
- The experiment uses spike-in material, where normalization should instead be driven by the manually supplied spike-in factor rather than sequencing depth

## Inputs

- Raw ChIP-seq reads from NCBI GEO (e.g. SRX367147 H3K4me3 mESC, GSM1019769 H3K36me3 mESC)
- BAM files of reads mapped to mm9 with unaligned reads excluded
- UCSC mm9 chromosome-size file (chromInfo.txt)

## Outputs

- Down-sampled BAM files at 1,000,000 and 8,000,000 reads per mark
- Compressed BedGraph genome-wide profiles from buildChIPprofile for each depth
- Depth-sensitivity comparison showing H3K4me3 resolution at 1M reads versus H3K36me3 resolution at 8M reads

## How to apply

Download raw mESC ChIP-seq reads (e.g. H3K4me3 SRX367147 from Tee et al.; H3K36me3 GSM1019769 from Ballare et al.) from NCBI GEO and map them to mouse mm9 with Bowtie, excluding unaligned reads from the resulting BAM files as SeqCode requires. Down-sample each BAM to two depths (1,000,000 and 8,000,000 reads) with SAMtools, then run SeqCode buildChIPprofile on each down-sampled BAM together with the mm9 chromInfo.txt file from UCSC, using default settings (100 bp window resolution, 150 bp average fragment size, single-end reads) to generate compressed BedGraph profiles. Rely on SeqCode's depth-based normalization (total number of reads in the BAM file) to make 1M and 8M profiles comparable within each histone mark. Validate by checking that 1 million reads suffice to resolve sharp H3K4me3 promoter patterning while 8 million reads are needed to visualize the broad H3K36me3 domains, matching Supplementary Figure 5.

## Related tools

- **SeqCode** (Generates normalized genome-wide BedGraph profiles from down-sampled BAM files via buildChIPprofile using depth-based normalization) — https://github.com/eblancoga/seqcode
- **SAMtools** (Down-samples each mapped BAM file to the target read depths (1M and 8M reads))
- **Bowtie** (Maps raw mESC ChIP-seq reads to the mouse mm9 genome)

## Examples

```
samtools view -s 0.1 -b mapped.bam > downsampled_1M.bam; bin/buildChIPprofile chromInfo.txt downsampled_1M.bam
```

## Evaluation signals

- Each down-sampled BAM contains exactly the target count (1,000,000 or 8,000,000 reads) after SAMtools down-sampling
- Unaligned reads are absent from the BAM files fed to buildChIPprofile
- Within each histone mark, 1M and 8M BedGraph profiles are comparable in scale due to SeqCode's depth-based normalization
- The 1M-read H3K4me3 profile resolves sharp promoter patterning, while only the 8M-read H3K36me3 profile distinguishes broad domains
- Generated profiles visually reproduce Supplementary Figure 5 of the SeqCode article

## Limitations

- Unaligned reads must be excluded from BAM files before SeqCode processing or commands may fail
- SeqCode does not yet support bigwig genome-wide profiles; output is compressed BedGraph format
- Spike-in normalization factors must be manually introduced by the user; sequencing depth is the default normalization basis

## Evidence

- [other] Down-sampling reveals differential saturation: 1 million reads suffice to visualize the sharp H3K4me3 pattern, while 8 million reads are needed to distinguish the broad H3K36me3 domains.: [evidence span withheld — anchor status: not_found]
- [methods] SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [methods] Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3).: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes) and the RefSeq gene transcript annotations.: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes)"
