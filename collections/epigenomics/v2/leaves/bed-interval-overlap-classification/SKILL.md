---
name: bed-interval-overlap-classification
description: Use when a spikChIP-style spike-in normalization needs every segmentation
  bin of the sample (and spike-in) genome labelled as ChIP-enriched or background,
  from user-supplied BED peak files.
when_to_use_negative:
- When the BED peak files have not yet been generated — peaks must be called beforehand
  (e.g. MACS2 broad peaks on the Bowtie-mapped BAM); spikChIP only checks their existence
  in Step 0
- When a genome-wide normalization without any pre-defined selection of loci is desired
  and no external peak set should bias the classification
- When bins must have variable or annotation-driven sizes rather than fixed-size bins
  following ChromInfo.txt
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Provides the matchpeaks function used to compute the overlap between the 1-kbp
    segmentation bins and the BED peak intervals; also counts BAM reads in bins
  repo: https://github.com/eblancoga/seqcode
- name: MACS2
  role: Peak caller run in broad-peak mode on the reference-sample BAM to produce
    the BED peak set used for bin classification
  repo: https://pypi.org/project/MACS2
- name: spikChIP
  role: Perl pipeline embedding the segmentation (-k bin size) and bin classification
    steps within its genome-wide spike-in normalization workflow
  repo: https://github.com/eblancoga/spikChIP
- name: Bowtie
  role: Mapping tool that generated the BAM files of each sample/spike genome prior
    to peak calling (multi-locus reads discarded)
  repo: http://bowtie-bio.sourceforge.net
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:dd9e2ab1d7cb665a4a17823c0539f94fd0ab50a8c2d2bbd5dbea83455c16acf1
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
  - SeqCode
  - spikChIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# BED interval overlap classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Labels fixed-size genomic bins (e.g. 1,000-bp bins of the hg19 sample genome) as 'peak' or 'background' by computing the overlap between MACS2-called broad peaks in BED format and the segmentation bins using SeqCode's matchpeaks, the bin-classification step of the spikChIP spike-in normalization pipeline.

## When to use

Use when a spikChIP-style spike-in normalization needs every segmentation bin of the sample (and spike-in) genome labelled as ChIP-enriched or background, from user-supplied BED peak files.

## When NOT to use

- When the BED peak files have not yet been generated — peaks must be called beforehand (e.g. MACS2 broad peaks on the Bowtie-mapped BAM); spikChIP only checks their existence in Step 0
- When a genome-wide normalization without any pre-defined selection of loci is desired and no external peak set should bias the classification
- When bins must have variable or annotation-driven sizes rather than fixed-size bins following ChromInfo.txt

## Inputs

- BAM file of the reference sample genome (e.g. Bowtie-mapped human hg19 reads, multi-locus reads discarded)
- BED peak file produced by MACS2 broad-peak mode (e.g. peak_files/1_sample_peaks.bed)
- ChromInfo.txt with chromosome names and lengths for sample (hg19) and spike-in (dm3, FLY-suffixed) genomes
- spikChIP plain-text configuration file mapping each condition to its sample/spike BAM and BED peak files
- Bin size parameter (-k, e.g. 1,000 bp / 1 kbp)

## Outputs

- Labelled bin files assigning each 1-kbp segmentation bin to the peaks class
- Labelled bin files assigning each bin to the background class
- Results folder containing the peak/background classification used by downstream spikChIP normalization (RAW, TRADITIONAL, CHIPRX, TAG REMOVAL, SPIKCHIP)

## How to apply

First run MACS2 in broad-peak mode (the mode used for H3K79me2) on the BAM of the reference sample (e.g. Jurkat_K79_25%_R1, GSM1465005) and convert the result to a BED peak file such as peak_files/1_sample_peaks.bed; peaks must be generated before running spikChIP. Load the configuration file and ChromInfo.txt listing the chromosome names/lengths of the sample (hg19) and spike-in (dm3, 'FLY'-suffixed) genomes, and segment both genomes into fixed-size bins via the -k option (1,000 bp in the published workflow). Then use the matchpeaks function of the SeqCode package to compute the overlap between the 1-kbp bins and the BED peaks. Bins overlapping a peak are written to the peaks class and all remaining bins to the background class, with the labelled bin files stored in the results folder. The rationale is that spikChIP focuses its local-regression correction on ChIP-signal-enriched regions and diminishes secondary effects over the background, so this binary labelling drives which bins receive the correction.

## Related tools

- **SeqCode** (Provides the matchpeaks function used to compute the overlap between the 1-kbp segmentation bins and the BED peak intervals; also counts BAM reads in bins) — https://github.com/eblancoga/seqcode
- **MACS2** (Peak caller run in broad-peak mode on the reference-sample BAM to produce the BED peak set used for bin classification) — https://pypi.org/project/MACS2
- **spikChIP** (Perl pipeline embedding the segmentation (-k bin size) and bin classification steps within its genome-wide spike-in normalization workflow) — https://github.com/eblancoga/spikChIP
- **Bowtie** (Mapping tool that generated the BAM files of each sample/spike genome prior to peak calling (multi-locus reads discarded)) — http://bowtie-bio.sourceforge.net

## Examples

```
./spikChIP -vhcde FLY -k 1 -p 0 configuration_file ChromInfo.txt
```

## Evaluation signals

- The peak BED file output exists and is in valid BED format
- The labelled bin files contain non-null bins assigned to both the peaks and the background classes, reproducing the published spikChIP results
- Every bin labelled as peak overlaps at least one interval of the MACS2 broad-peak BED file per the matchpeaks result
- The number of bins equals the segmentation of all chromosomes declared in ChromInfo.txt at the chosen -k size (1,000 bp)
- Running spikChIP Step 0 completes without errors, i.e. the configuration file's BAM and BED paths all exist

## Limitations

- Classification quality depends entirely on the user-provided peaks; spikChIP 'diminishing secondary effects of the normalization over the background' holds only if the external peak caller output is trustworthy
- Peaks must be called in the appropriate mode (broad peaks for H3K79me2); narrow-peak defaults could mislabel broad histone-mark domains
- Multi-locus reads are discarded during mapping, and BAM files with < 1 million reads require the -d flag, both of which affect the reads available for peak calling and bin counting

## Evidence

- [methods] Classification of the bins of both genomes into peaks or background regions according to the peaks provided by the user: "Classification of the bins of both genomes into peaks or background regions according to the peaks provided by the user"
- [methods] SeqCode (to count reads of BAM files, overlap bins and peaks of BED files): "SeqCode (to count reads of BAM files, overlap bins and peaks of BED files)"
