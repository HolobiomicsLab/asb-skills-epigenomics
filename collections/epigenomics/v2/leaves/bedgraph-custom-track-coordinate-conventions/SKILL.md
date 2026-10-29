---
name: bedgraph-custom-track-coordinate-conventions
description: Use when you must convert mapped ChIP-seq reads (BAM/SAM) into genome-wide
  signal profiles (BedGraph) for browser visualization, especially when comparing
  profiles across sequencing depths or histone marks (e.g. sharp H3K4me3 vs broad
  H3K36me3).
when_to_use_negative:
- The profiles are needed in bigWig format — SeqCode defers bigWig generation to future
  versions
- The input is already a processed BedGraph/peak file rather than mapped reads — use
  tools like processmacs or combineChIPprofiles instead of buildChIPprofile
- Spike-in-normalized signal is required without manual work — the spike-in normalization
  factor must be introduced manually by users
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0092
tools:
- name: SeqCode
  role: buildChIPprofile converts BAM reads + chromosome-size file into depth-normalized
    compressed BedGraph profiles
  repo: https://github.com/eblancoga/seqcode
- name: SAMtools
  role: Down-samples BAM files to target depths (e.g. 1M and 8M reads) and excludes
    unaligned reads before SeqCode processing
- name: Bowtie
  role: Maps raw ChIP-seq reads to the reference genome (e.g. mouse mm9) to produce
    the input BAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:e9436a6935dcb7b0a5efdbfee3782a882c0f1e08a63c11a9e208fc6a4a7666be
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
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bedgraph-custom-track-coordinate-conventions

## Summary

Generate compressed BedGraph genome-browser custom tracks of ChIP-seq signal with SeqCode buildChIPprofile, following SeqCode's conventions: a chromosome-size file defines coordinates/bins, BAM depth drives normalization, and unaligned reads must first be removed so profiles load correctly in genome browsers.

## When to use

Use when you must convert mapped ChIP-seq reads (BAM/SAM) into genome-wide signal profiles (BedGraph) for browser visualization, especially when comparing profiles across sequencing depths or histone marks (e.g. sharp H3K4me3 vs broad H3K36me3).

## When NOT to use

- The profiles are needed in bigWig format — SeqCode defers bigWig generation to future versions
- The input is already a processed BedGraph/peak file rather than mapped reads — use tools like processmacs or combineChIPprofiles instead of buildChIPprofile
- Spike-in-normalized signal is required without manual work — the spike-in normalization factor must be introduced manually by users

## Inputs

- Mapped reads in BAM (or SAM) format with unaligned reads excluded (e.g. Bowtie mappings to mm9)
- Chromosome size file for the genome assembly (e.g. mm9 chromInfo.txt from UCSC downloads)
- Optional down-sampled BAMs at fixed read depths (e.g. 1,000,000 and 8,000,000 reads via SAMtools)

## Outputs

- Compressed BedGraph genome-wide profile per input BAM, usable as a genome-browser custom track

## How to apply

Download the chromosome-size list (e.g. mm9 chromInfo.txt from UCSC) and the mapped reads in BAM format, then run SeqCode buildChIPprofile with the chromInfo file and the BAM as inputs; unaligned reads must be excluded from the BAM beforehand because SeqCode commands will not process them correctly. buildChIPprofile uses the default parameters of 100 bp window (bin) resolution, 150 bp average fragment size, and single-end reads, and normalizes values in each bin by sequencing depth (total reads in the BAM), so down-sampled 1M- and 8M-read BAMs produce directly comparable profiles within a mark. Output is a compressed BedGraph suitable as a genome browser custom track. Correctness is judged by whether the profiles reproduce the expected biology: 1 million reads resolve sharp H3K4me3 promoter patterning while 8 million are required for broad H3K36me3 domains.

## Related tools

- **SeqCode** (buildChIPprofile converts BAM reads + chromosome-size file into depth-normalized compressed BedGraph profiles) — https://github.com/eblancoga/seqcode
- **SAMtools** (Down-samples BAM files to target depths (e.g. 1M and 8M reads) and excludes unaligned reads before SeqCode processing)
- **Bowtie** (Maps raw ChIP-seq reads to the reference genome (e.g. mouse mm9) to produce the input BAM)

## Examples

```
bin/buildChIPprofile tests/inputs/ChromInfo.txt H3K4me3_1M.bam
```

## Evaluation signals

- The output is a compressed BedGraph (not bigWig) and loads as a custom track in a genome browser using the same assembly (e.g. mm9) as the chromInfo file
- Within a histone mark, 1M and 8M down-sampled profiles are comparable in scale because normalization is by total reads in each BAM
- The 1M-read profile reproduces sharp H3K4me3 promoter patterning; only the 8M-read profile resolves broad H3K36me3 domains (as in Supplementary Figure 5)
- BAM inputs contain no unaligned reads; SeqCode runs terminate without processing errors on the BAM
- Bins follow the default 100 bp window resolution with 150 bp average fragment size for single-end reads

## Limitations

- Generation of genome-wide profiles in bigwig format is not supported and deferred to future versions
- Unaligned reads must be manually excluded from BAM files; SeqCode commands expect cleaned BAM input
- Normalization is by total reads in the BAM (or a manually supplied spike-in factor); cross-mark comparisons at different depths require care

## Evidence

- [methods] SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth: "SeqCode performed the normalization of values calculated within any finite element (e.g. regions, bins, etc.) for a given sequencing experiment by sequencing depth"
- [methods] Chromosomes are divided into series of consecutive fragments of the same size, which are termed bins.: "Chromosomes are divided into series of consecutive fragments of the same size, which are termed bins."
- [methods] Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands.: "Unaligned reads must be excluded from the resulting BAM files that will be processed by SeqCode commands."
- [readme] Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes): "Most programs require the user to provide two files for the particular genome that is being analyzed: the list of chromosomes (and their sizes)"
- [discussion] Generation of genome-wide profiles in bigwig format62,63, for instance, should be considered in future versions.: "Generation of genome-wide profiles in bigwig format62,63, for instance, should be considered in future versions."
