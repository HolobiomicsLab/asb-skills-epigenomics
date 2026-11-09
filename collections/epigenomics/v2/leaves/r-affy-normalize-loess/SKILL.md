---
name: r-affy-normalize-loess
description: Use when comparing multiple ChIP-seq experiments with exogenous spike-in
  chromatin (e.g. human hg19 + fly dm3) and a genome-wide correction that preserves
  background equilibration is needed rather than a single global scaling factor.
when_to_use_negative:
- The ChIP-seq experiments lack exogenous spike-in chromatin (no sample/spike-in BAM
  pairs), since the correction factors are derived from spike-in bins
- Only a subset of pre-selected loci needs comparison and genome-wide normalized tracks
  are not required
- The input is already normalized ChIP-seq signal (e.g. bigWig tracks) rather than
  mapped BAM files plus peak BED files, which the spikChIP pipeline expects
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: spikChIP
  role: Perl pipeline that orchestrates binning, counting, classification and the
    local-regression normalization; exposes normalize.loess as its SPIKCHIP strategy
  repo: https://github.com/eblancoga/spikChIP
- name: SeqCode
  role: Counts reads of BAM files inside bins and overlaps bins with peak BED files
    to classify peaks vs background
  repo: https://github.com/eblancoga/seqcode
- name: Samtools
  role: Reports the number of reads of BAM files and performs downsampling for normalization
    strategies
- name: gawk
  role: Plain-text column handling of the per-bin normalization files
- name: MACS2
  role: Calls broad peaks (BED) used to classify bins into peaks or background before
    the local regression
  repo: https://pypi.org/project/MACS2
- name: R (affy, MASS)
  role: Runs normalize.loess on traditionally-normalized bin values and draws the
    distribution boxplot PDFs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:a878958844898d0e74f6f17f2745d8c062a40383cf0d3cfbd1d32165aa12e0a7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/eblancoga/spikChIP
  repo_url: https://github.com/eblancoga/spikChIP
  tools:
  - MACS2
  - R
  - R (affy, MASS)
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# R affy normalize.loess local-regression spike-in normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the local regression normalization step of spikChIP in R, using the affy normalize.loess function on traditionally-normalized ChIP-seq bin values guided by spike-in (fly) bins, so that spike-in correction is applied gradually from background to enriched regions instead of uniformly across the genome.

## When to use

Use when comparing multiple ChIP-seq experiments with exogenous spike-in chromatin (e.g. human hg19 + fly dm3) and a genome-wide correction that preserves background equilibration is needed rather than a single global scaling factor.

## When NOT to use

- The ChIP-seq experiments lack exogenous spike-in chromatin (no sample/spike-in BAM pairs), since the correction factors are derived from spike-in bins
- Only a subset of pre-selected loci needs comparison and genome-wide normalized tracks are not required
- The input is already normalized ChIP-seq signal (e.g. bigWig tracks) rather than mapped BAM files plus peak BED files, which the spikChIP pipeline expects

## Inputs

- BAM files of sample and spike-in genomes per condition (e.g. human and fruit fly reads, GSM1465005 / GSM1465007), mapped with Bowtie and multi-locus reads discarded
- BED files of broad peaks called per sample and spike-in genome with MACS2
- Plain-text spikChIP configuration file (columns: condition, bam_sample, bam_spike, peaks_sample, peaks_spike)
- ChromInfo file listing chromosome names and sizes for hg19 and dm3 (FLY suffix convention for spike-in chromosomes)

## Outputs

- SPIKCHIP-normalized genome-wide 1000-bp bin files per condition (e.g. FINAL_H3K79me2_25-75-75-25_SPIKCHIP_1000_avg_normalized_sample.txt.gz and _spike.txt.gz)
- Peak and background bin classification files per genome
- Distribution boxplots of normalized signal as PDFs generated in R (affy and MASS libraries)
- RAW, TRADITIONAL, CHIPRX and TAG REMOVAL normalization tracks computed alongside for benchmarking

## How to apply

Segment the sample genome (hg19) and spike-in genome (dm3, FLY-suffixed chromosomes) into 1000-bp bins per the ChromInfo file, count reads in bins (average and maximum) with SeqCode, and compute RAW, TRADITIONAL, CHIPRX and TAG REMOVAL values. Overlap the bins of both genomes with user-provided MACS2 broad-peak BED files to classify bins into peaks or background. In R, run the local regression step with the affy normalize.loess function on the TRADITIONAL-normalized values, guided by the spike-in (fly) bins with pseudo-count 0.1, to derive the SPIKCHIP correction factors for each condition; the correction is applied increasingly from background to positive ChIP-signal regions so that reads from background bins, though dominating total counts, have minimal influence. Write the SPIKCHIP-normalized genome-wide bin files for sample and spike genomes and generate distribution boxplots as PDFs with R (affy and MASS libraries). Validate that the median background signal differs by less than 5% between conditions under spikChIP, unlike ChIP-Rx and tag removal (~20-25% median difference on the 25:75 vs 75:25 H3K79me2 Jurkat benchmark), and that spike-in ChIP-signal slope equilibration improves over competing strategies.

## Related tools

- **spikChIP** (Perl pipeline that orchestrates binning, counting, classification and the local-regression normalization; exposes normalize.loess as its SPIKCHIP strategy) — https://github.com/eblancoga/spikChIP
- **SeqCode** (Counts reads of BAM files inside bins and overlaps bins with peak BED files to classify peaks vs background) — https://github.com/eblancoga/seqcode
- **Samtools** (Reports the number of reads of BAM files and performs downsampling for normalization strategies)
- **gawk** (Plain-text column handling of the per-bin normalization files)
- **MACS2** (Calls broad peaks (BED) used to classify bins into peaks or background before the local regression) — https://pypi.org/project/MACS2
- **R (affy, MASS)** (Runs normalize.loess on traditionally-normalized bin values and draws the distribution boxplot PDFs)

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 1 config/H3K79me2_25-75_config.txt ChromInfo.txt
```

## Evaluation signals

- Median background ChIP-seq signal differs by less than 5% between the compared conditions under SPIKCHIP normalization (vs ~20-25% for ChIP-Rx and tag removal)
- Spike-in ChIP-signal equilibration (slope between conditions) improves relative to ChIP-Rx and tag removal
- Output contains FINAL_*_SPIKCHIP_1000_avg_normalized_sample.txt.gz and _spike.txt.gz files for every condition in the configuration file
- Both sample and spike-in genomes are segmented into identical 1000-bp bins following the FLY-suffix chromosome convention of the ChromInfo file
- Correction factors are derived from traditionally-normalized values guided by spike-in bins with pseudo-count 0.1 via affy normalize.loess

## Limitations

- BAM files must be generated beforehand with a mapping tool (Bowtie with multi-locus reads discarded) and peak files with an external peak caller (MACS2 broad peaks); spikChIP does not perform mapping or peak calling
- Spike-in reads in the final sequencing step should be at least one million reads, approximately 2%-5% of the experimental genome, to minimize changes in overall material used
- BAM files with fewer than 1 million reads require the -d option; intermediate files can otherwise inflate the output folder (use -c to remove them)
- Using the experimental antibody to capture spike-in material carries a cross-reactivity constraint; a second antibody against a fly-specific histone variant (H2Av) is recommended

## Evidence
