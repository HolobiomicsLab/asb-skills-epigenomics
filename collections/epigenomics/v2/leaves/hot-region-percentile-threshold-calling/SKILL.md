---
name: hot-region-percentile-threshold-calling
description: Use when you need to define or reproduce HOT regions computationally
  from many TF ChIP-seq narrowPeak peak files (e.g. ENCODE/modENCODE) for a genome
  like hg19, mm9, dm3 or ce10 using summit-density percentile thresholding.
when_to_use_negative:
- You want biologically validated TF co-binding sites rather than summit-density calls
  — the paper argues HOT regions defined this way are likely ChIP-seq artifacts ('hyper-ChIPable'
  regions) lacking canonical motifs for many TFs thought to be bound there.
- Your input is already a HOT-region BED track or a processed occupancy feature table
  rather than raw narrowPeak peak lists — no summit extraction or window tallying
  is needed.
- You have only one or a few TF ChIP-seq datasets per genome, so a 99th-percentile
  genome-wide occupancy threshold is not meaningful.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: pybedtools
  role: Load narrowPeak files, extract summit coordinates, and tally summits in genomic
    windows
- name: Python
  role: Implementation language for the summit-density windowing and 99th-percentile
    thresholding (pandas/pybedtools)
- name: HOT-region-main.R
  role: Repository analysis script (Scripts/HOT-region-main.R) used for the HOT regions
    analysis
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:ef8d26c398ce0bde686fddaa6a97b5b934561536a37523d62902b7b0d7b2f9af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-region-main.R
  - Python
  - pybedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hot-region-percentile-threshold-calling

## Summary

Calls high-occupancy target (HOT) regions by tallying ChIP-seq peak summits (from narrowPeak files) in sliding windows tiled across a genome assembly and labeling windows at or above the 99th percentile of genome-wide TF-occupancy density as HOT. This reproduces the cross-species HOT-region definitions used in the 'HOT or not' NAR study.

## When to use

Use when you need to define or reproduce HOT regions computationally from many TF ChIP-seq narrowPeak peak files (e.g. ENCODE/modENCODE) for a genome like hg19, mm9, dm3 or ce10 using summit-density percentile thresholding.

## When NOT to use

- You want biologically validated TF co-binding sites rather than summit-density calls — the paper argues HOT regions defined this way are likely ChIP-seq artifacts ('hyper-ChIPable' regions) lacking canonical motifs for many TFs thought to be bound there.
- Your input is already a HOT-region BED track or a processed occupancy feature table rather than raw narrowPeak peak lists — no summit extraction or window tallying is needed.
- You have only one or a few TF ChIP-seq datasets per genome, so a 99th-percentile genome-wide occupancy threshold is not meaningful.

## Inputs

- narrowPeak ChIP-seq peak files for multiple TF datasets (ENCODE/modENCODE) per species
- Reference genome assemblies hg19, mm9, dm3, ce10 (chromosome sizes / tiling coordinates)

## Outputs

- BED files of HOT regions per species (e.g. hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed)
- Per-window TF-occupancy (peak-summit density) count track per assembly
- Per-species HOT region counts (4324 human, 2638 mouse, 422 C. elegans, 408 D. melanogaster)

## How to apply

Load the ENCODE/modENCODE narrowPeak ChIP-seq peak files for each genome assembly (hg19, mm9, dm3, ce10) with Python (pybedtools/pandas). Extract each peak's summit coordinate (chromStart + the narrowPeak 'peak' offset column) for every TF dataset per species. Tile each assembly with consecutive sliding windows and tally distinct peak summits per window to build a per-window TF-occupancy count track. Call HOT regions as windows whose summit density lies at or above the 99th percentile of the genome-wide occupancy distribution, merging adjacent qualifying windows, and write the results as per-species BED tracks (e.g. hot_regions_hg19.bed). The 99th-percentile cutoff is the key threshold decision point; the study applied it identically across human, mouse, C. elegans and D. melanogaster.

## Related tools

- **pybedtools** (Load narrowPeak files, extract summit coordinates, and tally summits in genomic windows)
- **Python** (Implementation language for the summit-density windowing and 99th-percentile thresholding (pandas/pybedtools))
- **HOT-region-main.R** (Repository analysis script (Scripts/HOT-region-main.R) used for the HOT regions analysis) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
python -c "import pybedtools; p=pybedtools.BedTool('peaks.narrowPeak'); p.each(lambda f: [f.chrom, f.start+int(f[9]), f.start+int(f[9])+1, f[3]]).saveas('summits.bed')"
```

## Evaluation signals

- Number of HOT regions per species matches the reported counts: 4324 human, 2638 mouse, 422 C. elegans, 408 D. melanogaster; investigate and report any discrepancy.
- Summit extraction check: each BED3 summit interval equals chromStart + narrowPeak 'peak' offset for a sampled subset of peaks.
- Threshold check: the minimum summit density of called HOT windows equals or exceeds the 99th percentile of the genome-wide per-window occupancy distribution.
- Output schema check: HOT regions are emitted as valid BED tracks per assembly (hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed).
- Adjacency check: adjacent qualifying windows are merged so no two output HOT intervals overlap or abut within one window width.

## Limitations

- HOT regions called by this method are likely ChIP-seq artifacts similar to previously proposed 'hyper-ChIPable' regions, and lack canonical motifs for many TFs thought to be bound there.
- False positive ChIP-seq signals are present on HOT regions even in ChIP-seq data from knocked-out transcription factors; a lack of signal in a KO experiment can also stem from low read numbers, low library complexity or unsuccessful IP.
- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes.

## Evidence

- [abstract] HOT regions are likely to be ChIP-seq artifacts: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [abstract] HOT region definition: "High-occupancy target (HOT) regions are segments of the genome with unusually high number of transcription factor binding sites."
- [intro] ENCODE/modENCODE input data: "Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species."
- [readme] Repository analysis scripts: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
