---
name: sequencing-depth-regression-normalization
description: Use when a SnapATAC cell-by-cell Jaccard kernel must be normalized for
  per-cell sequencing-depth bias before spectral dimensionality reduction of scATAC-seq
  data.
when_to_use_negative:
- Input is already a normalized feature matrix or embedding rather than a raw cell-by-cell
  Jaccard kernel — re-normalizing would double-correct depth effects
- Peak-by-cell count matrices from predefined population-level peaks — this skill
  is designed for the peak-free, genome-wide bin Jaccard framework of SnapATAC
- When using SnapATAC2, which replaces the Jaccard kernel with a matrix-free spectral
  embedding algorithm and does not expose runNormJaccard
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: SnapATAC
  role: Provides runNormJaccard (regression-based depth normalization with row.center/row.scale
    and ncell.chunk parallelization) and runDimReduct for the eigendecomposition that
    consumes the normalized kernel
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Preprocesses reads and creates the snap file whose AM session cell-by-bin
    matrix (added via snap-add-bmat / addBmatToSnap at 5-kb resolution) feeds the
    Jaccard computation being normalized
  repo: github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:b4b8f840d84aff9c317610b55b5de9982ca05156fa1795093776332e6a9cac18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sequencing-depth-regression-normalization

## Summary

Regression-based sequencing-depth normalization of the cell-by-cell Jaccard similarity kernel in SnapATAC, followed by row centering and scaling (runNormJaccard), so that eigendecomposition is not dominated by per-cell differences in sequencing depth.

## When to use

Use when a SnapATAC cell-by-cell Jaccard kernel must be normalized for per-cell sequencing-depth bias before spectral dimensionality reduction of scATAC-seq data.

## When NOT to use

- Input is already a normalized feature matrix or embedding rather than a raw cell-by-cell Jaccard kernel — re-normalizing would double-correct depth effects
- Peak-by-cell count matrices from predefined population-level peaks — this skill is designed for the peak-free, genome-wide bin Jaccard framework of SnapATAC
- When using SnapATAC2, which replaces the Jaccard kernel with a matrix-free spectral embedding algorithm and does not expose runNormJaccard

## Inputs

- cell-by-cell Jaccard similarity kernel computed from the binarized 5-kb cell-by-bin matrix (runJaccard output)
- Snap object loaded from a .snap file (HDF5, snap format v4) with AM session cell-by-bin matrix
- per-cell sequencing-depth / total-fragment-count covariates used for regression normalization

## Outputs

- normalized Jaccard kernel (jmat) with rows centered and scaled, ready for runDimReduct eigendecomposition
- leading eigenvectors (low-dimensional cell embedding) consumed by plotViz/plotDimReductPW and cluster detection

## How to apply

After building the cell-by-cell Jaccard kernel from the binarized cell-by-bin matrix (5-kb bins added via addBmatToSnap), apply runNormJaccard with regression-based depth normalization to remove the association between each cell's total fragment counts and its similarity profile. Set row.center=TRUE and row.scale=TRUE to center and scale each row of the normalized kernel, which is required before spectral eigendecomposition in runDimReduct. For large datasets, use ncell.chunk-based chunking with do.par and num.cores to parallelize the computation, directing temporary files to tmp.folder and fixing the random seed with seed.use for reproducibility. The output normalized kernel (jmat) is then passed to runDimReduct(input.mat='jmat') with the spectral method to obtain the low-dimensional embedding.

## Related tools

- **SnapATAC** (Provides runNormJaccard (regression-based depth normalization with row.center/row.scale and ncell.chunk parallelization) and runDimReduct for the eigendecomposition that consumes the normalized kernel) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Preprocesses reads and creates the snap file whose AM session cell-by-bin matrix (added via snap-add-bmat / addBmatToSnap at 5-kb resolution) feeds the Jaccard computation being normalized) — github.com/r3fang/SnapTools

## Examples

```
x.norm <- runNormJaccard(x.sp, tmp.folder=tempdir(), row.center=TRUE, row.scale=TRUE, do.par=TRUE, ncell.chunk=5000, num.cores=4, seed.use=10)
```

## Evaluation signals

- Regression of kernel rows on per-cell depth covariates removes the correlation between each cell's total usable fragments and its similarity profile
- After row.center=TRUE/row.scale=TRUE, each row of the normalized kernel has approximately zero mean and unit scaling
- runDimReduct(input.mat='jmat', spectral) converges and yields leading eigenvectors that are not dominated by the top depth-associated component
- Elbow plot (plotDimReductElbow) shows a clear number of significant dimensions, and plotViz/plotDimReductPW reveals visually separable PBMC cell-population clusters
- Chunked run (ncell.chunk with num.cores) reproduces the same normalized kernel when seed.use is fixed

## Limitations

- Requires R (>= 3.4.0 & < 3.6.0) because the rhdf5 package fails on R 3.6, and SnapTools under Python (>= 2.7 & < 3.0)
- Without bin.downsample on highly variable bins and depth regression, the leading eigenvector can capture technical depth variation instead of biological cell types
- Memory/compute-heavy for very large datasets; SnapATAC addresses scale via the ensemble Nyström method, and SnapATAC2 replaces this kernel pipeline entirely

## Evidence

- [methods] runNormJaccard signature: "runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)"
- [methods] makeBinary signature: "makeBinary(obj, mat, outlier.filter)"
- [intro] peak-free similarity principle: "it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells."
- [methods] R version constraint: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [readme] Python version constraint: "Python (>= 2.7 & < 3.0) (SnapTools) (highly recommanded for 2.7);"
