---
name: single-cell-atac-data-processing
description: Use when raw scATAC-seq fragment files need conversion into filtered,
  normalized, binned accessibility matrices as input to CNV calling or subclonal deconvolution
  with Copy-scAT.
when_to_use_negative:
- Malignancy is driven primarily by single nucleotide variants with no or rare CNVs,
  since Copy-scAT cannot infer subclones in that setting
- Input is already a normalized, filtered binned matrix produced by prior Copy-scAT
  preprocessing (filterCells/normalizeMatrixN), in which case proceed directly to
  collapseChrom3N
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Copy-scAT
  role: R package providing filterCells, normalizeMatrixN, collapseChrom3N, cutAverage
    for scATAC preprocessing
  repo: github.com/spcdot/CopyscAT
- name: R
  role: runtime environment (developed and tested under R 4.0.0)
  repo: github.com/spcdot/CopyscAT
- name: fastcluster
  role: dependency of Copy-scAT used during clustering-dependent processing
  repo: github.com/spcdot/CopyscAT
- name: NMF
  role: dependency of Copy-scAT for matrix operations
  repo: github.com/spcdot/CopyscAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1126/sciadv.abg6045
    title: 'Copy-scAT: Deconvoluting single-cell chromatin accessibility of genetic
      subclones in cancer'
  source_content_hash: sha256:a9047e24dd022ff5920e8676e854d8934f55ee2867649e7056fe56e5eda0abe2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Copy-scAT
  - NMF
  - R
  - fastcluster
  - mclust
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-atac-data-processing

## Summary

Preprocess raw single-cell ATAC-seq data into a normalized, filtered binned accessibility matrix suitable for Copy-scAT copy number inference, including cell filtering, matrix normalization, and collapsing of binned signal to chromosome-arm level.

## When to use

Use when raw scATAC-seq fragment files need conversion into filtered, normalized, binned accessibility matrices as input to CNV calling or subclonal deconvolution with Copy-scAT.

## When NOT to use

- Malignancy is driven primarily by single nucleotide variants with no or rare CNVs, since Copy-scAT cannot infer subclones in that setting
- Input is already a normalized, filtered binned matrix produced by prior Copy-scAT preprocessing (filterCells/normalizeMatrixN), in which case proceed directly to collapseChrom3N

## Inputs

- raw scATAC-seq fragment file (e.g., .tsv fragments from Cell Ranger)
- binned scATAC-seq accessibility matrix (bins x cells)
- CpG-island length annotations for bin weighting (minCPG)

## Outputs

- normalized, filtered binned scATAC accessibility matrix
- chromosome-arm-level collapsed accessibility matrix
- per-cell quality-filter flag summary (retained vs removed cells)

## How to apply

Generate fragment matrices from raw scATAC-seq data with the process_fragment_file.py script provided in the spcdot/CopyscAT repository. Load the fragment-derived binned matrix in an R 4.0.0 environment and apply filterCells with minimumSegments, minDensity, and signalSDcut parameters to remove low-quality cells with too few segments or aberrant signal distributions. Normalize the filtered matrix (normalizeMatrixN) and collapse bins to chromosome arms with collapseChrom3N, weighting bins by CpG-island length (minCPG), applying log transformation (logTrans/logBase) and optional bin expansion (binExpand), and summarizing each arm with the cutAverage trimmed mean over the 50th-80th quantile of bin values. The output normalized, filtered binned matrix feeds downstream arm-level CNV inference (clusterCNV/Mclust) against a pseudodiploid reference (makeFakeCells).

## Related tools

- **Copy-scAT** (R package providing filterCells, normalizeMatrixN, collapseChrom3N, cutAverage for scATAC preprocessing) — github.com/spcdot/CopyscAT
- **R** (runtime environment (developed and tested under R 4.0.0)) — github.com/spcdot/CopyscAT
- **fastcluster** (dependency of Copy-scAT used during clustering-dependent processing) — github.com/spcdot/CopyscAT
- **NMF** (dependency of Copy-scAT for matrix operations) — github.com/spcdot/CopyscAT

## Examples

```
python3 process_fragment_file.py  # then in R: filtered <- filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut); arms <- collapseChrom3N(filtered, minCPG, cutAverage, logTrans=TRUE)
```

## Evaluation signals

- Filtered matrix contains only cells passing minimumSegments, minDensity, and signalSDcut thresholds
- collapseChrom3N output has one summarized value per chromosome arm per cell, with cutAverage computed over the 50th-80th quantile of bin values
- Log transformation (logTrans/logBase) applied to arm-level values is consistent with the configured logBase
- Bin weighting by CpG-island length (minCPG) and binExpand settings match the configured parameters in the run
- Downstream clusterCNV runs without error on the collapsed matrix, indicating values are on a scale compatible with pseudodiploid modeling

## Limitations

- Copy-scAT is in early beta; documentation and testing are still works in progress
- Must be installed within an R 4.0.0 environment as development is ongoing with R 4.0.0
- Copy-scAT cannot be used with malignancies driven primarily by single nucleotide variants with no or rare CNVs

## Evidence

- [methods] filterCells inputMatrix, minimumSegments, minDensity, signalSDcut: "filterCells(inputMatrix, minimumSegments, minDensity, signalSDcut)"
- [methods] collapseChrom3N parameters: "collapseChrom3N(inputMatrix, minimumSegments, summaryFunction, logTrans, binExpand, minimumChromValue, tssEnrich, logBase, minCPG, powVal)"
- [abstract] Copy-scAT is an R package using single-cell epigenomic data: "Here, we describe Copy-scAT, an R package that uses single-cell epigenomic data to infer copy number variants (CNVs) that define cancer cells."
- [readme] README fragment matrix generation: "To generate fragment matrices, use the process_fragment_file.py script (type python3 process_fragment_file.py for details regarding input parameters)"
- [readme] README R 4.0.0 requirement: "It is currently in early beta - documentation and testing are still works in progress. Development is ongoing with R 4.0.0, so please install within an R 4.0.0 environment."
