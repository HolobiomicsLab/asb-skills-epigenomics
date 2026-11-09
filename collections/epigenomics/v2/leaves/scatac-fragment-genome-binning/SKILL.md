---
name: scatac-fragment-genome-binning
description: Use when raw scATAC-seq fragment files (e.g., cellranger-atac output)
  must be converted into a per-cell genomic bin count matrix prior to CNV or copy
  number inference with Copy-scAT.
when_to_use_negative:
- Input is already a binned or normalized count matrix rather than a raw fragment
  file
- Malignancy is driven primarily by SNVs with no or rare CNVs, so fragment pileup-based
  CNV inference provides little signal
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Copy-scAT
  role: R package providing initialiseEnvironment, readInputTable, filterCells, and
    normalizeMatrixN for fragment pileup, binning, cell filtering, and normalization
  repo: github.com/spcdot/CopyscAT
- name: R
  role: Runtime environment for the Copy-scAT package; development is ongoing with
    R 4.0.0
- name: edgeR
  role: Supplies cpm normalization used by normalizeMatrixN to produce the normalized
    read-depth matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:3b195aa22db5bffcd079badf8f6fedf9aeeff816faf5867ecde053e5e9af769d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - R
  - edgeR
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# scatac-fragment-genome-binning

## Summary

Converts raw cellranger-atac fragment files into a filtered, edgeR-normalized barcode-by-bin read-depth matrix (1-Mbp bins, ≥5000 fragments per barcode) as the first stage of Copy-scAT's CNV inference workflow.

## When to use

Use when raw scATAC-seq fragment files (e.g., cellranger-atac output) must be converted into a per-cell genomic bin count matrix prior to CNV or copy number inference with Copy-scAT.

## When NOT to use

- Input is already a binned or normalized count matrix rather than a raw fragment file
- Malignancy is driven primarily by SNVs with no or rare CNVs, so fragment pileup-based CNV inference provides little signal

## Inputs

- cellranger-atac fragment file (tab-separated fragment positions per barcode)
- Genome annotation file (genomeFile)
- Cytoband file (cytobandFile)
- CpG island file (cpgFile)

## Outputs

- Filtered barcode-by-bin count matrix (1-Mbp bins, barcodes with ≥5000 fragments)
- Normalized read-depth matrix (edgeR cpm, log-normalized, zeros imputed)

## How to apply

Initialize the Copy-scAT environment with initialiseEnvironment(genomeFile, cytobandFile, cpgFile, binSize, minFrags, ...) using binSize=1 Mbp and minFrags=5000. Load the fragment file into R with readInputTable(inputFile, sep), then bin fragments genome-wide into the 1-Mbp bins to build the raw barcode-by-bin count matrix. Retain only barcodes meeting the 5000-fragment threshold via filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut), and drop bins where 80% or more of cells have zero counts. Finally normalize the filtered matrix with edgeR cpm via normalizeMatrixN(inputMatrix, logNorm, maxZero, imputeZeros, ...), imputing zeros as configured, to yield the normalized read-depth matrix for downstream CNV calling.

## Related tools

- **Copy-scAT** (R package providing initialiseEnvironment, readInputTable, filterCells, and normalizeMatrixN for fragment pileup, binning, cell filtering, and normalization) — github.com/spcdot/CopyscAT
- **R** (Runtime environment for the Copy-scAT package; development is ongoing with R 4.0.0)
- **edgeR** (Supplies cpm normalization used by normalizeMatrixN to produce the normalized read-depth matrix)

## Examples

```
initialiseEnvironment(genomeFile, cytobandFile, cpgFile, binSize=1000000, minFrags=5000); inputMatrix <- readInputTable(inputFile, sep); normMatrix <- normalizeMatrixN(filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut))
```

## Evaluation signals

- Output matrix exists and has expected barcode-by-bin dimensions for 1-Mbp bins
- Every retained barcode meets the 5000-fragment minimum (minFrags)
- Bins where 80% or more of cells have zero counts are dropped
- Normalized matrix values are consistent with edgeR cpm log-normalization and zero imputation settings

## Limitations

- Copy-scAT is in early beta — documentation and testing are still works in progress
- Must be installed within an R 4.0.0 environment
- Cannot be used with malignancies driven primarily by single nucleotide variants and having no or rare CNVs

## Evidence

- [methods] filterCells: [evidence span withheld — anchor status: not_found]
- [methods] normalizeMatrixN: [evidence span withheld — anchor status: not_found]
- [methods] readInputTable: [evidence span withheld — anchor status: not_found]
- [readme] fragment-matrix generation: "To generate fragment matrices, use the process_fragment_file.py script (type python3 process_fragment_file.py for details regarding input parameters)"
- [readme] R environment: "Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
