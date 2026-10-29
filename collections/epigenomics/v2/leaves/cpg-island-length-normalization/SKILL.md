---
name: cpg-island-length-normalization
description: 'Use when preparing binned scATAC accessibility data for arm-level copy-number
  inference: binned matrix rows must be reduced to chromosome arms with length-aware
  weighting prior to pseudodiploid reference generation and clusterCNV Gaussian mixture
  fitting.'
when_to_use_negative:
- Input is raw fragments rather than a normalized, cell-filtered binned matrix — run
  Copy-scAT preprocessing (filterCells/normalizeMatrixN) first
- The goal is focal amplification calling rather than chromosome arm-level gains and
  losses — the arm-level collapse discards focal resolution
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT cannot be used in that setting
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Copy-scAT
  role: R package providing collapseChrom3N, cutAverage, filterCells, normalizeMatrixN,
    makeFakeCells and clusterCNV for the arm-level collapse and downstream CNV inference
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment (Copy-scAT is developed under R 4.0.0)
  repo: github.com/spcdot/CopyscAT
- name: mclust
  role: Gaussian mixture model fitting used by clusterCNV on the collapsed arm-level
    results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:15876ac364674a4dfccc20e45d35ab12a72279c6bc6136656e134310681960e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# CpG-island-length-normalized arm-level collapsing of scATAC accessibility

## Summary

Collapses a filtered, binned single-cell ATAC-seq accessibility matrix to chromosome-arm level using Copy-scAT's collapseChrom3N, weighting bins by CpG-island length (minCPG) and summarizing each arm with the cutAverage trimmed mean over the 50th-80th quantile, with optional log transformation, so arm-level signals are length-normalized before pseudodiploid comparison and Mclust-based CNV calling.

## When to use

Use when preparing binned scATAC accessibility data for arm-level copy-number inference: binned matrix rows must be reduced to chromosome arms with length-aware weighting prior to pseudodiploid reference generation and clusterCNV Gaussian mixture fitting.

## When NOT to use

- Input is raw fragments rather than a normalized, cell-filtered binned matrix — run Copy-scAT preprocessing (filterCells/normalizeMatrixN) first
- The goal is focal amplification calling rather than chromosome arm-level gains and losses — the arm-level collapse discards focal resolution
- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot be used in that setting

## Inputs

- Normalized, filtered binned scATAC-seq accessibility matrix (output of Copy-scAT filterCells/normalizeMatrixN)
- Configuration parameters: minimumSegments, summaryFunction, logTrans, binExpand, minimumChromValue, tssEnrich, logBase, minCPG, powVal

## Outputs

- Collapsed chromosome-arm-level accessibility matrix suitable for pseudodiploid reference generation and clusterCNV Mclust fitting
- Downstream per-cell arm-level CNV call matrix (arm_level_cnv_calls.csv)

## How to apply

Load the normalized, filtered binned scATAC accessibility matrix produced by Copy-scAT's upstream filterCells/normalizeMatrixN preprocessing. Call collapseChrom3N with minimumSegments, the summaryFunction set to cutAverage (a trimmed mean taken over the 50th-80th quantile of bin values), the minCPG CpG-island length weighting, plus logTrans/logBase log transformation, binExpand expansion, minimumChromValue and tssEnrich thresholds as configured. The rationale is that CpG-island content biases accessibility signal, so weighting by CpG-island length and trimming the per-arm distribution prevents GC/island-rich bins from dominating arm-level estimates. The collapsed arm-level matrix feeds makeFakeCells (sd_fact-scaled pseudodiploid reference) and clusterCNV (Mclust, maxClust components, minDiff mean separation) for per-cell arm-level CNV calls.

## Related tools

- **Copy-scAT** (R package providing collapseChrom3N, cutAverage, filterCells, normalizeMatrixN, makeFakeCells and clusterCNV for the arm-level collapse and downstream CNV inference) — github.com/spcdot/CopyscAT
- **R** (Runtime environment (Copy-scAT is developed under R 4.0.0)) — github.com/spcdot/CopyscAT
- **mclust** (Gaussian mixture model fitting used by clusterCNV on the collapsed arm-level results)

## Examples

```
collapseChrom3N(inputMatrix, minimumSegments, summaryFunction=cutAverage, logTrans=TRUE, logBase=2, minCPG=minCPG, powVal=powVal)
```

## Evaluation signals

- Each chromosome arm is represented by a single summary value per cell, computed as the cutAverage trimmed mean over the 50th-80th quantile of its bins rather than a plain mean
- Arm-level values differ from unweighted bin averaging: bins are weighted by CpG-island length (minCPG) as configured
- The collapsed matrix is compatible with clusterCNV inputs (initialResultList, medianIQR, maxClust, minDiff) and yields a Gaussian decomposition per cell
- Component means from Mclust can be compared against the makeFakeCells pseudodiploid distribution with z-score filtering and mixing-proportion cutoffs to assign arm states
- A per-cell arm-level CNV call matrix can be written to arm_level_cnv_calls.csv

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants that have no or rare CNVs
- Copy-scAT was in early beta per the README: documentation and testing were still works in progress
- Requires an R 4.0.0 environment as stated in the README development notes
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified; further studies are needed

## Evidence

- [methods] collapseChrom3N input: "collapseChrom3N(inputMatrix, minimumSegments, summaryFunction, logTrans, binExpand, minimumChromValue, tssEnrich, logBase, minCPG, powVal)"
- [methods] cutAverage summary function: "cutAverage(inputVector)"
- [methods] clusterCNV on collapsed arm-level results: "clusterCNV(initialResultList, medianIQR, maxClust, minDiff)"
- [abstract] arm-level CNV inference using scATAC data: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [readme] README scope statement: "Copy number variant inference with single-cell ATAC seq"
