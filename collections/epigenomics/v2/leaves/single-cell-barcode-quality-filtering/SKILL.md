---
name: single-cell-barcode-quality-filtering
description: Use when raw cellranger-atac fragment files must be converted into a
  filtered, binned coverage matrix and low-quality cell barcodes need removal prior
  to single-cell CNV inference.
when_to_use_negative:
- The malignancy is driven primarily by single nucleotide variants with no or rare
  CNVs — Copy-scAT is explicitly limited here
- Input is already a normalized feature matrix or a filtered single-cell object; re-filtering
  fragments would be redundant
- The goal is motif or gene-score analysis (e.g., ChromVAR) rather than CNV inference
  from fragment coverage
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Copy-scAT
  role: R package providing initialiseEnvironment, readInputTable, filterCells, and
    normalizeMatrixN for barcode filtering and binned matrix construction
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment (R 4.0.0 per README) in which the filtering workflow executes
- name: edgeR
  role: Supplies cpm normalization for the filtered barcode-by-bin matrix
- name: process_fragment_file.py
  role: Python helper script from the repository used to generate fragment matrices
    prior to R-side filtering
  repo: github.com/spcdot/CopyscAT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:dbe2eb7e02f09bf95abb24271f6b7fb5388ec8d702cfa7315afee71362c7c4d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - edgeR
  - process_fragment_file.py
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-barcode-quality-filtering

## Summary

Quality-filters single-cell ATAC-seq fragment data by retaining only barcodes with a minimum number of fragments (Copy-scAT default 5000) and dropping sparse bins, before generating a barcode-by-bin coverage matrix for CNV inference.

## When to use

Use when raw cellranger-atac fragment files must be converted into a filtered, binned coverage matrix and low-quality cell barcodes need removal prior to single-cell CNV inference.

## When NOT to use

- The malignancy is driven primarily by single nucleotide variants with no or rare CNVs — Copy-scAT is explicitly limited here
- Input is already a normalized feature matrix or a filtered single-cell object; re-filtering fragments would be redundant
- The goal is motif or gene-score analysis (e.g., ChromVAR) rather than CNV inference from fragment coverage

## Inputs

- cellranger-atac fragment file (tabular fragments.tsv with barcode, chromosome, start, end columns)
- Genome annotation file (genomeFile)
- Cytoband file (cytobandFile)
- CpG island file (cpgFile)
- Filter parameters: minFrags (default 5000), binSize (default 1 Mbp)

## Outputs

- Filtered barcode-by-bin count matrix (1-Mbp bins, barcodes >= 5000 fragments)
- edgeR cpm-normalized read-depth matrix (via normalizeMatrixN, with zero-imputation options)

## How to apply

Initialize the Copy-scAT environment with initialiseEnvironment(genomeFile, cytobandFile, cpgFile, binSize, minFrags, ...), setting binSize to 1 Mbp and minFrags to 5000 as the barcode quality threshold. Read the cellranger-atac fragment file into R with readInputTable(inputFile, sep) and bin fragments genome-wide into 1-Mbp bins to build the raw barcode-by-bin count matrix. Apply filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut) semantics to retain only barcodes meeting the 5000-fragment minimum and to drop bins where 80% or more of cells have zero counts. Finally, normalize the filtered matrix with edgeR cpm via normalizeMatrixN(inputMatrix, logNorm, maxZero, imputeZeros, ...) to yield the normalized read-depth matrix used downstream for CNV calling.

## Related tools

- **Copy-scAT** (R package providing initialiseEnvironment, readInputTable, filterCells, and normalizeMatrixN for barcode filtering and binned matrix construction) — github.com/spcdot/CopyscAT
- **R** (Runtime environment (R 4.0.0 per README) in which the filtering workflow executes)
- **edgeR** (Supplies cpm normalization for the filtered barcode-by-bin matrix)
- **process_fragment_file.py** (Python helper script from the repository used to generate fragment matrices prior to R-side filtering) — github.com/spcdot/CopyscAT

## Examples

```
R> filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut)  # retain barcodes >= 5000 fragments, drop bins with >= 80% zero counts
```

## Evaluation signals

- Every retained barcode in the output matrix meets the 5000-fragment (minFrags) minimum
- Output matrix has the expected barcode-by-bin dimensions for 1-Mbp genome-wide bins
- Bins where 80% or more of cells have zero counts have been dropped from the matrix
- The normalized matrix reflects edgeR cpm scaling (with zero-imputation per normalizeMatrixN settings) and is non-empty and finite-valued
- The raw pre-filter count matrix can be reproduced from the fragment file binning, confirming lossless read-through of the input table

## Limitations

- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants and having no or rare CNVs
- The tool is early beta per the README — documentation and testing are still works in progress; requires R 4.0.0 environment
- Inability to deconvolute malignant and nonmalignant populations after library generation can confound downstream interpretation if barcodes are not filtered adequately

## Evidence

- [methods] filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut): "filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut)"
- [readme] To generate fragment matrices, use the process_fragment_file.py script (type python3 process_fragment_file.py for details regarding input parameters): "To generate fragment matrices, use the process_fragment_file.py script (type python3 process_fragment_file.py for details regarding input parameters)"
- [abstract] an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells: "an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells"
- [readme] It is currently in early beta - documentation and testing are still works in progress. Development is ongoing with R 4.0.0: "It is currently in early beta - documentation and testing are still works in progress. Development is ongoing with R 4.0.0"
