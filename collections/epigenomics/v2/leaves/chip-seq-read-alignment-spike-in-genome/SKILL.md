---
name: chip-seq-read-alignment-spike-in-genome
description: Use when raw ChIP-seq FASTQ libraries from a spike-in experiment (e.g.
  Jurkat H3K79me2 GSE60104 with fruit fly spike-in) must be mapped to a concatenated
  synthetic reference and filtered before genome segmentation and normalization.
when_to_use_negative:
- BAM files of each sample/spike genome have already been generated with a mapping
  tool — spikChIP consumes these directly and remapping would be redundant
- The experiment has no exogenous spike-in material, so a synthetic combined genome
  is unnecessary and mapping should target the single sample genome
- Peak calling or normalization stages are the goal — those use MACS2 BED peaks and
  the spikChIP normalization strategies (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP),
  not the mapping step
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0102
tools:
- name: Bowtie
  role: Maps ChIP-seq FASTQ reads to the combined hg19+dm3 synthetic genome indexes
    with multi-locus reads discarded (-m 1)
  repo: http://bowtie-bio.sourceforge.net
- name: Samtools
  role: Removes unmapped reads from alignments (-F 0x4) and reports the number of
    reads in the resulting BAM files
- name: spikChIP
  role: Downstream pipeline that consumes the split sample/spike BAM files from map_files/
    for bin segmentation, counting, and normalization
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of the filtered BAM files inside genome bins after the mapping
    step
  repo: https://github.com/eblancoga/seqcode
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:de59a02bfcf1af719132a0f418c5bfd7a80911c1c35b9c6d0bc7e0b4a7732d17
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  tools:
  - Bowtie
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chip-seq-read-alignment-spike-in-genome

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps raw ChIP-seq FASTQ reads against a combined sample+spike-in (e.g. hg19+dm3) reference genome with Bowtie, discarding multi-locus reads, and removes unmapped alignments with Samtools so downstream per-bin counting can separate human sample reads from fly spike-in reads.

## When to use

Use when raw ChIP-seq FASTQ libraries from a spike-in experiment (e.g. Jurkat H3K79me2 GSE60104 with fruit fly spike-in) must be mapped to a concatenated synthetic reference and filtered before genome segmentation and normalization.

## When NOT to use

- BAM files of each sample/spike genome have already been generated with a mapping tool — spikChIP consumes these directly and remapping would be redundant
- The experiment has no exogenous spike-in material, so a synthetic combined genome is unnecessary and mapping should target the single sample genome
- Peak calling or normalization stages are the goal — those use MACS2 BED peaks and the spikChIP normalization strategies (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP), not the mapping step

## Inputs

- Raw ChIP-seq FASTQ reads (e.g. H3K79me2 Jurkat libraries GSM1465004-GSM1465008 from GSE60104)
- Reference genome sequences hg19 (human) and dm3 (fruit fly)
- Combined sample+spike-in chromInfo file listing hg19 chromosomes and dm3 chromosomes renamed with 'FLY' suffix

## Outputs

- Bowtie indexes for the concatenated hg19+dm3 synthetic genome
- Aligned, unaligned-filtered BAM files per library
- Split BAM files of human (sample) and fly (spike-in) reads deposited in map_files/
- Mapped-read statistics

## How to apply

Build Bowtie indexes for a concatenated hg19+dm3 synthetic genome, using a combined chromInfo file in which every fruit fly (dm3) chromosome carries a mandatory 'FLY' suffix. Map each ChIP-seq FASTQ library with Bowtie using parameters '-p 4 -t -m 1 -S' against the combined index; the '-m 1' setting discards multi-locus reads as specified in the spikChIP protocol. Filter the resulting SAM/BAM with Samtools '-F 0x4' to remove unmapped reads, report mapped-read statistics, and split the filtered BAM into human (sample) and fly (spike-in) read sets deposited in map_files/. These sample/spike BAM pairs are the required input to spikChIP, which later segments both genomes into 1-kbp bins (~3,095,665 human bins and ~120,397 fly bins) and counts reads per bin.

## Related tools

- **Bowtie** (Maps ChIP-seq FASTQ reads to the combined hg19+dm3 synthetic genome indexes with multi-locus reads discarded (-m 1)) — http://bowtie-bio.sourceforge.net
- **Samtools** (Removes unmapped reads from alignments (-F 0x4) and reports the number of reads in the resulting BAM files)
- **spikChIP** (Downstream pipeline that consumes the split sample/spike BAM files from map_files/ for bin segmentation, counting, and normalization) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of the filtered BAM files inside genome bins after the mapping step) — https://github.com/eblancoga/seqcode

## Examples

```
bowtie -p 4 -t -m 1 -S hg19_dm3_index reads.fastq | samtools view -bS -F 0x4 - > map_files/1_sample.bam
```

## Evaluation signals

- BAM files contain only mapped alignments: samtools view -c -f 4 returns 0 (i.e. -F 0x4 filtering was applied)
- Split BAM files exist in map_files/ for both sample (human) and spike (fly) reads of every condition
- Reads align to fly chromosomes carrying the mandatory 'FLY' suffix in the chromInfo file, confirming the synthetic reference was used
- After 1-kbp binning, human and spike-in bin counts approximately match ~3,095,665 and ~120,397 bins respectively
- Multi-locus reads were discarded, consistent with Bowtie '-m 1' alignment statistics

## Limitations

- Multi-locus reads are discarded during mapping, which reduces the number of usable reads
- BAM files and peak BED files must be generated before running spikChIP by the user; the pipeline does not perform mapping or peak calling itself
- spikChIP by default refuses BAM files with fewer than 1 million reads (the -d option must be passed to process smaller files)

## Evidence

- [results] ChIP-seq FASTQ reads are mapped with BOWTIE using parameters -p 4 -t -m 1 -S against the combined human+fly genome indexes: [evidence span withheld — anchor status: not_found]
