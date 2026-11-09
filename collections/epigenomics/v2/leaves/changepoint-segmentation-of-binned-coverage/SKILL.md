---
name: changepoint-segmentation-of-binned-coverage
description: Use when you have a normalized, binned scATAC-seq accessibility matrix
  and need to call focal amplifications (e.g., double minutes) at single-cell resolution
  without a matched normal control.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT cannot be used in this setting
- The goal is chromosome arm-level gains and losses rather than focal amplifications,
  or the input is a raw fragment file rather than a normalized binned matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Copy-scAT
  role: R package providing scaleMatrix/scaleMatrixBins, getAlteredSegments/getDoubleMinutes
    functions for scaling, changepoint-based amplification calling, and consensus
    region pooling
  repo: https://github.com/spcdot/CopyscAT
- name: changepoint
  role: R package supplying the mean-variance changepoint segmentation applied to
    each cell's scaled chromosome bin profile
- name: R
  role: Implementation environment; Copy-scAT is developed under R 4.0.0
- name: data.table
  role: R package used for fast table handling of binned matrices and output tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:bdc210fcc02ffcdfc8f5195fce8494b1f3778b6da52410ccdb48397ca9072b86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/spcdot/CopyscAT
  tools:
  - Copy-scAT
  - R
  - changepoint
  - data.table
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# changepoint-segmentation-of-binned-coverage

## Summary

Detects focal amplifications in single cells by running mean-variance changepoint segmentation on per-cell, per-chromosome scaled scATAC-seq bin profiles (Copy-scAT), flagging segments whose scaled signal exceeds a z-score > 5 cutoff relative to the chromosome baseline, then pooling per-cell calls into consensus amplified regions.

## When to use

Use when you have a normalized, binned scATAC-seq accessibility matrix and need to call focal amplifications (e.g., double minutes) at single-cell resolution without a matched normal control.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot be used in this setting
- The goal is chromosome arm-level gains and losses rather than focal amplifications, or the input is a raw fragment file rather than a normalized binned matrix

## Inputs

- Normalized binned single-cell ATAC-seq accessibility matrix (cells x genomic bins)
- Per-cell, per-chromosome scaled bin profiles (median-IQR scaled)
- Parameter values: z-score cutoff 5, gainThreshold, peakCutoff, minSeglen

## Outputs

- Consensus amplified-region table (amplified_regions.tsv) with one row per region and per-cell positive/negative score columns
- Per-cell, per-chromosome changepoint segmentations

## How to apply

First load the normalized binned scATAC matrix and scale it per cell and per chromosome with Copy-scAT's scaleMatrix/scaleMatrixBins functions, which use median-IQR-based scaling so that signal is comparable across cells with differing depth. For each cell and each chromosome, run mean-variance changepoint segmentation from the R changepoint package on the scaled bin profile to partition the chromosome into segments of homogeneous signal. Flag every segment whose scaled signal exceeds a z-score of 5 relative to the chromosome baseline as amplified. Pool per-cell flagged high-signal segments into consensus amplified regions using Copy-scAT's getAlteredSegments/getDoubleMinutes logic, applying the gainThreshold, peakCutoff, and minSeglen parameters to merge overlapping per-cell calls, then score each consensus region per cell by counting positive and negative bins/segments and write the table to amplified_regions.tsv.

## Related tools

- **Copy-scAT** (R package providing scaleMatrix/scaleMatrixBins, getAlteredSegments/getDoubleMinutes functions for scaling, changepoint-based amplification calling, and consensus region pooling) — https://github.com/spcdot/CopyscAT
- **changepoint** (R package supplying the mean-variance changepoint segmentation applied to each cell's scaled chromosome bin profile)
- **R** (Implementation environment; Copy-scAT is developed under R 4.0.0)
- **data.table** (R package used for fast table handling of binned matrices and output tables)

## Examples

```
scaled <- scaleMatrixBins(binnedMatrix); segs <- getDoubleMinutes(scaled, zScoreCut = 5, gainThreshold, peakCutoff, minSeglen)
```

## Evaluation signals

- amplified_regions.tsv exists and parses as a valid table
- The table contains one row per consensus amplified region with separate per-cell positive and negative score columns
- Only segments whose scaled signal exceeds the z-score > 5 cutoff were pooled into consensus regions
- Per-cell scaling used median-IQR scaling per cell and per chromosome before segmentation
- Consensus regions reflect merging of overlapping per-cell calls under gainThreshold/peakCutoff/minSeglen parameters

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants and having no or rare CNVs
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified and requires further study
- The package is in early beta with documentation and testing still in progress; development targets R 4.0.0

## Evidence

- [methods] Copy-scAT calls focal amplifications by running changepoint analysis using mean and variance: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [intro] we demonstrate the effectiveness of Copy-scAT in calling (i) focal amplifications: "we demonstrate the effectiveness of Copy-scAT in calling (i) focal amplifications and (ii) chromosome arm-level gains and losses"
- [readme] CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file.: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file. It is currently in early beta - documentation and testing are still works in"
- [discussion] However, a limitation of Copy-scAT is that it cannot be used with malignancies that are driven primarily by single nucleotide variants: "However, a limitation of Copy-scAT is that it cannot be used with malignancies that are driven primarily by single nucleotide variants and have no or rare CNVs."
