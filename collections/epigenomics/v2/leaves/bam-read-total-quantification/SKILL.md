---
name: bam-read-total-quantification
description: Use when running spike-in ChIP-seq normalization (e.g. ChIP-Rx in spikChIP)
  where each human bin count must be divided by total fly (dm3) spike-in reads, or
  when BAM totals must be reported or used for downsampling.
when_to_use_negative:
- When the goal is the spikChIP local-regression normalization itself, which deliberately
  minimizes the influence of background-region reads rather than relying on a raw
  total-read division
- When a BAM file contains fewer than 1 million reads — spikChIP excludes these by
  default unless the -d option is explicitly set
- When the experiment has no exogenous spike-in genome; ChIP-Rx requires a second
  genome's read total as denominator
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: Samtools
  role: Reports the total number of reads of sample and spike-in BAM files and performs
    downsampling; its totals are the ChIP-Rx denominators
  repo: http://samtools.sourceforge.net/samtools-c.shtml
- name: spikChIP
  role: Pipeline that checks BAM existence (Step 0) and applies the CHIPRX normalization
    dividing human bin counts by total dm3 spike-in reads
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of the BAM files inside 1000-bp bins (average and maximum values)
    prior to normalization
  repo: https://github.com/eblancoga/seqcode
- name: Bowtie
  role: Mapping tool that generated the BAM files being quantified (multi-locus reads
    discarded)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:fcd545cd228a4a336bcefa68971e5090f0ccf72316d5a61ccb774117aa80f711
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bam-read-total-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the total number of mapped reads in sample and spike-in BAM files using Samtools, providing the denominators required by spike-in normalization approaches (notably ChIP-Rx) computed within the spikChIP pipeline.

## When to use

Use when running spike-in ChIP-seq normalization (e.g. ChIP-Rx in spikChIP) where each human bin count must be divided by total fly (dm3) spike-in reads, or when BAM totals must be reported or used for downsampling.

## When NOT to use

- When the goal is the spikChIP local-regression normalization itself, which deliberately minimizes the influence of background-region reads rather than relying on a raw total-read division
- When a BAM file contains fewer than 1 million reads — spikChIP excludes these by default unless the -d option is explicitly set
- When the experiment has no exogenous spike-in genome; ChIP-Rx requires a second genome's read total as denominator

## Inputs

- Mapped sample ChIP-seq BAM files (e.g. human hg19, GSM1465005 Jurkat_K79_25%_R1)
- Mapped spike-in BAM files (e.g. dm3 fruit fly reads from the same mixture)
- spikChIP configuration file listing bam_sample and bam_spike per condition (e.g. config/H3K79me2_25-75_config.txt)
- ChromInfo.txt chromosome-size file for the combined genome (hg19 + dm3)

## Outputs

- Total read counts reported per BAM file (sample and spike-in) for use as normalization denominators
- CHIPRX-normalized gzip-compressed bin tables written to results/ (bin counts divided by total spike-in reads)

## How to apply

After mapping with Bowtie (multi-locus reads discarded) into sample and spike-in BAM files, spikChIP's Step 0 loads the configuration file and checks the existence of the BAM files listed per condition. Use Samtools to report the total number of reads in each human sample BAM (e.g. GSM1465005) and each fruit fly spike-in BAM (e.g. GSM1465007). SeqCode then counts reads inside 1000-bp bins of both genomes (computing average and maximum per-bin values), and the CHIPRX normalization divides each human bin count by the total dm3 spike-in read count. Samtools totals also serve downsampling, and the spike-in total should be checked to be at least ~1 million reads (approximately 2%-5% of the experimental genome).

## Related tools

- **Samtools** (Reports the total number of reads of sample and spike-in BAM files and performs downsampling; its totals are the ChIP-Rx denominators) — http://samtools.sourceforge.net/samtools-c.shtml
- **spikChIP** (Pipeline that checks BAM existence (Step 0) and applies the CHIPRX normalization dividing human bin counts by total dm3 spike-in reads) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of the BAM files inside 1000-bp bins (average and maximum values) prior to normalization) — https://github.com/eblancoga/seqcode
- **Bowtie** (Mapping tool that generated the BAM files being quantified (multi-locus reads discarded))

## Examples

```
samtools view -c map_files/GSM1465005_sample.bam; samtools view -c map_files/GSM1465005_spike.bam
```

## Evaluation signals

- The reported spike-in (dm3) total is at least ~1 million reads and roughly 2%-5% of the experimental genome read count
- Re-running the read-count command on the same BAM (e.g. samtools idxstats / view -c) reproduces the same total, confirming no downsampling or filtering was silently applied
- Applying CHIPRX to the 25:75 vs 75:25 H3K79me2 data yields a genome-wide median background signal difference of approximately 20-25% between conditions
- Step 0 of spikChIP confirms the existence of every BAM listed in the configuration file before counting proceeds
- Bin counting outputs contain both average and maximum per-bin values for every condition

## Limitations

- Bowtie mapping discarded multi-locus reads, so BAM totals reflect uniquely mapped reads only and can differ from raw sequenced-read counts
- The final amount of spike-in reads can be influenced by the ratio of the cell mixture as well as by the quality of the antibody and/or the abundance of the target in the experimental condition
- BAM files of < 1 million reads are excluded by default (use the -d flag to override), which can block processing of low-yield spike-in libraries

## Evidence

- [methods] Samtools role in read quantification: "Samtools (to inform about number of reads of BAM files and for downsampling)"
- [methods] Counting reads inside bins before normalization: "Perform the counting of reads inside the bins (average and maximum values) and normalize these initial values with multiple approaches"
- [readme] BAM files as prerequisite input: "BAM files of each sample/spike genome (e.g. human reads and fruit fly reads) must be generated with a mapping tool prior to running spikChIP"
- [discussion] Expected spike-in read depth: "the number of spike-in reads in the final sequencing step should be at least one million reads, and approximately, 2%–5% of the experimental genome"
