---
name: gaussian-mixture-clustering-cnv
description: Use when binned scATAC-seq accessibility data must be decomposed into
  per-cell copy-number states using Gaussian mixture modeling, e.g. with Copy-scAT's
  clusterCNV/Mclust after collapseChrom3N arm-level summarization.
when_to_use_negative:
- Malignancies driven primarily by single nucleotide variants with no or rare CNVs
  — Copy-scAT cannot infer subclones without CNVs
- When a matched diploid control sample is available and directly usable — Copy-scAT
  is designed to identify alterations 'without need for a control file' via the modeled
  pseudodiploid instead
- Focal-level CNV detection from raw unprocessed bins — this skill operates on arm-level
  collapsed summaries, not the binned matrix itself
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3674
tools:
- name: Copy-scAT
  role: R package providing clusterCNV, collapseChrom3N, makeFakeCells and annotateCNV
    functions that together implement Mclust-based arm-level CNV calling
  repo: github.com/spcdot/CopyscAT
- name: mclust
  role: Gaussian mixture model fitting engine invoked by clusterCNV to decompose per-cell
    arm-level accessibility distributions
- name: R
  role: Runtime environment; Copy-scAT is an R package developed under R 4.0.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:74a015b08ba554518c681a805f8aa13c8220281cac525b021196ed8fbd337d3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gaussian-mixture-clustering-cnv

## Summary

Fit Gaussian mixture models (mclust) to per-cell chromosome arm-level accessibility summaries within Copy-scAT so that cells partition into copy-number components, enabling arm-level CNV calling and malignant vs nonmalignant discrimination in scATAC-seq data.

## When to use

Use when binned scATAC-seq accessibility data must be decomposed into per-cell copy-number states using Gaussian mixture modeling, e.g. with Copy-scAT's clusterCNV/Mclust after collapseChrom3N arm-level summarization.

## When NOT to use

- Malignancies driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot infer subclones without CNVs
- When a matched diploid control sample is available and directly usable — Copy-scAT is designed to identify alterations 'without need for a control file' via the modeled pseudodiploid instead
- Focal-level CNV detection from raw unprocessed bins — this skill operates on arm-level collapsed summaries, not the binned matrix itself

## Inputs

- Normalized, filtered binned scATAC-seq accessibility matrix from Copy-scAT preprocessing (filterCells/normalizeMatrixN)
- Collapsed chromosome-arm-level matrix from collapseChrom3N (cutAverage trimmed mean over 50th-80th quantile, log-transformed)
- Modeled pseudodiploid reference population from makeFakeCells (sd_fact scaling)
- clusterCNV parameters: medianIQR, maxClust, minDiff

## Outputs

- Per-cell Gaussian component assignments (Mclust component means, mixing proportions) for each chromosome arm
- Per-cell arm-level copy number state matrix (comparisons against pseudodiploid distribution)
- arm_level_cnv_calls.csv (per-cell arm-level CNV call matrix)
- CNV result objects for downstream annotateCNV3/annotateCNV4/annotateCNV4B annotation

## How to apply

First collapse the normalized, filtered binned scATAC matrix to chromosome arms with collapseChrom3N, weighting bins by CpG-island length (minCPG) and summarizing each arm with the cutAverage trimmed mean over the 50th-80th quantile, applying log transformation (logTrans/logBase) and binExpand as configured. Generate a modeled pseudodiploid control population with makeFakeCells (sd_fact scaling) so diploid cells have an expected distribution to compare against. Then run clusterCNV(initialResultList, medianIQR, maxClust, minDiff), which fits a Gaussian mixture model (Mclust) per cell allowing up to maxClust components with minDiff separation between component means. Assign arm-level copy number states per cell by comparing each component mean to the pseudodiploid distribution, filtering components by z-score distance and mixing proportion (downstream annotateCNV4/annotateCNV4B with sdCNV, filterResults, filterRange, minAlteredCells/minAlteredCellProp). Rationale: cells sharing a CNV tend to cluster together in scATAC data because genetics imparts specific accessibility profiles, so Gaussian components capture subclonal copy-number structure without requiring matched control samples.

## Related tools

- **Copy-scAT** (R package providing clusterCNV, collapseChrom3N, makeFakeCells and annotateCNV functions that together implement Mclust-based arm-level CNV calling) — github.com/spcdot/CopyscAT
- **mclust** (Gaussian mixture model fitting engine invoked by clusterCNV to decompose per-cell arm-level accessibility distributions)
- **R** (Runtime environment; Copy-scAT is an R package developed under R 4.0.0)

## Examples

```
armResults <- collapseChrom3N(normMatrix, minCPG=minCPG, logTrans=TRUE); fake <- makeFakeCells(armResults, sd_fact=sd_fact); cnv <- clusterCNV(armResults, medianIQR, maxClust, minDiff)
```

## Evaluation signals

- arm_level_cnv_calls.csv is produced with per-cell, per-arm copy number states and cells are partitioned into multiple Gaussian components where subclones exist
- Component means of the fitted mixtures are separable by at least minDiff and each retained component passes z-score distance and mixing-proportion filtering against the pseudodiploid distribution
- Called gains/losses reproduce known arm-level events (e.g. in glioblastoma) and discriminate malignant from nonmalignant cells as expected
- Downstream annotateCNV4/annotateCNV4B with sdCNV, filterResults, filterRange, and minAlteredCells/minAlteredCellProp thresholds returns annotated CNV calls without excessive spurious components
- Pseudodiploid makeFakeCells control distribution (sd_fact scaling) covers expected diploid cells, so nonmalignant cells cluster near the control mean

## Limitations

- Copy-scAT cannot be used with malignancies that are driven primarily by single nucleotide variants and have no or rare CNVs
- Further studies are needed to identify the optimal way to address the effects of CNVs in downstream analyses of scATAC datasets
- The package is in early beta; documentation and testing are still works in progress, and development targets an R 4.0.0 environment
- Requires an appropriately scaled pseudodiploid control (sd_fact); mis-specification will bias z-score-based component filtering

## Evidence

- [methods] clusterCNV fits Mclust Gaussian mixtures per cell: "clusterCNV(initialResultList, medianIQR, maxClust, minDiff)"
- [methods] collapseChrom3N summarizes bins to chromosome arms with trimmed mean and log transform: "collapseChrom3N(inputMatrix, minimumSegments, summaryFunction, logTrans, binExpand, minimumChromValue, tssEnrich, logBase, minCPG, powVal)"
- [intro] Copy-scAT calls arm-level gains and losses at single-cell level: "we demonstrate the effectiveness of Copy-scAT in calling (i) focal amplifications and (ii) chromosome arm-level gains and losses"
- [intro] Copy-scAT discriminates malignant from nonmalignant cells via CNV presence: "Copy-scAT can therefore discriminate between malignant and nonmalignant cells in scATAC datasets based on the presence or absence, respectively, of CNVs."
- [readme] CopyscAT identifies alterations without a control file: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
