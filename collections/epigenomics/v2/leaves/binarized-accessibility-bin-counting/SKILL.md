---
name: binarized-accessibility-bin-counting
description: Use when scATAC-seq nuclei need a peak-free similarity representation
  — SnapATAC's runNormJaccard requires a binarized cell-by-bin matrix (makeBinary
  + addBmatToSnap) because per-cell signals are too sparse for peak-based clustering.
when_to_use_negative:
- Input is already a peak-by-cell feature matrix or count table from a peak-aware
  pipeline (e.g., CellRanger-ATAC) — binarization of genome-wide bins is redundant
  and the peak matrix can be used directly
- The analysis requires quantitative count depth (e.g., differential accessibility
  testing on counts) rather than binary accessibility similarity — binarization discards
  per-bin fragment counts
- The data are single-cell RNA-seq or other non-accessibility assays — the snap AM
  session and Jaccard framework are designed for ATAC-seq fragments
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: SnapATAC
  role: R functions makeBinary (obj, mat, outlier.filter) and addBmatToSnap (bin.size)
    binarize genome-wide accessibility and attach the bin count matrix to the snap
    object for Jaccard-based similarity computation
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: snaptools snap-add-bmat creates the cell x bin count matrices at chosen bin
    sizes and stores them in the snap AM session before R-side binarization
  repo: https://github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:b93e1d3907fb552294a3a72f5fa0ee31983dafd6d7ca2650e91a55ff9d301bae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# binarized-accessibility-bin-counting

## Summary

Binarizes genome-wide scATAC-seq accessibility into cell-by-bin matrices (e.g., 5,000 and 10,000 bp bins stored in a snap AM session) and counts fragments per bin so that cell-to-cell Jaccard similarity can be computed directly on genome-wide accessibility, without population-level peak annotation prior to clustering.

## When to use

Use when scATAC-seq nuclei need a peak-free similarity representation — SnapATAC's runNormJaccard requires a binarized cell-by-bin matrix (makeBinary + addBmatToSnap) because per-cell signals are too sparse for peak-based clustering.

## When NOT to use

- Input is already a peak-by-cell feature matrix or count table from a peak-aware pipeline (e.g., CellRanger-ATAC) — binarization of genome-wide bins is redundant and the peak matrix can be used directly
- The analysis requires quantitative count depth (e.g., differential accessibility testing on counts) rather than binary accessibility similarity — binarization discards per-bin fragment counts
- The data are single-cell RNA-seq or other non-accessibility assays — the snap AM session and Jaccard framework are designed for ATAC-seq fragments

## Inputs

- snap file (hierarchically structured hdf5, version 4, with HD/BD/AM/PM/GM/FM sessions)
- BAM or BED file of aligned fragments (input to snaptools snap-pre / snap-add-bmat)
- bin size list (e.g., 5000, 10000 bp) for cell-by-bin matrix construction
- QC-filtered snap object after filterCells (subset.names, low.thresholds, high.thresholds)

## Outputs

- Binarized cell-by-bin accessibility matrix attached to the snap object (AM session) via addBmatToSnap
- Binarized snap object input to runNormJaccard for normalized cell-to-cell Jaccard similarity
- snaptools .snap.qc QC summary (usable fragments, unique fragments, chrM fragments per library)

## How to apply

After QC/doublet filtering with filterCells (subset.names, low.thresholds, high.thresholds), binarize the genome-wide accessibility matrix with makeBinary(obj, mat, outlier.filter), which converts each cell's fragment counts per genomic bin to accessible/not-accessible calls while removing outlier bins. Ensure a cell-by-bin count matrix exists in the snap file by running snaptools snap-add-bmat with a --bin-size-list (e.g., 5000 10000) before loading it in R via addBmatToSnap(obj, bin.size); the snap format (version 4) stores these matrices in the AM session of the hdf5 snap file. Binarization is the key decision point: it discards count depth but makes cells comparable across the whole genome, which is what runNormJaccard (row.center, row.scale, ncell.chunk, seed.use) then exploits to compute normalized Jaccard similarity feeding the ensemble Nyström dimensionality reduction. Choose bin size large enough that individual nuclei have usable coverage (5–15% of peaks detected per cell makes fine bins too sparse) but small enough to resolve regulatory elements (~370,000 candidate elements were recovered in the mouse secondary motor cortex atlas).

## Related tools

- **SnapATAC** (R functions makeBinary (obj, mat, outlier.filter) and addBmatToSnap (bin.size) binarize genome-wide accessibility and attach the bin count matrix to the snap object for Jaccard-based similarity computation) — https://github.com/r3fang/SnapATAC
- **SnapTools** (snaptools snap-add-bmat creates the cell x bin count matrices at chosen bin sizes and stores them in the snap AM session before R-side binarization) — https://github.com/r3fang/SnapTools

## Examples

```
snaptools snap-add-bmat --snap-file=demo.snap --bin-size-list 5000 10000 --verbose=True; R: obj <- makeBinary(addBmatToSnap(obj, bin.size=5000), mat="bmat", outlier.filter=TRUE)
```

## Evaluation signals

- The snap file's AM session contains cell-by-bin matrices at the requested bin sizes (e.g., 5,000 and 10,000 bp) after snap-add-bmat
- The binarized matrix contains only 0/1 accessibility calls and outlier bins flagged by outlier.filter are removed
- The .snap.qc summary reports non-zero usable fragments (US) and unique fragments (UQ) per library, with chrM fragments (CM) minimized
- runNormJaccard returns an n-cell x n-cell normalized Jaccard similarity matrix whose dimensions match the filtered cell barcode count
- Downstream Leiden clustering (runCluster) on the embedded bins resolves the expected cell populations (e.g., 20 major clusters in the mouse secondary motor cortex dataset of 55,592 nuclei)

## Limitations

- Per-cell signal is sparse — only 5–15% of peaks are detected per cell — so bin size must be chosen to retain enough covered bins per nucleus
- Binarization discards quantitative fragment counts, so it is unsuitable for analyses that need count-level intensity
- This skill reflects SnapATAC v1 (Python 2.7 for SnapTools, R < 3.6 for rhdf5); the maintained SnapATAC2 (scverse) uses matrix-free spectral embedding instead
- Iterative/peak-based clustering alternatives underrepresent rare cell populations, which is the reason binarized genome-wide bins are preferred here — but rare-population cis-elements still need functional validation

## Evidence

- [methods] makeBinary binarizes genome-wide accessibility and outlier.filter removes anomalous bins: "makeBinary(obj, mat, outlier.filter)"
- [readme] SnapTools snap-add-bmat creates the cell x bin count matrix stored in the snap AM session: [evidence span withheld — anchor status: external_doc]
- [readme] Cell-by-bin matrices of different resolutions (5,000 and 10,000 bp) are added to the snap file without creating another file: [evidence span withheld — anchor status: external_doc]
- [intro] Rationale: binarized genome-wide bins enable peak-free comparison of cell similarity: "it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells"
- [intro] Sparsity of per-cell scATAC-seq signal motivates the binarized approach: "The growing volume of scATAC-seq datasets coupled with the sparsity of signals in each individual profile due to low detection efficiency (5–15% of peaks detected per cell)"
