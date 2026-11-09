---
name: chipseq-peak-summit-density-windowing
description: Use when you have many narrowPeak ChIP-seq peak files for numerous TFs
  in a genome assembly and need to call HOT regions by peak-summit density rather
  than individual binding sites.
when_to_use_negative:
- When the goal is calling individual TF binding sites or motifs from raw ChIP-seq
  reads rather than aggregating many TF peak files into multi-TF occupancy regions
- When peaks are already summarized into a feature or occupancy table — the skill
  expects raw narrowPeak files whose summit offsets must still be extracted
- 'When studying the biological validity of HOT regions themselves: the paper concludes
  HOT regions ''are likely to be ChIP-seq artifacts'', so downstream interpretation
  should not treat them as genuine multi-TF binding hubs'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: pybedtools
  role: Loading narrowPeak files, extracting peak summit coordinates (chromStart +
    peak offset), and tiling windows per genome assembly
- name: R (HOT-region-main.R)
  role: Repository script used for the HOT regions analysis
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:9f3c6418bbaf87a5c25def05921f8598600c8c717ceef7edafe34d4c3293af9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - Python
  - R (HOT-region-main.R)
  - pybedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# chipseq-peak-summit-density-windowing

## Summary

A genome-tiling density method that defines high-occupancy target (HOT) regions from ENCODE/modENCODE ChIP-seq narrowPeak files by tallying peak summits per sliding window and calling windows at or above the 99th percentile of the genome-wide TF-occupancy distribution, per species (hg19, mm9, dm3, ce10).

## When to use

Use when you have many narrowPeak ChIP-seq peak files for numerous TFs in a genome assembly and need to call HOT regions by peak-summit density rather than individual binding sites.

## When NOT to use

- When the goal is calling individual TF binding sites or motifs from raw ChIP-seq reads rather than aggregating many TF peak files into multi-TF occupancy regions
- When peaks are already summarized into a feature or occupancy table — the skill expects raw narrowPeak files whose summit offsets must still be extracted
- When studying the biological validity of HOT regions themselves: the paper concludes HOT regions 'are likely to be ChIP-seq artifacts', so downstream interpretation should not treat them as genuine multi-TF binding hubs

## Inputs

- ENCODE/modENCODE narrowPeak ChIP-seq peak files per TF dataset and per genome assembly (hg19, mm9, dm3, ce10)
- Genome assembly definitions for window tiling

## Outputs

- BED tracks of HOT regions per species (e.g. hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed)
- Per-species HOT region counts for comparison against reported values (4324 human, 2638 mouse, 422 C. elegans, 408 D. melanogaster)

## How to apply

Load the ENCODE/modENCODE narrowPeak peak files for each assembly (hg19, mm9, dm3, ce10) with Python (pybedtools/pandas) or the repository's R script. Extract each peak summit as chromStart plus the narrowPeak peak-offset column for every TF dataset per species. Tile the genome assembly with consecutive sliding windows and tally the distinct summits per window to build a per-window TF-occupancy count track. Call HOT regions as windows at or above the 99th percentile of the genome-wide occupancy distribution, merging adjacent qualifying windows into BED tracks. Validate against the reported counts: 4324 HOT regions in human, 2638 in mouse, 422 in C. elegans, and 408 in D. melanogaster.

## Related tools

- **pybedtools** (Loading narrowPeak files, extracting peak summit coordinates (chromStart + peak offset), and tiling windows per genome assembly)
- **R (HOT-region-main.R)** (Repository script used for the HOT regions analysis) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- Number of HOT regions per species matches reported values: 4324 (human, hg19), 2638 (mouse, mm9), 422 (C. elegans, ce10), 408 (D. melanogaster, dm3)
- Every HOT region BED interval corresponds to windows at or above the 99th percentile of the genome-wide summit-density distribution
- Adjacent qualifying windows are merged into contiguous intervals in the output BED files
- Summit coordinates reproduce chromStart + peak offset from the narrowPeak columns for every TF dataset
- Called HOT regions coincide with house-keeping gene promoters, as reported in the paper

## Limitations

- HOT regions defined this way are 'likely to be ChIP-seq artifacts' and resemble previously proposed 'hyper-ChIPable' regions, with false positive signals demonstrated in knocked-out TF ChIP-seq data
- HOT regions 'lack canonical motifs for transcription factors that are thought to be bound there', so occupancy counts should not be read as true multi-TF binding
- Lack of signal in a KO ChIP-seq control may instead stem from technical issues such as low read number, low library complexity, or unsuccessful IP
- The paper proposes more stringent filtering, such as removing peaks without canonical motifs, as a necessary refinement of the basic density-based definition

## Evidence

- [methods] Extract the peak summit coordinate from each narrowPeak entry (chromStart + peak offset column) for every TF dataset per species.: [evidence span withheld — anchor status: not_found]
- [methods] Call HOT regions as the windows whose summit density lies at or above the 99th percentile of the genome-wide occupancy distribution, merging adjacent qualifying windows.: [evidence span withheld — anchor status: not_found]
- [results] Applying the peak-summit density method with a 99th percentile TF-occupancy threshold detected 4324 HOT regions in human, 2638 in mouse, 422 in C. elegans and 408 in D. melanogaster.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions.: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
