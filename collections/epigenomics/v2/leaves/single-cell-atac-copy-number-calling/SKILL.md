---
name: single-cell-atac-copy-number-calling
description: Use when you have a normalized binned single-cell ATAC-seq matrix and
  need to call CNVs per cell to identify malignant subclones, focal amplifications,
  or arm-level chromosome gains/losses without a matched control file.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT cannot be used in this case per the article's discussion.
- The goal is copy number calling from scRNA-seq rather than scATAC data (other dedicated
  tools exist for that modality).
- The input is already a curated CNV profile or feature table rather than raw/binned
  scATAC accessibility data.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Copy-scAT
  role: 'Core R package performing CNV inference from scATAC-seq data: cell filtering,
    scaling, changepoint-based focal amplification calling, and CNV-based cell annotation/clustering'
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment for Copy-scAT; README specifies installation within an
    R 4.0.0 environment
- name: changepoint
  role: R package providing mean-variance changepoint segmentation used to partition
    each cell's per-chromosome scaled bin profiles for focal amplification detection
- name: process_fragment_file.py
  role: Python script from the CopyscAT repository that generates fragment/bin matrices
    from scATAC fragment files as the pipeline's input matrix
  repo: github.com/spcdot/CopyscAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:0a9ab49eae0d563cce08fecf2a707bee4ae6ec2b5d48b59efaf6b156a3d932da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - changepoint
  - process_fragment_file.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-atac-copy-number-calling

## Summary

Infers copy number variants (CNVs) — including focal amplifications and chromosome arm-level gains/losses — from single-cell ATAC-seq data using the Copy-scAT R package, enabling discrimination of malignant from nonmalignant cells and subclone deconvolution in cancer.

## When to use

Use when you have a normalized binned single-cell ATAC-seq matrix and need to call CNVs per cell to identify malignant subclones, focal amplifications, or arm-level chromosome gains/losses without a matched control file.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT cannot be used in this case per the article's discussion.
- The goal is copy number calling from scRNA-seq rather than scATAC data (other dedicated tools exist for that modality).
- The input is already a curated CNV profile or feature table rather than raw/binned scATAC accessibility data.

## Inputs

- Binned, normalized single-cell ATAC-seq count/accessibility matrix (e.g., 500kb or per-chromosome bins generated from fragment files via process_fragment_file.py)
- scATAC fragment file (paired-end fragments BED-like, for fragment matrix generation)
- Optional expected normals profile for baseline correction (annotateCNV4B)

## Outputs

- Per-cell CNV profile matrix (scaled bin-level or chromosome-level copy number states)
- Consensus amplified-region table with per-cell positive and negative bin/segment scores (amplified_regions.tsv)
- Per-cell malignant vs nonmalignant and subclone/CNV-cluster annotations
- Chromosome arm-level gain/loss calls per cell

## How to apply

Start from a fragment-derived binned scATAC matrix, filtering cells with Copy-scAT's filterCells (minimumSegments, minDensity, signalSDcut) and collapsing bins via collapseChrom3N. Scale the matrix per cell and per chromosome using median-IQR-based scaling (scaleMatrix/scaleMatrixBins). For focal amplifications, run mean-variance changepoint segmentation (changepoint package) on each cell's per-chromosome scaled bin profile, flagging segments with z score > 5 as amplified, then pool per-cell calls into consensus amplified regions using getAlteredSegments/getDoubleMinutes logic with gainThreshold, peakCutoff, and minSeglen merging. For arm-level events, cluster cells by CNV profiles with clusterCNV (medianIQR, maxClust, minDiff) and classify cells via annotateCNV3/annotateCNV4/annotateCNV4B (parameters sdCNV, filterResults, minAlteredCells/minAlteredCellProp, expectedNormals). The rationale is that CNV-driven accessibility changes leave coherent signal imprints per cell, allowing per-cell copy number inference and clustering of cells sharing CNVs.

## Related tools

- **Copy-scAT** (Core R package performing CNV inference from scATAC-seq data: cell filtering, scaling, changepoint-based focal amplification calling, and CNV-based cell annotation/clustering) — github.com/spcdot/CopyscAT
- **R** (Runtime environment for Copy-scAT; README specifies installation within an R 4.0.0 environment)
- **changepoint** (R package providing mean-variance changepoint segmentation used to partition each cell's per-chromosome scaled bin profiles for focal amplification detection)
- **process_fragment_file.py** (Python script from the CopyscAT repository that generates fragment/bin matrices from scATAC fragment files as the pipeline's input matrix) — github.com/spcdot/CopyscAT

## Examples

```
Rscript -e 'library(CopyscAT); scaled <- scaleMatrixBins(binnedMatrix); cnv <- clusterCNV(scaled, medianIQR, maxClust, minDiff); annotateCNV4(cnv, saveOutput=TRUE, sdCNV, filterResults, filterRange, minAlteredCells)'
```

## Evaluation signals

- amplified_regions.tsv exists, parses as a table, and contains one row per consensus amplified region with positive and negative per-cell score columns
- Only segments exceeding the z score > 5 cutoff relative to the chromosome baseline were pooled into consensus amplified regions
- Cells sharing a given CNV cluster together in the CNV-clustered scATAC embedding, consistent with the paper's expectation that cells with the same CNV cluster by epigenetic profile
- Malignant vs nonmalignant calls from annotateCNV4/annotateCNV4B satisfy their filter settings (sdCNV, minAlteredCells/minAlteredCellProp) and are concordant with known CNV status
- CNV calls are reproducible with the semi-automated neoplastic vs non-neoplastic distinction and baseline correction workflow from the Mar 2021 tutorial update

## Limitations

- Copy-scAT cannot be used with malignancies that are driven primarily by single nucleotide variants and have no or rare CNVs
- The package was in early beta per the README — documentation and testing are incomplete
- The optimal way to address CNV effects in downstream scATAC analyses is not yet identified; further studies needed
- Whether CNVs prime cells to acquire specific chromatin accessibility states or stabilize chromatin states of genetic subclones remains undetermined
- Inability to deconvolute malignant and nonmalignant populations after scATAC library generation would confound downstream analyses, so filtering/annotation steps are essential

## Evidence

- [results] Copy-scAT calls focal amplifications by running changepoint analysis using mean and variance: [evidence span withheld — anchor status: not_found]
- [abstract] Copy-scAT, an R package that uses single-cell epigenomic data to infer CNVs: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [discussion] Limitation: cannot be used with SNV-driven malignancies with no or rare CNVs: "a limitation of Copy-scAT is that it cannot be used with malignancies that are driven primarily by single nucleotide variants and have no or rare CNVs."
- [readme] README: designed to identify large-scale and local alterations without a control file: "CopyscAT is designed to identify large-scale and local alterations in chromosomes without need for a control file."
- [methods] Methods: annotateCNV4 function signature with filter parameters: "annotateCNV4(cnvResults, saveOutput, maxClust2, outputSuffix, sdCNV, filterResults, filterRange, minAlteredCells)"
- [readme] README: install within R 4.0.0 environment: "Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
