---
name: cross-species-hot-region-count-validation
description: Use when narrowPeak ChIP-seq peak files for multiple genomes (hg19, mm9,
  dm3, ce10) must be converted into HOT region BED tracks and per-species counts checked
  against published HOT-region numbers.
when_to_use_negative:
- When the goal is functional interpretation of already-called HOT regions (e.g. GC/CpG
  k-mer, R-loop or G-quadruplex enrichment) rather than de novo calling and count
  validation from peak files.
- When only a single TF's ChIP-seq peaks are available — HOT region definition requires
  aggregated summit density across hundreds of TF datasets from ENCODE/modENCODE.
- 'When assessing whether HOT region signals are antibody-specific or artifact-free:
  the paper states it is not possible to estimate the proportion of antibody-specific
  error with currently available data.'
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Python (pybedtools/pandas)
  role: Load narrowPeak files, extract summit coordinates (chromStart + peak offset),
    tally summits per sliding window, apply the 99th-percentile threshold, and write
    HOT region BED tracks per species.
- name: HOT-or-not manuscript analysis scripts
  role: R scripts (Scripts/HOT-region-main.R) used in the HOT regions manuscript analysis;
    the reference implementation behind the published HOT region calls.
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:77a89f57ca09ec62e7cbdcc0420c249e4a021615eab8d348787ac50ffb189a8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-or-not manuscript analysis scripts
  - Python
  - Python (pybedtools/pandas)
  - pybedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-species-hot-region-count-validation

## Summary

Reproduce high-occupancy target (HOT) region calls from ENCODE/modENCODE ChIP-seq narrowPeak files across human, mouse, D. melanogaster and C. elegans using a peak-summit density method with a 99th-percentile occupancy threshold, and validate species-level counts against the HOT-or-not paper (4324 human, 2638 mouse, 422 C. elegans, 408 D. melanogaster).

## When to use

Use when narrowPeak ChIP-seq peak files for multiple genomes (hg19, mm9, dm3, ce10) must be converted into HOT region BED tracks and per-species counts checked against published HOT-region numbers.

## When NOT to use

- When the goal is functional interpretation of already-called HOT regions (e.g. GC/CpG k-mer, R-loop or G-quadruplex enrichment) rather than de novo calling and count validation from peak files.
- When only a single TF's ChIP-seq peaks are available — HOT region definition requires aggregated summit density across hundreds of TF datasets from ENCODE/modENCODE.
- When assessing whether HOT region signals are antibody-specific or artifact-free: the paper states it is not possible to estimate the proportion of antibody-specific error with currently available data.

## Inputs

- ENCODE/modENCODE narrowPeak ChIP-seq peak files per TF dataset for hg19 (human)
- ENCODE/modENCODE narrowPeak ChIP-seq peak files per TF dataset for mm9 (mouse)
- modENCODE narrowPeak ChIP-seq peak files per TF dataset for dm3 (D. melanogaster)
- modENCODE narrowPeak ChIP-seq peak files per TF dataset for ce10 (C. elegans)

## Outputs

- hot_regions_hg19.bed (human HOT regions)
- hot_regions_mm9.bed (mouse HOT regions)
- hot_regions_dm3.bed (D. melanogaster HOT regions)
- hot_regions_ce10.bed (C. elegans HOT regions)
- per-species HOT region counts for comparison with 4324/2638/422/408

## How to apply

Load the ENCODE/modENCODE narrowPeak files for each assembly (hg19, mm9, dm3, ce10) with Python (pybedtools/pandas) and extract the peak summit coordinate (chromStart plus the narrowPeak peak-offset column) for every TF dataset per species. Tile each genome with sliding windows and tally distinct peak summits per window to build a genome-wide TF-occupancy count track. Call HOT regions as windows whose summit density lies at or above the 99th percentile of that occupancy distribution, merging adjacent qualifying windows, and write per-species BED tracks (e.g. hot_regions_hg19.bed, hot_regions_mm9.bed, hot_regions_dm3.bed, hot_regions_ce10.bed). Compare the resulting counts against the reported 4324/2638/422/408 HOT regions and report discrepancies; note the paper's caveat that such HOT regions may be ChIP-seq artifacts similar to 'hyper-ChIPable' regions.

## Related tools

- **Python (pybedtools/pandas)** (Load narrowPeak files, extract summit coordinates (chromStart + peak offset), tally summits per sliding window, apply the 99th-percentile threshold, and write HOT region BED tracks per species.)
- **HOT-or-not manuscript analysis scripts** (R scripts (Scripts/HOT-region-main.R) used in the HOT regions manuscript analysis; the reference implementation behind the published HOT region calls.) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- Human HOT region count matches 4324; mouse 2638; C. elegans 422; D. melanogaster 408.
- Called HOT windows have summit density at or above the 99th percentile of the genome-wide TF-occupancy distribution.
- Each output BED track coordinates match the correct assembly (hg19, mm9, dm3, ce10) and contain merged, non-adjacent-overlapping intervals.
- Summit extraction uses chromStart + narrowPeak peak offset column, not chromStart alone.
- Number of qualifying windows before and after merging adjacent windows is recorded for the discrepancy report.

## Limitations

- HOT regions detected this way are likely to be ChIP-seq artifacts, similar to previously proposed 'hyper-ChIPable' regions, so counts should not be interpreted as biological binding events.
- Lack of signal in KO ChIP-seq controls may reflect technical conditions (low read number, low library complexity, unsuccessful IP), confounding artifact interpretation.
- It is not possible with currently available data to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes.
- The paper proposes more stringent filtering (e.g. removing peaks without canonical motifs) as a remedy; the raw 99th-percentile counts are not artifact-corrected.

## Evidence

- [other] Applying the peak-summit density method with a 99th percentile TF-occupancy threshold detected 4324 HOT regions in human, 2638 in mouse, 422 in C. elegans and 408 in D. melanogaster.: [evidence span withheld — anchor status: not_found]
- [other] Extract summit from narrowPeak: chromStart + peak offset column.: [evidence span withheld — anchor status: not_found]
- [abstract] HOT regions are likely to be ChIP-seq artifacts.: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [intro] Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species.: "Large scale projects such as ENCODE (1) and modENCODE (2) used this technology to find the binding sites of hundreds of proteins in multiple species."
- [readme] The scripts in this repository have been used for analysis of HOT regions.: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
