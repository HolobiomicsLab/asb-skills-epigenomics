---
name: chip-seq-genome-browser-profile-generation
description: Use when you have BAM/SAM files of mapped ChIP-seq reads and need genome-wide
  coverage profiles (BedGraph) for genome-browser visualization or for testing sequencing-depth
  saturation of a histone mark.
when_to_use_negative:
- You need bigwig-format genome-wide profiles — SeqCode does not support bigwig output;
  it is deferred to future versions
- You need discrete peak coordinates rather than continuous coverage profiles — peak
  calling (e.g. MACS) followed by processmacs cleaning is the appropriate route
- The reads are not yet aligned to a reference genome — buildChIPprofile consumes
  mapped BAM/SAM input, not raw FASTQ
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SeqCode
  role: buildChIPprofile generates the compressed BedGraph genome-wide profiles from
    BAM input, with per-bin values normalized by sequencing depth (100 bp windows,
    150 bp average fragment size, single-end)
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: maps raw ChIP-seq reads to the reference genome (e.g. mouse mm9) prior to
    profile construction
- name: SAMtools
  role: down-samples BAM files to specified read depths (e.g. 1M and 8M reads) to
    test depth saturation of profiles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:a5cbe60f86127344d7bf903809af95ab2e0fb976f210397f1174877d8158b559
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# chip-seq-genome-browser-profile-generation

## Summary

Generate compressed, genome-browser-ready ChIP-seq profiles (BedGraph) from mapped BAM reads using SeqCode's buildChIPprofile, which normalizes per-bin coverage by sequencing depth so profiles from different depths (e.g. 1M vs 8M down-sampled reads) remain comparable.

## When to use

Use when you have BAM/SAM files of mapped ChIP-seq reads and need genome-wide coverage profiles (BedGraph) for genome-browser visualization or for testing sequencing-depth saturation of a histone mark.

## When NOT to use

- You need bigwig-format genome-wide profiles — SeqCode does not support bigwig output; it is deferred to future versions
- You need discrete peak coordinates rather than continuous coverage profiles — peak calling (e.g. MACS) followed by processmacs cleaning is the appropriate route
- The reads are not yet aligned to a reference genome — buildChIPprofile consumes mapped BAM/SAM input, not raw FASTQ

## Inputs

- BAM/SAM files of ChIP-seq reads mapped with Bowtie (unaligned reads removed)
- Chromosome size file (chromInfo.txt) from UCSC for the target assembly (e.g. mm9)
- Optionally down-sampled BAMs (e.g. 1M and 8M reads via SAMtools)

## Outputs

- Compressed BedGraph genome-wide ChIP-seq profiles suitable for genome-browser visualization

## How to apply

Map raw reads with Bowtie to the target assembly (e.g. mouse mm9), and exclude unaligned reads from the resulting BAM files since SeqCode commands require aligned reads only. Retrieve the chromosome-size file (chromInfo.txt) from the UCSC genome browser downloads for the assembly, as SeqCode programs require the list of chromosomes and their sizes. Down-sample BAMs to chosen depths with SAMtools (e.g. 1,000,000 and 8,000,000 reads) to probe saturation: sharp marks like H3K4me3 resolve at 1M reads whereas broad H3K36me3 domains require 8M. Run buildChIPprofile with default settings (100 bp window resolution, 150 bp average fragment size, single-end reads) on each BAM; its depth-based normalization (total reads in the BAM) makes profiles comparable across depths. Validate the BedGraph outputs by confirming the expected signal pattern — sharp promoter peaks for H3K4me3 at 1M reads, broad continuous domains for H3K36me3 only at 8M reads.

## Related tools

- **SeqCode** (buildChIPprofile generates the compressed BedGraph genome-wide profiles from BAM input, with per-bin values normalized by sequencing depth (100 bp windows, 150 bp average fragment size, single-end)) — https://github.com/eblancoga/seqcode
- **Bowtie** (maps raw ChIP-seq reads to the reference genome (e.g. mouse mm9) prior to profile construction)
- **SAMtools** (down-samples BAM files to specified read depths (e.g. 1M and 8M reads) to test depth saturation of profiles)

## Examples

```
bin/buildChIPprofile chromInfo.txt H3K4me3_1M_reads.bam
```

## Evaluation signals

- The output is a compressed BedGraph profile generated per input BAM, as produced by buildChIPprofile
- Profiles from different down-sampled depths (1M vs 8M) show comparable magnitude within a histone mark, confirming depth-based normalization (total reads in the BAM)
- Sharp H3K4me3 promoter patterning is clearly resolved already at 1 million reads
- Broad H3K36me3 domains are distinguishable only at 8 million reads; at 1M they should be unresolved
- No unaligned reads remain in the input BAMs processed by SeqCode commands

## Limitations

- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- Unaligned reads must be excluded from the resulting BAM files before any SeqCode command will work on them
- Depth normalization assumes uniform sequencing depth scaling; spike-in correction must be manually introduced by users when needed

## Evidence

- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [methods] Bowtie mapping to genome assemblies: "Bowtie7 was used to map the reads over the genome of mouse (mm9 and mm10), human (hg38), and the fruit fly (dm3)."
- [methods] Depth-based normalization rationale: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [readme] Chromosome size file requirement from README: "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes)"
- [readme] Profile generation purpose from README: "Generate profiles of sequencing data for genome browser visualizations"
