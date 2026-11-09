---
name: arm-level-copy-number-calling
description: Use when you have a normalized, filtered binned scATAC-seq accessibility
  matrix and need per-cell chromosome arm-level gains/losses (e.g., to separate malignant
  from nonmalignant cells or define genetic subclones).
when_to_use_negative:
- Tumors driven primarily by single nucleotide variants with no or rare CNVs — the
  article notes Copy-scAT cannot be used in this setting since arm-level calling depends
  on CNV signals
- Task requires focal amplification calling rather than arm-level gains and losses
  — a separate focal-level workflow is used for focal amplifications
- Input is a raw fragment file rather than a filtered, normalized binned accessibility
  matrix — run process_fragment_file.py and filterCells/normalizeMatrixN first
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3517
tools:
- name: Copy-scAT
  role: R package providing collapseChrom3N, makeFakeCells, clusterCNV, and annotation
    functions for arm-level CNV inference from scATAC-seq data
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Language environment (development on R 4.0.0) in which the Copy-scAT arm-level
    CNV workflow is executed
- name: mclust
  role: Gaussian mixture model fitting engine used by clusterCNV to decompose per-cell
    arm-level accessibility distributions into copy-number components
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:284502f202579010655714963234fdbe33772096a6d5da6afbaab2b20f590129
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# arm-level-copy-number-calling

## Summary

Calls chromosome arm-level copy number alterations per cell from binned scATAC-seq accessibility data using Copy-scAT: bins are collapsed to chromosome arms (CpG-island-length weighted, trimmed-mean summarized, log transformed), a modeled pseudodiploid control is generated, and Gaussian mixture models (Mclust) fit per cell are compared to that control to assign arm-level CNV states.

## When to use

Use when you have a normalized, filtered binned scATAC-seq accessibility matrix and need per-cell chromosome arm-level gains/losses (e.g., to separate malignant from nonmalignant cells or define genetic subclones).

## When NOT to use

- Tumors driven primarily by single nucleotide variants with no or rare CNVs — the article notes Copy-scAT cannot be used in this setting since arm-level calling depends on CNV signals
- Task requires focal amplification calling rather than arm-level gains and losses — a separate focal-level workflow is used for focal amplifications
- Input is a raw fragment file rather than a filtered, normalized binned accessibility matrix — run process_fragment_file.py and filterCells/normalizeMatrixN first

## Inputs

- normalized, filtered binned scATAC-seq accessibility matrix from Copy-scAT preprocessing (filterCells/normalizeMatrixN)
- copy-scAT function parameters: maxClust, minDiff, sd_fact, minCPG, logTrans/logBase, binExpand, cutAverage quantile range (50th-80th)
- chromosome arm annotation / CpG-island length reference used by collapseChrom3N

## Outputs

- per-cell chromosome arm-level CNV call matrix (e.g., arm_level_cnv_calls.csv)
- per-cell Mclust Gaussian mixture component assignments with component means, z-score distances, and mixing proportions
- modeled pseudodiploid reference distribution (makeFakeCells) for the sample

## How to apply

Start from the normalized, filtered binned matrix produced by upstream Copy-scAT preprocessing (filterCells/normalizeMatrixN). Collapse bins to chromosome arms with collapseChrom3N, weighting bins by CpG-island length (minCPG) and summarizing each arm with the cutAverage trimmed mean over the 50th-80th quantile of bin values, applying log transformation (logTrans/logBase) and bin expansion (binExpand) as configured. Generate a modeled pseudodiploid reference population with makeFakeCells using sd_fact scaling so diploid cells have a control distribution to compare against. Run clusterCNV(initialResultList, medianIQR, maxClust, minDiff) to fit Gaussian mixture models (Mclust) per cell, allowing up to maxClust components with minDiff separation between component means. Assign arm-level copy number states by comparing each component mean to the pseudodiploid distribution, filtering components by z-score distance and mixing proportion, then write the per-cell arm-level CNV call matrix (e.g., arm_level_cnv_calls.csv).

## Related tools

- **Copy-scAT** (R package providing collapseChrom3N, makeFakeCells, clusterCNV, and annotation functions for arm-level CNV inference from scATAC-seq data) — github.com/spcdot/CopyscAT
- **R** (Language environment (development on R 4.0.0) in which the Copy-scAT arm-level CNV workflow is executed)
- **mclust** (Gaussian mixture model fitting engine used by clusterCNV to decompose per-cell arm-level accessibility distributions into copy-number components)

## Examples

```
clusterCNV(initialResultList, medianIQR, maxClust, minDiff)
```

## Evaluation signals

- Output CSV contains one CNV state call per cell per chromosome arm, matching the cell set of the input filtered binned matrix
- Mclust fits respect the minDiff separation between component means and the maxClust component ceiling; components failing z-score distance or mixing-proportion filters are excluded from calls
- Cells within the same arm-level gain/loss call are expected to cluster together in downstream scATAC analyses, consistent with the article's observation that cells sharing a CNV tend to cluster together
- Called arm-level states agree with the modeled pseudodiploid control distribution: diploid arms center on the makeFakeCells reference, gains fall above and losses below it
- Known large-scale alterations used to demonstrate Copy-scAT (chromosome arm-level gains and losses) are recovered in the output matrix

## Limitations

- Copy-scAT cannot be used with malignancies that are driven primarily by single nucleotide variants and have no or rare CNVs
- The tool is in early beta per the README — documentation and testing are still works in progress
- It is designed to work without a control file, relying instead on a modeled pseudodiploid reference, which may be less accurate than matched controls in some samples
- Requires an R 4.0.0 environment per the README installation instructions

## Evidence

- [abstract] Copy-scAT enables studies of subclonal chromatin dynamics in complex tumors like glioblastoma: "an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells"
- [intro] arm-level calling objective: "we demonstrate the effectiveness of Copy-scAT in calling (i) focal amplifications and (ii) chromosome arm-level gains and losses"
- [methods] arm collapsing step: "collapseChrom3N(inputMatrix, minimumSegments, summaryFunction, logTrans, binExpand, minimumChromValue, tssEnrich, logBase, minCPG, powVal)"
- [methods] Gaussian mixture fitting step: "clusterCNV(initialResultList, medianIQR, maxClust, minDiff)"
- [readme] no-control-file design: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
- [discussion] scope limitation: "However, a limitation of Copy-scAT is that it cannot be used with malignancies that are driven primarily by single nucleotide variants and have no or rare CNVs."
