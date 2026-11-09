---
name: synthetic-reference-genome-concatenation
description: Use when preprocessing spike-in ChIP-seq FASTQ reads that must align
  to both the experimental genome (hg19) and the exogenous spike-in genome (dm3) before
  spikChIP bin-level normalization.
when_to_use_negative:
- Input BAM files of each sample/spike genome are already generated and split — spikChIP
  expects these pre-mapped inputs and the concatenation/mapping step is redundant
- The spike-in genome is not a known xenogenic reference (FLY/HUMAN/MOUSE/EXO identifiers),
  so no standard concatenated genome can be constructed
- Reads have already been binned or normalized (e.g. a feature table of per-bin counts
  exists) — genome construction and mapping would not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: Bowtie
  role: Maps raw FASTQ reads against the concatenated hg19+dm3 synthetic genome with
    multi-locus reads discarded
  repo: http://bowtie-bio.sourceforge.net
- name: Samtools
  role: Filters unmapped reads from alignments (-F 0x4) and reports the number of
    reads of BAM files
- name: SeqCode
  role: Segments both genomes into 1-kbp bins using ChromInfo.txt and counts reads
    of the filtered BAM files inside each bin
  repo: https://github.com/eblancoga/seqcode
- name: gawk
  role: Formats the per-bin count tables into the RAW normalization output files
- name: spikChIP
  role: Consumes the filtered sample/spike BAM files and the chromInfo file for downstream
    bin-level normalization
  repo: https://github.com/eblancoga/spikChIP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:69c6db25d55fddf44f2ea0d28404afce830f749c8cd888cbc6a23331a1858223
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Synthetic reference genome concatenation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a combined sample+spike-in reference genome (e.g. human hg19 plus fruit fly dm3 chromosomes renamed with a mandatory 'FLY' suffix), builds Bowtie indexes over the concatenation, maps raw ChIP-seq FASTQ reads against it, and filters unmapped reads with SAMtools so sample and spike-in reads can later be separated and counted in 1-kbp bins.

## When to use

Use when preprocessing spike-in ChIP-seq FASTQ reads that must align to both the experimental genome (hg19) and the exogenous spike-in genome (dm3) before spikChIP bin-level normalization.

## When NOT to use

- Input BAM files of each sample/spike genome are already generated and split — spikChIP expects these pre-mapped inputs and the concatenation/mapping step is redundant
- The spike-in genome is not a known xenogenic reference (FLY/HUMAN/MOUSE/EXO identifiers), so no standard concatenated genome can be constructed
- Reads have already been binned or normalized (e.g. a feature table of per-bin counts exists) — genome construction and mapping would not apply

## Inputs

- Raw ChIP-seq FASTQ reads (e.g. GSM1465004-GSM1465008 Jurkat H3K79me2 libraries from GSE60104)
- hg19 and dm3 reference genome sequences
- Combined chromInfo file listing hg19 chromosomes and dm3 chromosomes renamed with 'FLY' suffix

## Outputs

- Aligned BAM files filtered for unmapped reads (-F 0x4), split into sample (human) and spike-in (fly) sets in map_files/
- Mapped-read statistics report
- Per-bin 1-kbp read count tables (chromosome-start-end-count), e.g. results/FINAL_H3K79me2_25-75-75-25_RAW_1000_sample_avg.txt.gz and _max.txt.gz

## How to apply

Download the H3K79me2 Jurkat FASTQ files from GEO (GSM1465004-GSM1465008 under GSE60104) plus the hg19 and dm3 reference sequences. Build a combined chromInfo file listing human chromosomes and fly chromosomes renamed with the mandatory 'FLY' suffix, then concatenate hg19+dm3 and construct Bowtie indexes over the synthetic genome. Map each ChIP-seq library with Bowtie using parameters -p 4 -t -m 1 -S, discarding multi-locus reads as specified in the spikChIP protocol. Filter alignments with SAMtools -F 0x4 to remove unmapped reads, report mapped-read statistics, and split the filtered BAMs into human (sample) and fly (spike-in) read sets deposited in map_files/. Finally use SeqCode with the ChromInfo.txt to segment both genomes into non-overlapping 1-kbp bins (~3,095,665 human bins, ~120,397 fly bins) and count reads per bin with gawk formatting into the RAW normalization outputs.

## Related tools

- **Bowtie** (Maps raw FASTQ reads against the concatenated hg19+dm3 synthetic genome with multi-locus reads discarded) — http://bowtie-bio.sourceforge.net
- **Samtools** (Filters unmapped reads from alignments (-F 0x4) and reports the number of reads of BAM files)
- **SeqCode** (Segments both genomes into 1-kbp bins using ChromInfo.txt and counts reads of the filtered BAM files inside each bin) — https://github.com/eblancoga/seqcode
- **gawk** (Formats the per-bin count tables into the RAW normalization output files)
- **spikChIP** (Consumes the filtered sample/spike BAM files and the chromInfo file for downstream bin-level normalization) — https://github.com/eblancoga/spikChIP

## Examples

```
bowtie -p 4 -t -m 1 -S combined_hg19_dm3_index reads.fastq | samtools view -bS -F 0x4 - > map_files/1_sample.bam
```

## Evaluation signals

- Both RAW output files (FINAL_H3K79me2_25-75-75-25_RAW_1000_sample_avg.txt.gz and _max.txt.gz) exist in results/
- Each output row is a chromosome-start-end-count record for a 1-kbp bin consistent with ChromInfo.txt
- Human bin counts approximate ~3,095,665 bins and spike-in bin counts approximate ~120,397 bins
- Fly chromosomes in the chromInfo file carry the mandatory 'FLY' suffix and are included in the Bowtie index
- SAMtools reports the expected mapped-read totals after the -F 0x4 filter, with multi-locus reads discarded by Bowtie -m 1

## Limitations

- Multi-locus reads are discarded during Bowtie mapping, reducing the number of aligned reads available for binning
- Spike-in material must be clearly xenogenic and separable by reference genome; cross-reactivity or ambiguous chromosome naming would break the human/fly split
- spikChIP by default may not process BAM files of < 1 Million reads unless the -d option is enabled

## Evidence
