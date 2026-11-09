---
name: consensus-focal-amplification-pooling
description: Use when calling focal amplifications at single-cell resolution from
  a normalized, binned scATAC-seq matrix to discriminate malignant from nonmalignant
  cells based on CNVs.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  copy number variants — Copy-scAT cannot be used for such cancers
- The goal is chromosome arm-level gain/loss inference rather than focal amplification
  detection — use Copy-scAT's arm-level CNV module instead
- Input is raw fragment files not yet converted into a normalized binned matrix —
  first generate fragment matrices with process_fragment_file.py
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3673
tools:
- name: Copy-scAT
  role: R package supplying scaleMatrix/scaleMatrixBins scaling and getAlteredSegments/getDoubleMinutes
    consensus amplification calling
  repo: https://github.com/spcdot/CopyscAT
- name: changepoint
  role: R package providing mean-variance changepoint segmentation of scaled bin profiles
    per cell and chromosome
- name: data.table
  role: Fast tabular handling of binned matrices and the amplified-region output table
- name: R
  role: Runtime environment; Copy-scAT is developed under R 4.0.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:764b07e48ee07ec6352c8121c73d4fc6d1cc8b9cdd740f756fb105d76c94a744
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# consensus-focal-amplification-pooling

## Summary

Detects focal amplifications from single-cell ATAC-seq data using Copy-scAT's mean-variance changepoint segmentation, then pools per-cell high-signal segments into consensus amplified regions scored per cell.

## When to use

Use when calling focal amplifications at single-cell resolution from a normalized, binned scATAC-seq matrix to discriminate malignant from nonmalignant cells based on CNVs.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare copy number variants — Copy-scAT cannot be used for such cancers
- The goal is chromosome arm-level gain/loss inference rather than focal amplification detection — use Copy-scAT's arm-level CNV module instead
- Input is raw fragment files not yet converted into a normalized binned matrix — first generate fragment matrices with process_fragment_file.py

## Inputs

- Normalized binned single-cell ATAC-seq count matrix (per cell, per genomic bin)
- Chromosome-level scaled bin profiles per cell (median-IQR scaled)
- Copy-scAT parameters: z-score cutoff (>5), gainThreshold, peakCutoff, minSeglen

## Outputs

- Consensus amplified-region table (amplified_regions.tsv) with one row per amplified region
- Per-cell positive and negative bin/segment scores for each consensus amplified region

## How to apply

Load the normalized binned scATAC matrix and scale it per cell and per chromosome with Copy-scAT's scaleMatrix/scaleMatrixBins functions using median-IQR-based scaling. For each cell and chromosome, run mean-variance changepoint segmentation from the R 'changepoint' package on the scaled bin profiles to partition each chromosome into segments. Flag as amplified every segment whose scaled signal exceeds a z-score of 5 relative to the chromosome baseline. Pool per-cell flagged segments into consensus amplified regions with getAlteredSegments/getDoubleMinutes logic, applying gainThreshold/peakCutoff and minSeglen parameters to merge overlapping calls, then score each consensus region per cell by counting positive and negative bins/segments. Rationale: focal amplifications appear as localized segments of abnormally high accessibility signal within individual cells, so pooling across cells yields robust consensus regions that define malignant populations.

## Related tools

- **Copy-scAT** (R package supplying scaleMatrix/scaleMatrixBins scaling and getAlteredSegments/getDoubleMinutes consensus amplification calling) — https://github.com/spcdot/CopyscAT
- **changepoint** (R package providing mean-variance changepoint segmentation of scaled bin profiles per cell and chromosome)
- **data.table** (Fast tabular handling of binned matrices and the amplified-region output table)
- **R** (Runtime environment; Copy-scAT is developed under R 4.0.0)

## Examples

```
Rscript -e 'library(CopyscAT); scaled <- scaleMatrixBins(binnedMatrix); segs <- runSegmentation(scaled, zCut=5); amp <- getAlteredSegments(segs, gainThreshold, peakCutoff, minSeglen); write.table(amp, "amplified_regions.tsv", sep="\t", quote=FALSE)'
```

## Evaluation signals

- amplified_regions.tsv exists and parses as a well-formed table
- Table contains one row per consensus amplified region plus positive and negative per-cell score columns
- Only segments whose scaled signal exceeded the z-score > 5 chromosome baseline cutoff were pooled into consensus regions
- Per-cell changepoint segmentation runs completed for every cell and chromosome of the scaled bin matrix
- Consensus regions reproduce known focal amplifications in the cancer type under study (e.g., calls validated against arm-level and focal CNV ground truth in the paper's glioblastoma datasets)

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants that have no or rare CNVs
- The tool is early beta — documentation and testing are still works in progress (README, Mar 2021)
- Requires development within an R 4.0.0 environment
- No control file is used, so baseline correction relies on per-cell/per-chromosome median-IQR scaling assumptions

## Evidence

- [methods] Focal amplification detection via mean-variance changepoint segmentation: [evidence span withheld — anchor status: not_found]
- [methods] Z-score > 5 amplification cutoff: [evidence span withheld — anchor status: not_found]
- [abstract] Copy-scAT purpose: single-cell CNV inference from scATAC data: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [intro] No prior dedicated scATAC CNV tool (motivation): "For instance, unlike scRNA-seq, now, no dedicated tool exists to call CNVs using scATAC data."
- [readme] README: large-scale and local alterations without control file; R 4.0.0 requirement: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file. It is currently in early beta - documentation and testing are still works in"
