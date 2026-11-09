---
name: genomic-bin-read-counting
description: Use when aligned ChIP-seq BAM files for experimental and spike-in genomes
  must be converted into genome-wide binned read-count tables (e.g. chromosome-start-end-count
  rows at 1-kbp resolution) before cross-condition normalization.
when_to_use_negative:
- Input is still raw FASTQ — reads must first be mapped with Bowtie (multi-locus reads
  discarded) and filtered with Samtools before bin counting
- The analysis goal is peak-level or per-gene quantification rather than genome-wide
  binned signal (e.g. differential binding on MACS2 peaks only)
- No reference chromosome-size file is available — the bin segmentation depends entirely
  on the ChromInfo.txt chromosome list
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Segments both genomes into bins according to ChromInfo.txt and counts reads
    of the BAM files inside each bin (average and maximum values)
  repo: https://github.com/eblancoga/seqcode
- name: spikChIP
  role: Perl pipeline that orchestrates the bin segmentation and per-bin counting
    step prior to the multiple normalization approaches
  repo: https://github.com/eblancoga/spikChIP
- name: Samtools
  role: Reports the number of reads in BAM files and performs downsampling used by
    the normalization strategies
- name: gawk
  role: Formats the per-bin count tables and manages plain-text column files
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_008
  - task_001
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:e3a2d622296058d600f7bcbf0954f12bc05eecb3fa95360f3d6cea85819cbfae
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
  - Bowtie
  - MACS2
  - R
  - Samtools
  - SeqCode
  - gawk
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Genomic bin read counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segments the sample (e.g. human hg19) and spike-in (e.g. fruit fly dm3 with 'FLY' suffix) genomes into non-overlapping 1-kbp bins and counts reads inside each bin (average and maximum values) with SeqCode, producing the per-bin count tables that spikChIP subsequently normalizes with RAW, TRADITIONAL, CHIPRX, TAGREMOVAL and SPIKCHIP approaches.

## When to use

Use when aligned ChIP-seq BAM files for experimental and spike-in genomes must be converted into genome-wide binned read-count tables (e.g. chromosome-start-end-count rows at 1-kbp resolution) before cross-condition normalization.

## When NOT to use

- Input is still raw FASTQ — reads must first be mapped with Bowtie (multi-locus reads discarded) and filtered with Samtools before bin counting
- The analysis goal is peak-level or per-gene quantification rather than genome-wide binned signal (e.g. differential binding on MACS2 peaks only)
- No reference chromosome-size file is available — the bin segmentation depends entirely on the ChromInfo.txt chromosome list

## Inputs

- Mapped sample BAM files (e.g. human hg19 reads, filtered with Samtools -F 0x4)
- Mapped spike-in BAM files (e.g. fruit fly dm3 reads with 'FLY'-suffixed chromosomes)
- ChromInfo.txt plain-text chromosome list with sizes for hg19 + dm3
- MACS2 broad-peak BED files for peaks/background classification of bins
- spikChIP configuration file listing sample, spike BAM and peak BED paths per condition

## Outputs

- Gzipped per-bin count tables (chromosome-start-end-count rows), e.g. results/FINAL_H3K79me2_25-75-75-25_RAW_1000_sample_avg.txt.gz and _sample_max.txt.gz
- Average and maximum per-bin read counts for both sample and spike-in genomes
- Peak vs background bin classification files (*_avg_peaks.txt.gz, *_avg_bg.txt.gz)

## How to apply

Build a ChromInfo.txt listing all chromosomes of both genomes — human (hg19) chromosomes plus fly (dm3) chromosomes renamed with the mandatory 'FLY' suffix. Provide pre-mapped BAM files (Bowtie, multi-locus reads discarded, filtered with Samtools -F 0x4) for each sample and spike genome, plus MACS2 broad-peak BED files if bins must later be classified as peaks or background. Run spikChIP with -k 1 so SeqCode segments both genomes into 1,000-bp non-overlapping bins and counts reads per bin, computing both average and maximum per-bin values (~3,095,665 human bins and ~120,397 fly bins for hg19/dm3). Use Samtools to report BAM read totals (needed for downsampling and ChIP-Rx/tag-removal normalization) and gawk to format the per-bin count tables into gzipped chromosome-start-end-count files under results/.

## Related tools

- **SeqCode** (Segments both genomes into bins according to ChromInfo.txt and counts reads of the BAM files inside each bin (average and maximum values)) — https://github.com/eblancoga/seqcode
- **spikChIP** (Perl pipeline that orchestrates the bin segmentation and per-bin counting step prior to the multiple normalization approaches) — https://github.com/eblancoga/spikChIP
- **Samtools** (Reports the number of reads in BAM files and performs downsampling used by the normalization strategies)
- **gawk** (Formats the per-bin count tables and manages plain-text column files)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 0 config/H3K79me2_25-75_config.txt config/ChromInfo.txt
```

## Evaluation signals

- Number of bins approximates ~3,095,665 human bins and ~120,397 fly bins for the hg19+dm3 1-kbp segmentation
- Each output row is a 4-column chromosome-start-end-count record whose intervals are non-overlapping 1-kbp bins consistent with ChromInfo.txt
- Both the average and maximum per-bin count files (RAW_1000_sample_avg.txt.gz and RAW_1000_sample_max.txt.gz) exist in results/
- BAM read totals reported by Samtools are consistent with the sum of counts plus multi-locus/unmapped reads discarded during mapping

## Limitations

- spikChIP by default refuses to process BAM files with fewer than 1 million reads unless the -d option is used
- The spike-in reads in the final sequencing should be at least one million reads, approximately 2%-5% of the experimental genome, to minimize changes in overall material
- Only bins overlapping user-provided MACS2 broad-peak BED files can be distinguished as peaks; without these BED files no peaks/background separation is possible

## Evidence

- [readme] SeqCode application for counting reads of one sequencing experiment on a set of regions: [evidence span withheld — anchor status: external_doc]
