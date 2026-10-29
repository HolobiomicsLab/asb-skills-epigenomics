---
name: genomic-interval-coordinate-handling
description: Use when you must convert ENCODE/modENCODE narrowPeak ChIP-seq peak files
  into per-genome interval tracks (e.g. HOT regions) by tallying peak-summit coordinates
  over sliding genomic windows.
when_to_use_negative:
- Input is already a processed interval/feature table rather than raw narrowPeak peak
  files with a summit-offset column.
- The goal is to validate biological function of HOT regions (motif content, expression
  stability) rather than to reproduce their interval definitions from peaks.
- Peaks are broad-peak or non-ENCODE formats lacking the narrowPeak summit offset,
  so chromStart + peak offset cannot be computed.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: Python (pybedtools/pandas)
  role: Load narrowPeak files, extract summit coordinates (chromStart + peak offset),
    tile windows and tally summits
- name: HOT-region-main.R
  role: Repository analysis script used for the HOT region analysis in the published
    manuscript
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:045d5bbb5b0a4d579d0386f59c84a22cafc5eccc4c1607051e95a904b6ad19ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-region-main.R
  - Python
  - Python (pybedtools/pandas)
  - pybedtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genomic-interval-coordinate-handling

## Summary

A coordinate-aware BED/narrowPeak interval workflow for defining high-occupancy target (HOT) regions from ChIP-seq peak summits, as implemented in the 'HOT or not' analysis across hg19, mm9, dm3 and ce10 assemblies.

## When to use

Use when you must convert ENCODE/modENCODE narrowPeak ChIP-seq peak files into per-genome interval tracks (e.g. HOT regions) by tallying peak-summit coordinates over sliding genomic windows.

## When NOT to use

- Input is already a processed interval/feature table rather than raw narrowPeak peak files with a summit-offset column.
- The goal is to validate biological function of HOT regions (motif content, expression stability) rather than to reproduce their interval definitions from peaks.
- Peaks are broad-peak or non-ENCODE formats lacking the narrowPeak summit offset, so chromStart + peak offset cannot be computed.

## Inputs

- ENCODE/modENCODE narrowPeak ChIP-seq peak files for multiple TF datasets
- Genome assembly identifiers: hg19, mm9, dm3, ce10
- Chromosome size / genome tiling definition for sliding windows

## Outputs

- hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed (merged 99th-percentile summit-density windows)
- Per-window TF-occupancy count track per species
- Per-species HOT region counts for validation against 4324/2638/422/408

## How to apply

Load the ENCODE/modENCODE narrowPeak files for the genome assembly of interest (hg19, mm9, dm3, ce10) with Python (pybedtools/pandas). Extract the peak summit from each narrowPeak entry as chromStart + the peak-offset column, so all TF datasets per species are reduced to a common 1-bp coordinate space. Tally distinct summits falling into consecutive sliding windows tiling the assembly to build a per-window TF-occupancy count track, then call windows whose summit density is at or above the 99th percentile of the genome-wide occupancy distribution, merging adjacent qualifying windows. Write the merged intervals as BED tracks (e.g. hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed) and compare counts against the reported 4324 human, 2638 mouse, 422 C. elegans and 408 D. melanogaster HOT regions. Rationale: HOT regions are defined solely from ChIP-seq experiments, so consistent summit-coordinate handling across assemblies is what makes the cross-species comparison of sequence features valid.

## Related tools

- **Python (pybedtools/pandas)** (Load narrowPeak files, extract summit coordinates (chromStart + peak offset), tile windows and tally summits)
- **HOT-region-main.R** (Repository analysis script used for the HOT region analysis in the published manuscript) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
import pybedtools as bt; summits = bt.BedTool('encode_tf.narrowPeak').each(lambda f: bt.create_interval_from_list([f.chrom, str(int(f.start)+int(f[9])), str(int(f.start)+int(f[9])+1), f.name]))
```

## Evaluation signals

- Number of HOT regions per species matches the reported counts: 4324 human, 2638 mouse, 422 C. elegans, 408 D. melanogaster; report any discrepancy.
- Each output BED interval lies within the correct assembly (hg19/mm9/dm3/ce10) and summit coordinates equal chromStart + narrowPeak peak-offset column.
- Called windows have summit density at or above the 99th percentile of the genome-wide occupancy distribution.
- Adjacent qualifying windows are merged with no unmerged overlaps in the BED output.
- The same 1-bp summit definition is applied to every TF dataset per species (consistent cross-TF, cross-species comparison).

## Limitations

- HOT regions defined this way are likely to be ChIP-seq artifacts ('hyper-ChIPable' regions), so downstream interpretation of the intervals must account for artifact enrichment.
- A threshold choice (99th percentile of genome-wide occupancy) directly drives the number of reported HOT regions; different thresholds change the counts.
- Lack of signal in knockout ChIP-seq validation may reflect technical conditions such as low number of reads, low library complexity or unsuccessful IP, not absence of binding.

## Evidence

- [other] Extract the peak summit coordinate from each narrowPeak entry (chromStart + peak offset column) for every TF dataset per species.: [evidence span withheld — anchor status: not_found]
- [other] 99th percentile threshold and reported HOT counts: [evidence span withheld — anchor status: not_found]
- [abstract] HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [readme] Repository scripts used for the analysis: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
