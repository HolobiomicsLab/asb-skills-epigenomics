---
name: wgbs-methylation-profiling
description: Use when comparing DNA methylation levels or methylation variability
  between pre-defined genomic region sets (e.g., HOT regions, occupancy-binned non-HOT
  regions, CpG islands) across multiple cell types using WGBS methylation calls.
when_to_use_negative:
- When the question is about transcription factor binding occupancy or ChIP-seq artifacts
  on HOT regions rather than methylation dynamics
- When only a single cell type is available — the skill's core signal (consistently
  low methylation and cross-cell-type IQR comparison) requires multiple cell types
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bedtools
  role: Intersect CpG-level WGBS methylation calls with HOT, non-HOT, and CpG-island
    BED interval sets to compute overlap-based per-region methylation
- name: genomation
  role: Interval-overlap tooling used as an alternative to bedtools for intersecting
    methylation calls with region sets and computing per-region mean methylation
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:0c4d8930f4607337687c1f4a889e74044cf3b064c2ff5cfd47c6c2109b573016
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - bedtools
  - genomation
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# wgbs-methylation-profiling

## Summary

Profiles CpG methylation dynamics of HOT regions versus non-HOT CpG islands from Roadmap Epigenomics WGBS data by intersecting CpG-level methylation calls with BED interval sets and comparing per-region-set median and IQR across cell types. It tests whether HOT regions are consistently hypo-methylated while non-HOT CpG islands show higher methylation variation.

## When to use

Use when comparing DNA methylation levels or methylation variability between pre-defined genomic region sets (e.g., HOT regions, occupancy-binned non-HOT regions, CpG islands) across multiple cell types using WGBS methylation calls.

## When NOT to use

- When the question is about transcription factor binding occupancy or ChIP-seq artifacts on HOT regions rather than methylation dynamics
- When only a single cell type is available — the skill's core signal (consistently low methylation and cross-cell-type IQR comparison) requires multiple cell types

## Inputs

- Roadmap Epigenomics WGBS CpG-level methylation calls per cell type
- HOT region coordinate set as a BED-style interval file
- Occupancy-binned non-HOT region coordinate set as a BED-style interval file
- Non-HOT CpG island coordinate set as a BED-style interval file

## Outputs

- Per-region mean methylation values for each region set and cell type
- Median and interquartile range (IQR) of methylation per region set across cell types
- methylation_summary.csv with medians and IQRs per region set

## How to apply

Retrieve the Roadmap Epigenomics whole-genome bisulfite sequencing (WGBS) methylation profiles for all available cell types, and load the HOT-region, occupancy-binned non-HOT-region, and non-HOT CpG-island coordinate sets as BED-style interval files. For each cell type, intersect CpG-level methylation calls with each region set using interval-overlap tooling such as bedtools or the Bioconductor package genomation, computing the mean methylation value per region. Aggregate per-region mean methylation values per region set and compute the median and interquartile range (IQR) across cell types for each set. The decision point is the paper's reported relationship: HOT regions should be hypo-methylated with low variation, while non-HOT CpG islands should show methylation variation approximately 3-4 times higher than over HOT regions. Write the per-set medians and IQRs to methylation_summary.csv and validate against that relationship.

## Related tools

- **bedtools** (Intersect CpG-level WGBS methylation calls with HOT, non-HOT, and CpG-island BED interval sets to compute overlap-based per-region methylation)
- **genomation** (Interval-overlap tooling used as an alternative to bedtools for intersecting methylation calls with region sets and computing per-region mean methylation) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
bedtools intersect -a hot_regions.bed -b celltype_WGBS_CpG_meth.bedGraph -wa -u | compute per-region mean methylation with genomation, then write medians/IQRs to methylation_summary.csv
```

## Evaluation signals

- Computed IQR of non-HOT CpG-island methylation is approximately 3-4 times the IQR of HOT-region methylation
- HOT-region median methylation is low (hypo-methylated) relative to non-HOT regions
- HOT-region methylation levels remain consistently low across all queried cell types
- methylation_summary.csv exists and contains medians and IQRs for each of the three region sets (HOT, occupancy-binned non-HOT, non-HOT CpG islands)

## Limitations

- Conclusions about methylation dynamics are tied to the HOT regions defined by ChIP-seq experiments, which the paper argues are likely ChIP-seq artifacts similar to 'hyper-ChIPable' regions
- The 3-4x IQR relationship is a validation target from the paper; other datasets or region definitions may not reproduce the same magnitude
- Only cell types with available Roadmap WGBS profiles can be included in the cross-cell-type comparison

## Evidence

- [results] HOT regions are distinct from non-HOT CpG islands in their methylation dynamics, maintaining low (hypo-methylated) methylation levels consistently across different cell types, whereas non-HOT CpG islands show higher methylation variation.: [evidence span withheld — anchor status: not_found]
- [methods] Retrieve the Roadmap Epigenomics whole-genome bisulfite sequencing (WGBS) methylation profiles for the available cell types.: [evidence span withheld — anchor status: not_found]
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [abstract] Furthermore, they coincide with house-keeping gene promoters and consequently associated genes are stably expressed across multiple cell types.: "they coincide with house-keeping gene promoters and consequently associated genes are stably expressed across multiple cell types"
