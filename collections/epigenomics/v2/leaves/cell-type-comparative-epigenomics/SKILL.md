---
name: cell-type-comparative-epigenomics
description: Use when CpG-level methylation calls (e.g., Roadmap WGBS) must be summarized
  per interval set and compared across cell types to test differential methylation
  level/variation between HOT regions and control regions like non-HOT CpG islands.
when_to_use_negative:
- The task is defining HOT regions from ChIP-seq peak data (TF occupancy thresholds)
  rather than comparing methylation over pre-defined region sets
- No WGBS/CpG-level methylation data are available for the cell types of interest,
  so methylation summaries cannot be computed
- The comparison is not cross-cell-type (a single cell type), since medians/IQRs across
  cell types per region set are the core statistics
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bedtools
  role: Interval-overlap tooling to intersect CpG-level methylation calls with each
    BED-style region set
- name: genomation
  role: Interval-overlap tooling (R/Bioconductor) used to compute the mean methylation
    value per region
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:5130e34ecaa17e9c7b80d4ab9eac02ef7c400d74c978c2fadcf53e38c4f949a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bedtools
  - genomation
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cell-type-comparative-epigenomics

## Summary

Compares CpG methylation dynamics across multiple human cell types between defined genomic interval sets (HOT regions, occupancy-binned non-HOT regions, non-HOT CpG islands) using Roadmap Epigenomics WGBS data, to test whether HOT regions are constitutively hypo-methylated while non-HOT CpG islands vary more.

## When to use

Use when CpG-level methylation calls (e.g., Roadmap WGBS) must be summarized per interval set and compared across cell types to test differential methylation level/variation between HOT regions and control regions like non-HOT CpG islands.

## When NOT to use

- The task is defining HOT regions from ChIP-seq peak data (TF occupancy thresholds) rather than comparing methylation over pre-defined region sets
- No WGBS/CpG-level methylation data are available for the cell types of interest, so methylation summaries cannot be computed
- The comparison is not cross-cell-type (a single cell type), since medians/IQRs across cell types per region set are the core statistics

## Inputs

- Roadmap Epigenomics whole-genome bisulfite sequencing (WGBS) methylation profiles per cell type (CpG-level methylation calls)
- BED-style interval files for HOT regions
- BED-style interval files for occupancy-binned non-HOT regions
- BED-style interval files for non-HOT CpG islands

## Outputs

- Per-region mean methylation values for each region set and cell type
- methylation_summary.csv containing medians and IQRs per region set

## How to apply

Retrieve Roadmap Epigenomics WGBS methylation profiles for the available cell types and load the HOT-region, occupancy-binned non-HOT-region, and non-HOT CpG-island coordinate sets as BED-style interval files. For each cell type, intersect CpG-level methylation calls with each region set and compute the mean methylation value per region using interval-overlap tooling (bedtools/genomation). Aggregate per-region means per region set and compute the median and interquartile range (IQR) across cell types for each set. The rationale is that HOT regions coincide with house-keeping gene promoters and are expected to stay hypo-methylated across cell types, whereas non-HOT CpG islands show higher methylation variation; validation expects the non-HOT CpG-island IQR to be 3-4 times the HOT-region IQR.

## Related tools

- **bedtools** (Interval-overlap tooling to intersect CpG-level methylation calls with each BED-style region set)
- **genomation** (Interval-overlap tooling (R/Bioconductor) used to compute the mean methylation value per region)

## Examples

```
bedtools intersect -a non_HOT_CpG_islands.bed -b roadmap_wgbs_CpG_methylation.bed -wa -u > cpg_overlaps.bed
```

## Evaluation signals

- The computed IQR of non-HOT CpG-island methylation is 3-4 times the IQR of HOT-region methylation
- HOT-region median methylation is low (hypo-methylated) relative to non-HOT region sets
- methylation_summary.csv contains a median and an IQR for each of the three region sets aggregated across cell types
- Per-region mean methylation values were produced for every cell type and all three interval sets (no missing region-set/cell-type combinations)

## Limitations

- HOT regions may themselves be ChIP-seq artifacts ('hyper-ChIPable' regions), so methylation differences should not be interpreted as proof of regulatory binding
- HOT regions are solely defined using ChIP-seq experiments and shown to lack canonical motifs for TFs thought to be bound there, limiting functional interpretation
- Comparisons are restricted to the Roadmap cell types for which WGBS methylation profiles are available

## Evidence

- [methods] For each cell type, intersect CpG-level methylation calls with each region set and compute the mean methylation value per region using interval-overlap tooling (bedtools/genomation).: [evidence span withheld — anchor status: not_found]
- [methods] verify that the computed IQR of non-HOT CpG-island methylation is 3-4 times the IQR of HOT-region methylation and that HOT-region median methylation is low relative to non-HOT regions: [evidence span withheld — anchor status: not_found]
- [results] maintaining low (hypo-methylated) methylation levels consistently across different cell types: [evidence span withheld — anchor status: not_found]
- [abstract] Furthermore, they coincide with house-keeping gene promoters and consequently associated genes are stably expressed across multiple cell types.: "Furthermore, they coincide with house-keeping gene promoters and consequently associated genes are stably expressed across multiple cell types."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
