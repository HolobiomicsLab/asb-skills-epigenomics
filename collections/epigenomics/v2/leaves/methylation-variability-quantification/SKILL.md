---
name: methylation-variability-quantification
description: Use when comparing methylation dynamics of a region set (e.g., HOT regions)
  against CpG islands across multiple cell types using WGBS methylation calls and
  BED-style interval files.
when_to_use_negative:
- Input is already an aggregated methylation summary table rather than CpG-level WGBS
  calls plus genomic interval files
- The comparison targets transcription factor occupancy or ChIP-seq peak enrichment
  rather than bisulfite-derived methylation levels
- Only a single cell type is available, so cross-cell-type methylation variation (IQR)
  cannot be computed
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: bedtools
  role: Interval-overlap tooling to intersect CpG-level methylation calls with HOT
    region and non-HOT CpG island interval sets
- name: genomation
  role: R/Bioconductor interval-overlap tooling used as an alternative to bedtools
    for computing per-region methylation summaries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:4b39f8a9f96f0d0afedc4978458f7935038639ed9ab0915b4bae9698a8bf934d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bedtools
  - genomation
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# methylation-variability-quantification

## Summary

Quantifies CpG methylation levels and methylation variation (median and interquartile range across cell types) for HOT regions, occupancy-binned non-HOT regions, and non-HOT CpG islands using Roadmap Epigenomics WGBS data, to show HOT regions are consistently hypo-methylated while non-HOT CpG islands vary more.

## When to use

Use when comparing methylation dynamics of a region set (e.g., HOT regions) against CpG islands across multiple cell types using WGBS methylation calls and BED-style interval files.

## When NOT to use

- Input is already an aggregated methylation summary table rather than CpG-level WGBS calls plus genomic interval files
- The comparison targets transcription factor occupancy or ChIP-seq peak enrichment rather than bisulfite-derived methylation levels
- Only a single cell type is available, so cross-cell-type methylation variation (IQR) cannot be computed

## Inputs

- Roadmap Epigenomics WGBS CpG-level methylation calls for multiple human cell types
- BED-style interval files for HOT regions
- BED-style interval files for occupancy-binned non-HOT regions
- BED-style interval files for non-HOT CpG islands

## Outputs

- Per-region mean methylation values for each region set and cell type
- methylation_summary.csv with median and IQR of methylation per region set

## How to apply

Retrieve Roadmap Epigenomics whole-genome bisulfite sequencing (WGBS) methylation profiles for all available cell types and load HOT regions, occupancy-binned non-HOT regions, and non-HOT CpG islands as BED-style interval files. For each cell type, intersect CpG-level methylation calls with each region set using interval-overlap tooling such as bedtools or genomation (R/Bioconductor), and compute the mean methylation value per region. Aggregate per-region mean methylation per region set and compute the median and interquartile range (IQR) across cell types for each set. The expected relationship, grounded in the HOT-or-not analysis, is that HOT regions maintain low (hypo-methylated) methylation consistently across cell types, while non-HOT CpG islands show higher methylation variation; write medians and IQRs per region set to methylation_summary.csv.

## Related tools

- **bedtools** (Interval-overlap tooling to intersect CpG-level methylation calls with HOT region and non-HOT CpG island interval sets)
- **genomation** (R/Bioconductor interval-overlap tooling used as an alternative to bedtools for computing per-region methylation summaries)

## Examples

```
bedtools intersect -a Roadmap_WGBS_CpG_meth.bed -b HOT_regions.bed -wao > hot_cpg_overlaps.bed
```

## Evaluation signals

- Computed IQR of non-HOT CpG-island methylation is approximately 3-4 times the IQR of HOT-region methylation, matching the paper's reported relationship
- HOT-region median methylation is low (hypo-methylated) relative to non-HOT region sets
- Per-region mean methylation values were computed for all three region sets (HOT, occupancy-binned non-HOT, non-HOT CpG islands) in every cell type
- methylation_summary.csv contains one median and one IQR entry per region set aggregated across cell types

## Limitations

- HOT regions themselves are likely ChIP-seq artifacts ('hyper-ChIPable' regions), so methylation differences may reflect biased chromatin/sequence features (GC/CpG-rich k-mers, R-loops, G-quadruplex DNA) rather than true co-binding of the listed transcription factors
- HOT regions coincide with house-keeping gene promoters, which constrains interpretation: stable hypo-methylation is expected for promoter-associated house-keeping loci across cell types

## Evidence

- [other] HOT regions are distinct from non-HOT CpG islands in their methylation dynamics, maintaining low (hypo-methylated) methylation levels: [evidence span withheld — anchor status: not_found]
- [other] Retrieve the Roadmap Epigenomics whole-genome bisulfite sequencing (WGBS) methylation profiles for the available cell types.: [evidence span withheld — anchor status: not_found]
- [readme] The scripts in this repository have been used for analysis of HOT regions: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [abstract] they coincide with house-keeping gene promoters and consequently associated genes are stably expressed: "Furthermore, they coincide with house-keeping gene promoters and consequently associated genes are stably expressed across multiple cell types."
