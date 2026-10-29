---
name: z-score-amplification-thresholding
description: Use when you have a normalized binned scATAC-seq matrix per cell and
  need to call focal amplifications (e.g., double minutes) at single-cell resolution
  without a matched control sample.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT explicitly cannot be used in that setting
- The goal is arm-level (whole-chromosome) gain/loss annotation rather than focal
  amplification detection — use the annotateCNV3/annotateCNV4 branch instead
- A matched control/baseline file already exists and is required by the analysis design
  — CopyscAT is designed to work without a control file, so control-based workflows
  are out of scope
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Copy-scAT
  role: R package supplying scaleMatrix/scaleMatrixBins normalization, getAlteredSegments/getDoubleMinutes
    consensus pooling, and per-cell gain/loss scoring
  repo: https://github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment (developed under R 4.0.0); hosts the changepoint mean-variance
    segmentation called per cell and chromosome
  repo: https://github.com/spcdot/CopyscAT
- name: changepoint
  role: R package used for mean-variance changepoint segmentation of scaled bin profiles
    along each chromosome in each single cell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:204c93df4759a18a57780304e665a2b529c78ab34ceb9d702a4199e8eea5a6bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/spcdot/CopyscAT
  tools:
  - Copy-scAT
  - R
  - changepoint
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# z-score-amplification-thresholding

## Summary

Detects focal amplifications in single-cell ATAC-seq data by scaling binned accessibility per cell/chromosome (median-IQR), segmenting with mean-variance changepoint analysis, and flagging segments whose scaled signal exceeds a z-score > 5, then pooling per-cell calls into consensus amplified regions.

## When to use

Use when you have a normalized binned scATAC-seq matrix per cell and need to call focal amplifications (e.g., double minutes) at single-cell resolution without a matched control sample.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT explicitly cannot be used in that setting
- The goal is arm-level (whole-chromosome) gain/loss annotation rather than focal amplification detection — use the annotateCNV3/annotateCNV4 branch instead
- A matched control/baseline file already exists and is required by the analysis design — CopyscAT is designed to work without a control file, so control-based workflows are out of scope

## Inputs

- Normalized binned single-cell ATAC-seq accessibility matrix (cells x genomic bins)
- Fragment file / fragment matrix processed via process_fragment_file.py
- Parameters: z-score cutoff (default 5), gainThreshold, peakCutoff, minSeglen

## Outputs

- amplified_regions.tsv — one row per consensus amplified region with per-cell positive and negative score columns
- Per-cell high-signal (z > 5) segment calls per chromosome

## How to apply

First normalize the binned scATAC matrix and scale it per cell and per chromosome with Copy-scAT's scaleMatrix/scaleMatrixBins (median-IQR-based scaling) so signal is comparable across cells and chromosomes. For each cell and chromosome, run mean-variance changepoint segmentation (from the R 'changepoint' package) on the scaled bin profile to partition the chromosome into segments. Flag a segment as amplified when its scaled signal exceeds z > 5 relative to the chromosome baseline — this z-score cutoff is the core decision threshold. Pool per-cell flagged segments into consensus amplified regions via Copy-scAT's getAlteredSegments/getDoubleMinutes logic, tuning gainThreshold, peakCutoff, and minSeglen to merge overlapping calls. Finally, score each consensus region per cell by counting positive and negative bins/segments, and write amplified_regions.tsv with per-cell positive/negative score columns.

## Related tools

- **Copy-scAT** (R package supplying scaleMatrix/scaleMatrixBins normalization, getAlteredSegments/getDoubleMinutes consensus pooling, and per-cell gain/loss scoring) — https://github.com/spcdot/CopyscAT
- **R** (Runtime environment (developed under R 4.0.0); hosts the changepoint mean-variance segmentation called per cell and chromosome) — https://github.com/spcdot/CopyscAT
- **changepoint** (R package used for mean-variance changepoint segmentation of scaled bin profiles along each chromosome in each single cell)

## Examples

```
ampSegs <- CopyscAT::getAlteredSegments(scaledBins, gainThreshold = 5, minSeglen = 3); write.table(ampSegs, "amplified_regions.tsv", sep = "\t")
```

## Evaluation signals

- amplified_regions.tsv exists and parses as a table with one row per consensus amplified region and positive/negative per-cell score columns
- Only segments whose scaled signal passed the z-score > 5 chromosome-baseline cutoff were pooled into consensus regions
- Per-cell positive score for a consensus amplified region is nonzero only in cells where the corresponding z > 5 segment call was made
- Scaled per-chromosome profiles are median-IQR normalized (via scaleMatrix/scaleMatrixBins) before segmentation — unscaled bin counts should never enter the changepoint step
- Consensus region merging respects the chosen gainThreshold/peakCutoff and minSeglen parameters (overlapping per-cell calls collapse to one row)

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants that have no or rare CNVs
- Whether CNVs prime cells to acquire specific chromatin states or stabilize subclone states is undetermined, so amplified-region calls should not be over-interpreted causally
- The package is early beta with documentation and testing still in progress (per README); the optimal way to address CNV effects in downstream scATAC analyses is not yet identified

## Evidence

- [abstract] an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells: "an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells"
- [intro] Focal amplification effectiveness claim: "we demonstrate the effectiveness of Copy-scAT in calling (i) focal amplifications and (ii) chromosome arm-level gains and losses"
- [readme] No-control-file design: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
