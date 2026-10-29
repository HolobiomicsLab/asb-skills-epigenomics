---
name: count-matrix-cpm-normalization
description: Use when a raw barcode-by-bin fragment count matrix from scATAC data
  has already passed cell/bin filtering (e.g., minFrags=5000, zero-density bins removed)
  and must be converted to a comparable normalized read-depth matrix before CNV calling.
when_to_use_negative:
- Input is already a normalized expression/accessibility matrix — cpm re-scaling would
  distort values
- Cancer type is driven primarily by SNVs with no or rare CNVs, so normalized read
  depth yields no usable copy-number signal (Copy-scAT limitation)
- The goal is per-cell clustering on motif activity (e.g., ChromVAR) rather than read-depth-based
  CNV calling
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0203
tools:
- name: Copy-scAT
  role: R package providing initialiseEnvironment, readInputTable, filterCells, and
    normalizeMatrixN for binning, filtering, and cpm-normalizing the fragment matrix
  repo: github.com/spcdot/CopyscAT
- name: edgeR
  role: supplies the cpm normalization applied by normalizeMatrixN to the filtered
    count matrix
- name: R
  role: execution environment (Copy-scAT developed with R 4.0.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:f05b1a333d6174f38cc36e1081b960aee5865a048257aba87c1520a2632b59eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - edgeR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# count-matrix-cpm-normalization

## Summary

Applies edgeR counts-per-million (cpm) normalization to a filtered single-cell ATAC-seq barcode-by-bin coverage matrix within the Copy-scAT pipeline, converting raw fragment pileup counts into a normalized read-depth matrix suitable for downstream CNV inference.

## When to use

Use when a raw barcode-by-bin fragment count matrix from scATAC data has already passed cell/bin filtering (e.g., minFrags=5000, zero-density bins removed) and must be converted to a comparable normalized read-depth matrix before CNV calling.

## When NOT to use

- Input is already a normalized expression/accessibility matrix — cpm re-scaling would distort values
- Cancer type is driven primarily by SNVs with no or rare CNVs, so normalized read depth yields no usable copy-number signal (Copy-scAT limitation)
- The goal is per-cell clustering on motif activity (e.g., ChromVAR) rather than read-depth-based CNV calling

## Inputs

- cellranger-atac fragment file (TSV of barcodes and fragment coordinates), loaded via readInputTable(inputFile, sep)
- barcode-by-bin raw count matrix from 1-Mbp genome-wide binning
- barcode quality threshold (minimumSegments / minFrags, default 5000 fragments per cell)

## Outputs

- filtered barcode-by-bin matrix after cell and zero-density bin removal
- edgeR cpm-normalized (optionally log-normalized) read-depth matrix for downstream Copy-scAT CNV inference

## How to apply

First construct the raw barcode-by-bin count matrix by binning cellranger-atac fragments genome-wide into 1-Mbp bins via Copy-scAT's initialiseEnvironment (binSize=1 Mbp, minFrags=5000) and readInputTable, then retain barcodes passing the 5000-fragment minimum and drop bins where 80% or more of cells have zero counts using filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut). Apply edgeR cpm normalization to the filtered matrix with normalizeMatrixN(inputMatrix, logNorm, maxZero, imputeZeros, ...); the logNorm, maxZero, and imputeZeros options control whether log transformation is applied and how residual zero counts are handled. The rationale is that per-barcode sequencing depth differs substantially across scATAC cells, so cpm scaling is required before relative read depth can be interpreted as copy-number signal. Validate that the output matrix retains the expected barcode-by-bin dimensions and that every retained barcode meets the 5000-fragment threshold.

## Related tools

- **Copy-scAT** (R package providing initialiseEnvironment, readInputTable, filterCells, and normalizeMatrixN for binning, filtering, and cpm-normalizing the fragment matrix) — github.com/spcdot/CopyscAT
- **edgeR** (supplies the cpm normalization applied by normalizeMatrixN to the filtered count matrix)
- **R** (execution environment (Copy-scAT developed with R 4.0.0))

## Examples

```
Rscript -e 'mat <- readInputTable(inputFile, sep); fmat <- filterCells(mat, minimumSegments=5000, minDensity=0.2, signalSDcut=NULL); norm <- normalizeMatrixN(fmat, logNorm=TRUE, maxZero=0.8, imputeZeros=TRUE)'
```

## Evaluation signals

- Output matrix exists and has expected barcode-by-bin dimensions (barcodes = retained cells, bins = 1-Mbp genomic bins)
- Every retained barcode passes the 5000-fragment (minimumSegments) quality threshold
- Bins dropped are those where 80% or more of cells have zero counts (minDensity semantics of filterCells)
- Per-barcode column sums of the cpm-normalized matrix are comparable across cells, confirming depth scaling succeeded
- Normalized matrix feeds correctly into Copy-scAT CNV calling steps (e.g., collapseChrom3N / annotateCNV4) without dimension or NA errors

## Limitations

- Requires R 4.0.0 environment; package is early beta with documentation and testing still works in progress
- Cannot be used with malignancies driven primarily by single nucleotide variants with no or rare CNVs
- Normalization depends on prior filtering quality; cells below the 5000-fragment minimum or very sparse bins must be removed first

## Evidence

- [methods] Copy-scAT filters scATAC fragment data, keeping only barcodes with a minimum number of fragments (default 5000), and generates a pileup of total coverage (reads × read lengths) over bins for each cell.: [evidence span withheld — anchor status: not_found]
- [methods] Apply edgeR cpm normalization to the filtered matrix via normalizeMatrixN(inputMatrix, logNorm, maxZero, imputeZeros, ...) to yield the normalized read-depth matrix.: [evidence span withheld — anchor status: not_found]
- [methods] filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut): "filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut)"
- [abstract] Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells.: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs)"
- [readme] To generate fragment matrices, use the process_fragment_file.py script (type python3 process_fragment_file.py for details regarding input parameters): "To generate fragment matrices, use the process_fragment_file.py script"
