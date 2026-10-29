---
name: jaccard-kernel-computation-single-cell
description: 'Use when scATAC-seq data must be clustered without predefined peaks:
  cell-by-bin count matrices (e.g. 5-kb bins in a .snap file) need conversion to a
  pair-wise cell similarity kernel for dimensionality reduction.'
when_to_use_negative:
- Input is already a cell-by-peak matrix derived from population-level peak calls
  — SnapATAC's design goal is to avoid peak-annotation bias toward abundant cell types
- You need to scale beyond the R/Python 2.7 SnapATAC stack to millions of cells —
  use SnapATAC2, which implements matrix-free spectral embedding instead of the full
  Jaccard kernel
- Analysis requires batch-effect-corrected multi-sample integration at this step —
  Harmony-based correction is handled as a separate SnapATAC capability
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SnapATAC
  role: R package providing makeBinary, runJaccard, runNormJaccard, runDimReduct,
    and plotting functions for the Jaccard-kernel workflow
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python module that builds the snap file and adds the cell-by-bin count matrix
    (snap-add-bmat) consumed by the Jaccard step
  repo: https://github.com/r3fang/SnapATAC/wiki/FAQs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:8891640e117cbd37c57e7f1c9981c0ea0c3d83f745936c6e2c7c442d1afbaf00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# jaccard-kernel-computation-single-cell

## Summary

Quantifies similarity between single-cell chromatin accessibility profiles by converting binary cell-by-bin matrices into a cell-by-cell Jaccard similarity kernel, then normalizing for sequencing depth, enabling unbiased cell-type identification without population-level peak annotation.

## When to use

Use when scATAC-seq data must be clustered without predefined peaks: cell-by-bin count matrices (e.g. 5-kb bins in a .snap file) need conversion to a pair-wise cell similarity kernel for dimensionality reduction.

## When NOT to use

- Input is already a cell-by-peak matrix derived from population-level peak calls — SnapATAC's design goal is to avoid peak-annotation bias toward abundant cell types
- You need to scale beyond the R/Python 2.7 SnapATAC stack to millions of cells — use SnapATAC2, which implements matrix-free spectral embedding instead of the full Jaccard kernel
- Analysis requires batch-effect-corrected multi-sample integration at this step — Harmony-based correction is handled as a separate SnapATAC capability

## Inputs

- snap file (.snap, HDF5) with cell-by-bin accessibility matrix (AM session), e.g. added via addBmatToSnap at 5-kb or 10-kb bin resolution
- Binarized cell-by-bin matrix (makeBinary output)
- R SnapATAC object (snap obj) with barcode metadata

## Outputs

- Cell-by-cell Jaccard similarity matrix (jmat) whose elements are the fraction of overlapping bins between cell pairs
- Depth-normalized, row-centered and row-scaled Jaccard kernel (runNormJaccard output)
- Low-dimensional eigenvector embedding of cells from runDimReduct (spectral method)
- Elbow plot and pairwise dimensionality-reduction plots for choosing the number of significant dimensions

## How to apply

Load the cell-by-bin accessibility matrix (AM session) from the snap file — e.g. addBmatToSnap at 5-kb resolution for the 10X PBMC example — and binarize it with makeBinary (outlier.filter enabled) so each cell is a binary vector over genome-wide bins. Compute the cell-by-cell Jaccard kernel with runJaccard, where each element is the fraction of overlapping accessible bins between every pair of cells, optionally bin.downsample on highly variable bins to control cost. Normalize the kernel with runNormJaccard to regress out sequencing-depth effects, setting row.center=TRUE and row.scale=TRUE, with ncell.chunk-based chunking and num.cores parallelism writing to tmp.folder. Feed the normalized kernel (input.mat='jmat') to runDimReduct for spectral eigenvector decomposition to obtain the low-dimensional cell embedding; choose significant dimensions via plotDimReductElbow.

## Related tools

- **SnapATAC** (R package providing makeBinary, runJaccard, runNormJaccard, runDimReduct, and plotting functions for the Jaccard-kernel workflow) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python module that builds the snap file and adds the cell-by-bin count matrix (snap-add-bmat) consumed by the Jaccard step) — https://github.com/r3fang/SnapATAC/wiki/FAQs

## Examples

```
jmat <- runJaccard(obj=obj, tmp.folder=tempdir(), max.dist=0.05, bin.downsample=1000); obj <- runNormJaccard(obj=obj, row.center=TRUE, row.scale=TRUE, ncell.chunk=1000, num.cores=4); obj <- runDimReduct(obj=obj, pc.num=50, input.mat='jmat', method='spectral')
```

## Evaluation signals

- The binarized matrix contains only 0/1 values per cell per bin after makeBinary with outlier.filter
- Jaccard matrix is symmetric with diagonal values of 1 (each cell fully overlaps itself)
- runNormJaccard removes the correlation between sequencing depth (fragment counts) and kernel values; row.center=TRUE and row.scale=TRUE confirmed in the call
- Leading eigenvectors from runDimReduct yield a 2D embedding in which PBMC cell populations form visually separable clusters
- Elbow plot shows an inflection point defining the number of significant dimensions retained

## Limitations

- Python 2.7 is required for SnapTools and R must be >=3.4.0 and <3.6.0 because rhdf5 fails on R 3.6
- Computing the full pair-wise Jaccard kernel is expensive at scale; only the ensemble Nyström method lets SnapATAC process up to a million cells
- This skill addresses similarity computation only; downstream steps (clustering, motif analysis, peak calling) belong to other skills

## Evidence

- [other] finding: SnapATAC represents each cell as a binary vector over genome-wide bins and converts these vectors from all cells into a Jaccard similarity matrix: [evidence span withheld — anchor status: not_found]
- [other] workflow: runJaccard and runNormJaccard parameters: [evidence span withheld — anchor status: not_found]
- [methods] runNormJaccard function signature: "runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)"
- [intro] no peak annotation required: "we introduce a software package, Single Nucleus Analysis Pipeline for ATAC-seq—SnapATAC (https://github.com/r3fang/SnapATAC)—that does not require population-level peak annotation prior to clustering"
- [abstract] Nyström scaling: "Using the Nyström method, SnapATAC can process data from up to a million cells."
- [readme] R/Python version constraints: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
