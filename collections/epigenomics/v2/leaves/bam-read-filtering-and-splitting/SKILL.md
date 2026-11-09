---
name: bam-read-filtering-and-splitting
description: Use when raw ChIP-seq FASTQ reads have been mapped to a combined sample+spike-in
  genome and unaligned reads must be removed before per-bin counting or human/fly
  separation.
when_to_use_negative:
- BAM files are already split into separate sample and spike-in files and contain
  no unmapped reads — spikChIP only requires pre-split bam_sample/bam_spike files
  in the configuration file
- Alignments were produced with a mapper retaining multi-locus reads, deviating from
  the spikChIP protocol in which multi-locus reads are discarded by Bowtie
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3169
tools:
- name: Bowtie
  role: Maps ChIP-seq FASTQ reads to the combined human+fly genome with multi-locus
    reads discarded, producing the alignments that are subsequently filtered
  repo: http://bowtie-bio.sourceforge.net
- name: Samtools
  role: Filters unmapped reads from BAM files (-F 0x4), reports read counts, and supports
    downsampling
- name: spikChIP
  role: Consumes the filtered and split sample/spike BAM files listed in its configuration
    file (map_files/*.bam) for downstream bin segmentation and normalization
  repo: https://github.com/eblancoga/spikChIP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:cf92a4451121995063910d164267f2395fd03231df74fba168711c60ecbdb306
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# BAM read filtering and splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-mapping step of the spikChIP workflow in which Bowtie alignments against the combined human (hg19) + fly (dm3) synthetic genome are filtered to remove unmapped reads (SAMtools -F 0x4) and then split into separate human (sample) and fly (spike-in) BAM files for downstream bin-level counting.

## When to use

Use when raw ChIP-seq FASTQ reads have been mapped to a combined sample+spike-in genome and unaligned reads must be removed before per-bin counting or human/fly separation.

## When NOT to use

- BAM files are already split into separate sample and spike-in files and contain no unmapped reads — spikChIP only requires pre-split bam_sample/bam_spike files in the configuration file
- Alignments were produced with a mapper retaining multi-locus reads, deviating from the spikChIP protocol in which multi-locus reads are discarded by Bowtie

## Inputs

- SAM/BAM alignments of ChIP-seq reads mapped to the combined human+fly (hg19+dm3) genome with Bowtie
- Combined chromInfo file listing hg19 chromosomes and dm3 chromosomes renamed with the mandatory 'FLY' suffix

## Outputs

- Aligned, unmapped-filtered BAM files (unmapped reads removed via SAMtools -F 0x4)
- Split BAM files deposited in map_files/: human (sample) reads and fly (spike-in) reads per condition
- Mapped-read statistics reported by Samtools

## How to apply

Map each ChIP-seq FASTQ library against the concatenated hg19+dm3 synthetic genome index with Bowtie (-p 4 -t -m 1 -S), which discards multi-locus reads as specified in the spikChIP protocol. Filter the resulting SAM/BAM with SAMtools using -F 0x4 to drop unmapped reads, and report mapped-read statistics with Samtools. Split the filtered BAM into human (sample) and fly (spike-in) read sets by chromosome name — fly chromosomes carry the mandatory 'FLY' suffix in the combined chromInfo file — and deposit both in map_files/. These split BAM files are the required input format for spikChIP, which expects bam_sample and bam_spike paths per condition in the configuration file.

## Related tools

- **Bowtie** (Maps ChIP-seq FASTQ reads to the combined human+fly genome with multi-locus reads discarded, producing the alignments that are subsequently filtered) — http://bowtie-bio.sourceforge.net
- **Samtools** (Filters unmapped reads from BAM files (-F 0x4), reports read counts, and supports downsampling)
- **spikChIP** (Consumes the filtered and split sample/spike BAM files listed in its configuration file (map_files/*.bam) for downstream bin segmentation and normalization) — https://github.com/eblancoga/spikChIP

## Examples

```
samtools view -b -F 0x4 mapped_against_hg19_dm3.sam > map_files/1_sample_filtered.bam
```

## Evaluation signals

- Filtered BAM no longer contains reads with the unmapped FLAG (0x4); mapped-read statistics from Samtools confirm a reduced but non-zero read count
- map_files/ contains both a sample (human) and a spike (fly) BAM per experimental condition, matching the configuration file rows
- Downstream SeqCode bin counts approximate ~3,095,665 human bins and ~120,397 fly bins for 1-kbp segmentation consistent with ChromInfo.txt
- Spike-in (fly) read fraction is roughly 2%-5% of the experimental genome, consistent with the article's recommended spike-in depth

## Limitations

- BAM files with fewer than 1 million reads require the spikChIP -d option to be processed
- The final amount of spike-in reads can be influenced by the mixture ratio, antibody quality, and target abundance, so filtered spike-in BAM depth may vary substantially across conditions

## Evidence

- [methods] Bowtie mapping parameters and SAMtools filtering: [evidence span withheld — anchor status: not_found]
- [methods] Samtools role in the pipeline: "Samtools (to inform about number of reads of BAM files and for downsampling)"
- [readme] Split BAM requirement before running spikChIP: "BAM files of each sample/spike genome (e.g. human reads and fruit fly reads) must be generated with a mapping tool prior to running spikChIP by the user."
- [readme] Multi-locus reads discarded during mapping: "We employed Bowtie for mapping (multi-locus reads discarded, http://bowtie-bio.sourceforge.net)"
