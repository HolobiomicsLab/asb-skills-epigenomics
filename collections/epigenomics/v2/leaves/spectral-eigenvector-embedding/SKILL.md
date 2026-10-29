---
name: spectral-eigenvector-embedding
description: Use when you have a binarized cell-by-bin accessibility matrix (e.g.,
  5-kb bins in a snap file) and need a peak-free low-dimensional embedding of single-cell
  chromatin profiles for clustering rare populations.
when_to_use_negative:
- Input is already a peak-by-cell matrix derived from population-level peak calls
  — SnapATAC's Jaccard/spectral approach is designed to avoid predefined peak annotation,
  which biases toward abundant cell types
- Data are extremely sparse raw fragment counts without binarization — the Jaccard
  kernel requires binary per-cell accessible-genome profiles
- 'Python/R environment constraints are violated: SnapTools requires Python >= 2.7
  & < 3.0 and SnapATAC requires R >= 3.4.0 & < 3.6.0 (rhdf5 fails on R 3.6); use SnapATAC2
  instead'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3366
tools:
- name: SnapATAC
  role: R package providing runJaccard, runNormJaccard, and runDimReduct (spectral
    eigendecomposition of the Jaccard kernel) plus elbow and 2D visualization plots
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python module used to build the snap file and add the cell-by-bin count matrix
    (snap-add-bmat) that serves as input to binarization and kernel construction
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC2
  role: Python/Rust successor implementing a matrix-free spectral embedding algorithm
    that scales beyond the original R implementation
  repo: https://github.com/scverse/SnapATAC2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:efca0b54367a35fde9fb886ce27a37e6796783d3b4fc5af2d3353310bc6a2e60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapATAC2
  - SnapTools
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# spectral-eigenvector-embedding

## Summary

SnapATAC's spectral embedding: eigenvector decomposition of the depth-normalized cell-by-cell Jaccard kernel (jmat) to produce a low-dimensional representation of scATAC-seq cells without predefined population-level peaks.

## When to use

Use when you have a binarized cell-by-bin accessibility matrix (e.g., 5-kb bins in a snap file) and need a peak-free low-dimensional embedding of single-cell chromatin profiles for clustering rare populations.

## When NOT to use

- Input is already a peak-by-cell matrix derived from population-level peak calls — SnapATAC's Jaccard/spectral approach is designed to avoid predefined peak annotation, which biases toward abundant cell types
- Data are extremely sparse raw fragment counts without binarization — the Jaccard kernel requires binary per-cell accessible-genome profiles
- Python/R environment constraints are violated: SnapTools requires Python >= 2.7 & < 3.0 and SnapATAC requires R >= 3.4.0 & < 3.6.0 (rhdf5 fails on R 3.6); use SnapATAC2 instead

## Inputs

- snap file with cell-by-bin accessibility matrix added at 5-kb resolution (via addBmatToSnap)
- binarized cell-by-bin matrix (from makeBinary)
- normalized cell-by-cell Jaccard kernel matrix (jmat)

## Outputs

- cell eigenvector embedding matrix (leading eigenvectors from runDimReduct)
- elbow/scree plot for dimension selection (plotDimReductElbow)
- 2D pairwise-dimension visualization of cell clusters (plotViz, plotDimReductPW)

## How to apply

First binarize the cell-by-bin matrix with makeBinary (outlier.filter enabled) so each cell is a binary vector over genome-wide bins. Compute the cell-by-cell Jaccard similarity matrix with runJaccard (with bin.downsample on highly variable bins), then remove sequencing-depth bias using runNormJaccard with regression-based depth normalization, row.center=TRUE, row.scale=TRUE, and ncell.chunk-based parallel processing (tmp.folder for chunked IO). Run eigendecomposition of the normalized kernel via runDimReduct with input.mat='jmat' and the spectral method to obtain leading eigenvectors as the cell embedding. Decide the number of significant dimensions from a plotDimReductElbow scree curve, and inspect the result with plotViz/plotDimReductPW 2D projections.

## Related tools

- **SnapATAC** (R package providing runJaccard, runNormJaccard, and runDimReduct (spectral eigendecomposition of the Jaccard kernel) plus elbow and 2D visualization plots) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python module used to build the snap file and add the cell-by-bin count matrix (snap-add-bmat) that serves as input to binarization and kernel construction) — https://github.com/r3fang/SnapTools
- **SnapATAC2** (Python/Rust successor implementing a matrix-free spectral embedding algorithm that scales beyond the original R implementation) — https://github.com/scverse/SnapATAC2

## Examples

```
runDimReduct(obj, input.mat='jmat', method='spectral', do.par=TRUE, num.cores=4)
```

## Evaluation signals

- Pipeline completes runDimReduct on input.mat='jmat' with the spectral method and returns leading eigenvectors for every cell
- plotDimReductElbow shows a clear scree/elbow from which a small number of significant dimensions is chosen
- plotViz/plotDimReductPW 2D projections show PBMC cell populations forming visually separable clusters
- Binarized matrix (makeBinary with outlier.filter) contains only 0/1 per-cell bin values before kernel computation
- runNormJaccard was run with row.center=TRUE and row.scale=TRUE so the kernel is depth-normalized and row-centered/scaled prior to eigendecomposition

## Limitations

- Software version constraints: SnapTools requires Python (>= 2.7 & < 3.0), SnapATAC requires R (>= 3.4.0 & < 3.6.0) because rhdf5 does not work on R 3.6
- Peak-based/iterative clustering alternatives require multiple rounds of clustering, aggregation, and peak calling, hindering application to large-scale datasets — the Jaccard-kernel approach avoids this but is specific to genome-wide bin representations
- Very large datasets (up to ~1 million cells) require the ensemble Nyström method rather than a full kernel eigendecomposition

## Evidence

- [other] SnapATAC represents each cell as a binary vector over genome-wide bins and converts these vectors from all cells into a Jaccard similarity matrix: [evidence span withheld — anchor status: not_found]
- [other] Run eigenvector decomposition of the normalized kernel with runDimReduct (input.mat='jmat', spectral method, centered and scaled): [evidence span withheld — anchor status: not_found]
- [other] Compute the cell-by-cell Jaccard kernel with runJaccard (bin.downsample on highly variable bins) and apply regression-based sequencing-depth normalization plus row centering/scaling with runNormJaccard: "runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)"
- [intro] it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells: "it resolves cellular heterogeneity by directly comparing the similarity in genome-wide accessibility profiles between cells"
- [readme] R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
