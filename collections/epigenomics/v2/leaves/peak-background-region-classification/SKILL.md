---
name: peak-background-region-classification
description: Use when ChIP-seq bins (e.g. 1000-bp from spikChIP/SeqCode binning) must
  be partitioned into peaks vs background regions to quantify how a normalization
  strategy (spikChIP, ChIP-Rx, Tag removal) affects enriched vs non-occupied loci.
when_to_use_negative:
- When no externally called peak BED files are available for the sample or spike-in
  genome — the classification is driven by user-provided peaks, so it cannot proceed
  from bins alone.
- When the analysis goal is a single global normalization factor without needing to
  distinguish enriched from non-enriched regions (e.g. a simple total-read-count normalization).
- When peak sets were not called as broad peaks with a compatible caller — the paper's
  workflow specifically used MACS2 broad peaks, and narrow/differently formatted peak
  lists may mislabel bins.
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: SeqCode
  role: Overlaps the 1000-bp bins with the BED peak files to classify each bin into
    peaks or background regions and count reads inside bins
  repo: https://github.com/eblancoga/seqcode
- name: MACS2
  role: Called broad peaks on sample and spike-in BAMs to produce the BED peak files
    used as the classification reference
- name: spikChIP
  role: Pipeline that performs the bin segmentation of sample and spike-in genomes
    and invokes the peak/background classification of both bin sets
  repo: https://github.com/eblancoga/spikChIP
- name: gawk
  role: Manages the plain-text class files to compute per-class (peaks vs background)
    statistics such as median differences between conditions
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  - task_003
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqab064
    title: 'SpikChIP: a novel computational methodology to compare multiple ChIP-seq
      using spike-in chromatin'
  source_content_hash: sha256:274d82fcd66df8b15ce418a6fc80a066ccdfb413200baf68ceaefe1eb9a3b078
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
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-background-region-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify genome-wide 1000-bp bins of both sample and spike-in genomes into peaks versus background regions by overlapping bins with user-provided peak BED files (MACS2 broad peaks) using SeqCode, enabling the paper's key metric: normalization-induced signal differences measured separately over enriched versus non-enriched regions.

## When to use

Use when ChIP-seq bins (e.g. 1000-bp from spikChIP/SeqCode binning) must be partitioned into peaks vs background regions to quantify how a normalization strategy (spikChIP, ChIP-Rx, Tag removal) affects enriched vs non-occupied loci.

## When NOT to use

- When no externally called peak BED files are available for the sample or spike-in genome — the classification is driven by user-provided peaks, so it cannot proceed from bins alone.
- When the analysis goal is a single global normalization factor without needing to distinguish enriched from non-enriched regions (e.g. a simple total-read-count normalization).
- When peak sets were not called as broad peaks with a compatible caller — the paper's workflow specifically used MACS2 broad peaks, and narrow/differently formatted peak lists may mislabel bins.

## Inputs

- MACS2 broad-peak BED files for each sample and spike-in genome (user-provided to spikChIP)
- 1000-bp binned read-count tables (average and maximum) produced by SeqCode for sample and spike-in genomes
- ChromInfo.txt listing sample (hg19) and spike-in (dm3, 'FLY'-suffixed) chromosomes
- BAM files of sample and spike-in reads mapped by Bowtie (multi-locus reads discarded)

## Outputs

- Per-bin peak/background class files in results/, e.g. *_avg_peaks.txt.gz and *_avg_bg.txt.gz for each normalization strategy (RAW, TRADITIONAL, CHIPRX, TAGREMOVAL, SPIKCHIP)
- gawk-computed per-class summary statistics (e.g. median background signal difference between conditions, percentage reduction in peaks vs background)

## How to apply

First call broad peaks with MACS2 on each sample and spike-in BAM and export them as BED files supplied by the user to spikChIP. After spikChIP segments the human (hg19) and spike-in (dm3, 'FLY'-suffixed chromosomes) genomes into 1000-bp bins and counts reads per bin with SeqCode (average and maximum values), overlap the bins against the BED peak files with SeqCode to assign every bin to peaks or background, emitting *_avg_peaks.txt.gz and *_avg_bg.txt.gz class files in results/. Use gawk on these class files to compute per-class statistics such as the median signal difference between conditions. This separation is the rationale of the skill: spike-in correction factors should be increasingly and gradually applied from background to positive ChIP signal regions, so judging a normalization (e.g. ChIP-Rx/Tag removal ~20-25% background shift vs spikChIP <5%) requires knowing which bins are non-enriched.

## Related tools

- **SeqCode** (Overlaps the 1000-bp bins with the BED peak files to classify each bin into peaks or background regions and count reads inside bins) — https://github.com/eblancoga/seqcode
- **MACS2** (Called broad peaks on sample and spike-in BAMs to produce the BED peak files used as the classification reference)
- **spikChIP** (Pipeline that performs the bin segmentation of sample and spike-in genomes and invokes the peak/background classification of both bin sets) — https://github.com/eblancoga/spikChIP
- **gawk** (Manages the plain-text class files to compute per-class (peaks vs background) statistics such as median differences between conditions)

## Examples

```
./spikChIP.pl -vcp 1 config/H3K79me2_25-75_config.txt config/ChromInfo.txt
```

## Evaluation signals

- Compressed class files (e.g. FINAL_H3K79me2_25-75-75-25_TAGREMOVAL_1000_sample_avg.txt.gz and derived peaks/bg files) exist in results/ for both genomes and all normalization strategies
- Every bin of the sample and spike-in genomes is assigned exactly one class (peak or background), with background bins dominating genome-wide counts
- Reproduced metrics fall in expected ranges: e.g. H3K27me3 GSK126 background reduction ~81-85% under ChIP-Rx/Tag removal vs 23% under spikChIP; H3K79me2 background median difference ~20-25% (ChIP-Rx/Tag removal) vs <5% (spikChIP)
- Spike-in equilibration Wilcoxon rank-sum P-values computed per class match reported values (e.g. ChIP-Rx <2.2e-16, tag removal 9.611e-14, spikChIP 0.01547 for GSE64243)
- Background-class distributions remain flat across conditions under spikChIP normalization while peak-class signal declines progressively (e.g. 0% to 100% EPZ5676)

## Limitations

- Classification quality depends entirely on the user-provided peak BED files; peaks must be generated before running spikChIP with an external peak caller (the authors used MACS2 broad peaks).
- The peak/background split defines which bins receive the gradually increasing spike-in correction, so erroneous or incomplete peak calls directly bias the spikChIP normalization and the resulting per-class metrics.
- Bins straddling peak boundaries are assigned by overlap with fixed 1000-bp segmentation, so per-class statistics inherit binning discretization effects.

## Evidence
