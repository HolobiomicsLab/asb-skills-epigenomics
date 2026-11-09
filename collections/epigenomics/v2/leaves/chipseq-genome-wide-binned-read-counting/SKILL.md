---
name: chipseq-genome-wide-binned-read-counting
description: Use when ChIP-seq BAM files (experimental sample plus exogenous spike-in)
  must be converted into fixed-bin read-count tables across the whole genome, e.g.
  to support comparative spike-in normalization or peak-vs-background signal quantification.
when_to_use_negative:
- Input is already a per-bin count table or normalized signal matrix rather than mapped
  BAM files
- The analysis requires per-read or sub-bin (single-base) resolution instead of fixed-size
  bin aggregates
- No chromosome-size reference (ChromInfo) is available for the genome assembly, so
  bins cannot be defined
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Counts reads of BAM files overlapping bins of BED-defined genome segmentation,
    reporting average and maximum per-bin values
  repo: https://github.com/eblancoga/seqcode
- name: spikChIP
  role: Orchestrates bin segmentation of sample and spike-in genomes (via -k bin size
    and ChromInfo file) and computes multiple normalizations on the resulting bin
    counts
  repo: https://github.com/eblancoga/spikChIP
- name: Samtools
  role: Informs about the number of reads of BAM files and performs downsampling prior
    to binned counting
- name: gawk
  role: Manages the plain-text per-bin count tables to classify bins into peaks versus
    background using MACS2 broad-peak BED files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:94b6fc873363fc1e807645e33939b82cfe7257203307de17f6f98717f983c7d7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# chipseq-genome-wide-binned-read-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Genome-wide ChIP-seq signal quantification by segmenting the sample (e.g. human hg19) and spike-in (e.g. fly dm3) genomes into fixed-size bins (e.g. 1000 bp) and counting reads per bin (average and maximum) with SeqCode, producing count tables that downstream spike-in normalization strategies (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP) operate on.

## When to use

Use when ChIP-seq BAM files (experimental sample plus exogenous spike-in) must be converted into fixed-bin read-count tables across the whole genome, e.g. to support comparative spike-in normalization or peak-vs-background signal quantification.

## When NOT to use

- Input is already a per-bin count table or normalized signal matrix rather than mapped BAM files
- The analysis requires per-read or sub-bin (single-base) resolution instead of fixed-size bin aggregates
- No chromosome-size reference (ChromInfo) is available for the genome assembly, so bins cannot be defined

## Inputs

- Mapped ChIP-seq BAM files for the experimental sample genome (e.g. human hg19 reads)
- Mapped spike-in BAM files for the exogenous genome (e.g. fruit fly dm3 reads)
- ChromInfo.txt chromosome-size file for the genome assembly
- spikChIP configuration file mapping each condition to its sample BAM, spike BAM and peak BED files
- BED peak files (MACS2 broad peaks) for bin classification
- Bin size parameter (e.g. 1000 bp) via spikChIP -k

## Outputs

- Per-bin read-count tables with average and maximum values (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz)
- Normalized bin tables for each strategy (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP)
- Peak/background bin classifications used for region-level signal statistics

## How to apply

Load the spikChIP configuration file (e.g. H3K79me2_25-75_config.txt) and the ChromInfo.txt chromosome-size file, and verify that the human and fly BAM files for every condition exist as declared. Choose the bin size with the spikChIP -k option (the H3K79me2 task uses 1000 bp) and segment both the sample genome and the spike-in genome into bins of the same size according to ChromInfo.txt. Use SeqCode to count reads per bin, reporting both average and maximum values per bin, and emit the normalized tables for the requested strategies (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP). The resulting per-bin tables (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz) then feed gawk-based classification of bins into peaks versus background using MACS2 broad-peak BED files, enabling region-class-specific signal comparisons such as the ~20-25% background median difference between the 25:75 and 75:25 conditions.

## Related tools

- **SeqCode** (Counts reads of BAM files overlapping bins of BED-defined genome segmentation, reporting average and maximum per-bin values) — https://github.com/eblancoga/seqcode
- **spikChIP** (Orchestrates bin segmentation of sample and spike-in genomes (via -k bin size and ChromInfo file) and computes multiple normalizations on the resulting bin counts) — https://github.com/eblancoga/spikChIP
- **Samtools** (Informs about the number of reads of BAM files and performs downsampling prior to binned counting)
- **gawk** (Manages the plain-text per-bin count tables to classify bins into peaks versus background using MACS2 broad-peak BED files)

## Examples

```
./spikChIP -vhcde FLY -k 1000 -p 1 H3K79me2_25-75_config.txt ChromInfo.txt
```

## Evaluation signals

- Compressed normalized bin tables (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and _max.txt.gz) exist in the results/ folder
- Bin intervals are non-overlapping, fixed-size (e.g. 1000 bp) and consistent with the chromosome lengths listed in ChromInfo.txt
- Per-bin average and maximum counts are non-negative integers and the max column is >= the average column for every bin
- Sum of counts across the sample bins is consistent with the total sample reads reported by Samtools for that BAM
- Downstream gawk peak/background classification yields the reported behavior, e.g. ~20-25% median background-region difference between H3K79me2 25:75 and 75:25 conditions under TAG REMOVAL

## Limitations

- BAM files must be pre-generated by the user with a mapping tool (the authors used Bowtie with multi-locus reads discarded) and peak BEDs pre-computed with a peak caller (MACS2 broad peaks) before bin counting
- spikChIP requires BAM files with sufficient reads; the -d option is needed to process BAM files of < 1 Million reads, otherwise processing fails
- Binned counting is performed separately on both the sample and spike-in genomes, so both genome assemblies need their own chromosome-size information

## Evidence
