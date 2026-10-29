---
name: clustering-separation-benchmarking-across-methods
description: Use when two or more scATAC-seq embedding/clustering methods must be
  compared on the same cells and you need a quantitative separation metric (connectivity
  index) rather than eyeballing t-SNE overlays.
when_to_use_negative:
- Input is scRNA-seq alone or already a feature/peak table — the benchmark requires
  genome-wide accessibility profiles where embeddings differ in how they resolve sparse
  (5–15% of peaks detected per cell) scATAC-seq signal.
- Goal is de novo cis-regulatory element discovery or rare-population peak calling
  rather than method comparison — use SnapATAC's downstream peak/trajectory workflow
  instead.
- The R environment is 3.6.0 or later, where rhdf5 breaks the legacy SnapATAC package
  — switch to SnapATAC2 rather than forcing this R-based benchmark.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SnapATAC
  role: Provides the benchmarked embedding (runDimReduct on the jaccard matrix), leiden
    clustering, t-SNE (runViz), and the LSA competitor via runLSA; exports to Seurat
    with snapToSeurat
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: 'Builds the snap-format input: snap-pre creates the snap file from aligned
    fragments and snap-add-bmat adds the cell-by-bin count matrix used by both methods'
  repo: https://github.com/r3fang/SnapTools
- name: Seurat
  role: Receives the SnapATAC embedding via snapToSeurat and performs CCA-based anchor
    transfer of cell-type labels from the scRNA-seq reference
- name: SnapATAC2
  role: Python/Rust successor package for users who cannot run the legacy R pipeline;
    offers matrix-free spectral embedding and clustering for the same comparison style
  repo: https://github.com/scverse/SnapATAC2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:49060c4658e8693d320d4ba89f4b1754bd6ed25aafaeabef2bc941e284b79109
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - Seurat
  - SnapATAC
  - SnapATAC2
  - SnapTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# clustering-separation-benchmarking-across-methods

## Summary

A benchmarking protocol that compares scATAC-seq dimensionality-reduction methods (SnapATAC vs. LSA, extensible to cisTopic) on an identical cell subset by quantifying cluster separation with a connectivity index after label transfer from a matched scRNA-seq reference.

## When to use

Use when two or more scATAC-seq embedding/clustering methods must be compared on the same cells and you need a quantitative separation metric (connectivity index) rather than eyeballing t-SNE overlays.

## When NOT to use

- Input is scRNA-seq alone or already a feature/peak table — the benchmark requires genome-wide accessibility profiles where embeddings differ in how they resolve sparse (5–15% of peaks detected per cell) scATAC-seq signal.
- Goal is de novo cis-regulatory element discovery or rare-population peak calling rather than method comparison — use SnapATAC's downstream peak/trajectory workflow instead.
- The R environment is 3.6.0 or later, where rhdf5 breaks the legacy SnapATAC package — switch to SnapATAC2 rather than forcing this R-based benchmark.

## Inputs

- snap file with cell-by-bin accessibility matrix (e.g. 10X_PBMC_15K snap from 10x scATAC-seq)
- binary-to-count matrix produced via addBmatToSnap + makeBinary
- matched scRNA-seq reference (~10k-cell PBMC 10x dataset) for cell-type label transfer
- R environment >= 3.4.0 and < 3.6.0 (3.6 breaks the rhdf5 dependency)

## Outputs

- per-method connectivity index quantifying cluster separation on the same cell subset
- cell-type annotations for scATAC cells transferred from the scRNA-seq reference (Seurat CCA anchors)
- joint t-SNE visualization of transferred labels (e.g. PBMC_ATAC_RNA.png via plotViz)
- ranked comparison of embedding methods by sensitivity to distinguishing cell types

## How to apply

Fix a single cell subset (here, the 4,792-cell PBMC subset from the 10X_PBMC_15K snap file) and run every competing method on identical input: build the binary-to-count matrix with addBmatToSnap followed by makeBinary, then reduce dimensionality per method (SnapATAC runDimReduct on the jaccard matrix; the competitor runLSA) and cluster with runCluster using the leiden algorithm. Transfer cell-type labels from the ~10k-cell PBMC scRNA-seq reference via Seurat's CCA-based anchor transfer using snapToSeurat, project labels onto each t-SNE (runViz/plotViz), and compute a connectivity index quantifying cluster separation for each embedding. Judge the result by the direction of the connectivity-index difference between methods on the same cells — the paper's PBMC example found SnapATAC separates predicted cell populations more effectively than LSA. Keep bin size, filtering (filterCells/filterBins), and leiden parameters identical across methods so the only variable is the embedding.

## Related tools

- **SnapATAC** (Provides the benchmarked embedding (runDimReduct on the jaccard matrix), leiden clustering, t-SNE (runViz), and the LSA competitor via runLSA; exports to Seurat with snapToSeurat) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Builds the snap-format input: snap-pre creates the snap file from aligned fragments and snap-add-bmat adds the cell-by-bin count matrix used by both methods) — https://github.com/r3fang/SnapTools
- **Seurat** (Receives the SnapATAC embedding via snapToSeurat and performs CCA-based anchor transfer of cell-type labels from the scRNA-seq reference)
- **SnapATAC2** (Python/Rust successor package for users who cannot run the legacy R pipeline; offers matrix-free spectral embedding and clustering for the same comparison style) — https://github.com/scverse/SnapATAC2

## Examples

```
x.sp <- runNormJaccard(x.sp, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)
```

## Evaluation signals

- The connectivity index is computed on the identical 4,792-cell PBMC subset for every method; only the embedding differs between runs.
- Direction of the connectivity-index difference is consistent with the paper's PBMC result: SnapATAC separates predicted PBMC cell populations more effectively than LSA (lower connectivity index, i.e. higher cluster separation in the task's definition).
- Transferred cell-type labels from the ~10k-cell scRNA-seq reference (Seurat CCA anchors) assign interpretable PBMC populations that form distinct clusters in the t-SNE.
- The joint visualization (PBMC_ATAC_RNA.png) faithfully reproduces the published PBMC_ATAC_RNA panel, per expert_review in the source task.
- Clustering uses the leiden algorithm and the same binary matrix (makeBinary) for both methods, so parameter differences cannot explain the separation gap.

## Limitations

- The legacy R pipeline is version-locked: R 3.6.0 does not work for the rhdf5 package, so the benchmark must run under R >= 3.4.0 & < 3.6.0.
- The connectivity-index direction ('higher = more separated' vs. 'lower = more separated') must be defined once and applied consistently; the source task's finding and validation notes phrase it in opposite directions.
- The comparison was demonstrated on one 4,792-cell PBMC subset; extending conclusions about sensitivity to other tissues requires re-running the full benchmark there.

## Evidence

- [methods] SnapATAC enables clustering using leiden algorithm: "SnapATAC enables clustering using leiden algorithm"
- [readme] SnapATAC integrates scRNA and scATAC: "SnapATAC integrates scRNA and scATAC"
- [methods] R version constraint for SnapATAC: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [intro] SnapATAC superiority claim motivating the benchmark: "we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues."
- [readme] SnapATAC dimensionality-reduction method under benchmark: "SnapATAC employs a new method for dimensionality reduction"
