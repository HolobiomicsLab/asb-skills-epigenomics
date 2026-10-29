---
name: region-set-summary-statistics
description: Use when you have CpG-level methylation calls (e.g. Roadmap WGBS) for
  several cell types plus BED-style genomic interval sets (HOT regions, non-HOT CpG
  islands) and need per-region and per-set median/IQR methylation summaries.
when_to_use_negative:
- The goal is to characterize HOT regions by sequence features (GC/CpG-rich k-mers,
  R-loops, G-quadruplex DNA) rather than methylation summary statistics
- ChIP-seq peak signal analysis on HOT regions (e.g. KO-transcription-factor false-positive
  signal assessment) rather than WGBS methylation aggregation
- The methylation data are single-cell or allele-specific and per-region means would
  obscure the variation being measured
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: bedtools
  role: Intersects CpG-level methylation calls with each region set of intervals for
    per-region mean methylation computation
- name: genomation
  role: Alternative interval-overlap tooling for computing per-region methylation
    summaries over BED interval sets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:80b8fbce646f146fe2d4feab8f1187f3f0bb72db01b2aa5d800c0a9756860d72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bedtools
  - genomation
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# region-set-summary-statistics

## Summary

Computes per-region and per-region-set summary statistics (median and IQR) of CpG methylation across multiple Roadmap WGBS cell types for HOT regions, occupancy-binned non-HOT regions, and non-HOT CpG islands, to quantify methylation dynamics differences between region sets.

## When to use

Use when you have CpG-level methylation calls (e.g. Roadmap WGBS) for several cell types plus BED-style genomic interval sets (HOT regions, non-HOT CpG islands) and need per-region and per-set median/IQR methylation summaries.

## When NOT to use

- The goal is to characterize HOT regions by sequence features (GC/CpG-rich k-mers, R-loops, G-quadruplex DNA) rather than methylation summary statistics
- ChIP-seq peak signal analysis on HOT regions (e.g. KO-transcription-factor false-positive signal assessment) rather than WGBS methylation aggregation
- The methylation data are single-cell or allele-specific and per-region means would obscure the variation being measured

## Inputs

- Roadmap Epigenomics WGBS CpG-level methylation calls per cell type
- BED-style interval files for HOT regions
- BED-style interval files for occupancy-binned non-HOT regions
- BED-style interval files for non-HOT CpG islands

## Outputs

- Per-region mean methylation values per cell type and region set
- methylation_summary.csv with medians and IQRs per region set

## How to apply

Retrieve Roadmap Epigenomics WGBS methylation profiles for all available cell types and load HOT regions, occupancy-binned non-HOT regions, and non-HOT CpG islands as BED-style interval files. For each cell type, intersect CpG-level methylation calls with each region set using interval-overlap tooling (bedtools or genomation) and compute the mean methylation value per region. Aggregate per-region means per region set, then compute the median and interquartile range (IQR) across cell types for each set. The rationale, grounded in the HOT-or-not study, is to confirm that HOT regions are consistently hypo-methylated across cell types while non-HOT CpG islands show higher methylation variation; write medians and IQRs per region set to methylation_summary.csv.

## Related tools

- **bedtools** (Intersects CpG-level methylation calls with each region set of intervals for per-region mean methylation computation)
- **genomation** (Alternative interval-overlap tooling for computing per-region methylation summaries over BED interval sets)

## Examples

```
bedtools intersect -a CpG_methylation.bed -b HOT_regions.bed -wa -u > HOT_methylation_hits.bed
```

## Evaluation signals

- The computed IQR of non-HOT CpG-island methylation is approximately 3-4 times the IQR of HOT-region methylation
- HOT-region median methylation is low (hypo-methylated) relative to non-HOT region sets
- The summary contains median and IQR values for all three region sets (HOT, occupancy-binned non-HOT, non-HOT CpG islands)
- methylation_summary.csv is produced with the medians and IQRs per region set
- Every cell type's methylation profile has been intersected with each region set before aggregation

## Limitations

- HOT regions are likely to be ChIP-seq artifacts, so methylation differences versus non-HOT CpG islands may partly reflect hyper-ChIPable region characteristics rather than distinct regulatory biology
- Validation depends on the paper's reported 3-4x IQR relationship, which requires enough available Roadmap cell types to estimate a stable IQR across cell types
- Mean methylation per region is a coarse aggregate; region-level heterogeneity in methylation is not captured

## Evidence

- [methods] For each cell type, intersect CpG-level methylation calls with each region set and compute the mean methylation value per region using interval-overlap tooling (bedtools/genomation).: [evidence span withheld — anchor status: not_found]
- [results] HOT regions are distinct from non-HOT CpG islands in their methylation dynamics, maintaining low (hypo-methylated) methylation levels consistently across different cell types: [evidence span withheld — anchor status: not_found]
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [discussion] verify that the computed IQR of non-HOT CpG-island methylation is 3-4 times the IQR of HOT-region methylation: [evidence span withheld — anchor status: not_found]
