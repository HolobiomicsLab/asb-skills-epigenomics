---
name: leiden-clustering-of-single-cell-data
description: Use when a snap-format scATAC-seq dataset has been filtered and dimensionality-reduced
  (e.g. via runDimReduct on the Jaccard-normalized cell-by-bin matrix) and you need
  unsupervised cell-type clustering without prior peak annotation.
when_to_use_negative:
- Input is already a peak-based feature matrix or aggregate population-level peak
  annotation — the article argues peak-based approaches bias toward abundant cell
  types and require robust aggregate signals
- You need a scaled pipeline beyond the legacy stack — SnapATAC 2.0 (scverse/SnapATAC2)
  is the maintained successor, and legacy SnapATAC requires R >= 3.4.0 & < 3.6.0 (rhdf5
  incompatibility with 3.6)
- Cells have not been filtered or dimensionality-reduced; clustering raw sparse profiles
  (5–15% of peaks detected per cell) is the very problem this skill presumes you have
  already mitigated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SnapATAC
  role: Provides runCluster implementing Leiden clustering of cells after runDimReduct
    dimensionality reduction of the cell-by-bin matrix
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Preprocessing (snap-pre, snap-add-bmat) that produces the .snap file whose
    cell-by-bin matrix is the input to clustering
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC2
  role: Successor package offering clustering within its end-to-end pipeline for single-cell
    ATAC-seq; recommended for current use
  repo: https://github.com/scverse/SnapATAC2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:cd03fd59a36a498b29acc4f4a801298e07370b879421a2e9aa6e446f3dc117d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  - SnapATAC2
  - SnapTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# leiden-clustering-of-single-cell-data

## Summary

Cluster single-cell ATAC-seq cells using the Leiden algorithm, as implemented in SnapATAC's runCluster on dimensionality-reduced cell-by-bin embeddings, to dissect cellular heterogeneity into discrete cell populations.

## When to use

Use when a snap-format scATAC-seq dataset has been filtered and dimensionality-reduced (e.g. via runDimReduct on the Jaccard-normalized cell-by-bin matrix) and you need unsupervised cell-type clustering without prior peak annotation.

## When NOT to use

- Input is already a peak-based feature matrix or aggregate population-level peak annotation — the article argues peak-based approaches bias toward abundant cell types and require robust aggregate signals
- You need a scaled pipeline beyond the legacy stack — SnapATAC 2.0 (scverse/SnapATAC2) is the maintained successor, and legacy SnapATAC requires R >= 3.4.0 & < 3.6.0 (rhdf5 incompatibility with 3.6)
- Cells have not been filtered or dimensionality-reduced; clustering raw sparse profiles (5–15% of peaks detected per cell) is the very problem this skill presumes you have already mitigated

## Inputs

- Snap (.snap, hdf5) file containing the cell-by-bin accessibility matrix (AM session) and barcodes (BD session)
- Dimensionality-reduced embedding from runDimReduct (pc.num components of the Jaccard-normalized matrix)
- tmp.folder scratch directory for the Leiden graph

## Outputs

- Cell-level cluster assignments appended to the SnapATAC object (x.sp@cluster)
- Cluster labels used to compute ARI/NMI against reference cell-type annotations
- Discrete cell populations for downstream marker/peak analysis (e.g. 31 distinct cell populations in the mouse secondary motor cortex atlas)

## How to apply

After filtering cells (filterCells) and bins (filterBins), building the binary matrix with makeBinary, and normalizing with runNormJaccard, reduce dimensionality with runDimReduct (specifying pc.num) on the cell-by-bin matrix from the .snap file. Then apply runCluster with parameters tmp.folder (scratch space for the Leiden shared-nearest-neighbor graph), louvain.lib, resolution (tunes cluster granularity), and seed.use (reproducibility) to partition cells with the Leiden algorithm. SnapATAC's Leiden clustering operates on genome-wide accessibility similarity rather than population-level peaks, avoiding bias toward abundant cell types. Decision point: adjust resolution iteratively and validate that recovered populations match reference labels or known marker accessibility; in the benchmark, resulting cluster assignments were scored against reference cell labels with ARI and NMI.

## Related tools

- **SnapATAC** (Provides runCluster implementing Leiden clustering of cells after runDimReduct dimensionality reduction of the cell-by-bin matrix) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Preprocessing (snap-pre, snap-add-bmat) that produces the .snap file whose cell-by-bin matrix is the input to clustering) — https://github.com/r3fang/SnapTools
- **SnapATAC2** (Successor package offering clustering within its end-to-end pipeline for single-cell ATAC-seq; recommended for current use) — https://github.com/scverse/SnapATAC2

## Examples

```
runCluster(obj, tmp.folder, louvain.lib, resolution, seed.use)
```

## Evaluation signals

- Clustering assignments are reproducible across different seed.use values at a fixed resolution
- ARI and NMI between Leiden clusters and reference cell labels are computed for every condition; in the coverage benchmark (1,000–10,000 reads/cell) SnapATAC's clustering attained the highest ARI/NMI, significantly outperforming LSA and cisTopic (Wilcoxon signed-rank test, p < 0.01)
- Recovered populations recapitulate known cell types (e.g. the 31 distinct cell populations in the mouse secondary motor cortex) rather than fragments dominated by the most abundant cells
- runCluster completes without tmp.folder I/O errors and the louvain.lib Leiden backend is installed and callable

## Limitations

- Legacy SnapATAC requires R (>= 3.4.0 & < 3.6.0) because 3.6 does not work for the rhdf5 package; SnapTools requires Python 2.7 (< 3.0), and version 1.4.+ now supports both python2 and python3
- Rare cell populations may be underrepresented: in iterative clustering strategies the crude clusters represent the most dominant cell types, so peaks in rare populations may still be underrepresented
- Sparse scATAC-seq signals (5–15% of peaks detected per cell) degrade clustering quality at low coverage, which is why the method is benchmarked across 1,000–10,000 reads per cell

## Evidence

- [methods] SnapATAC enables clustering using leiden algorithm: "SnapATAC enables clustering using leiden algorithm"
- [abstract] SnapATAC dissects cellular heterogeneity in an unbiased manner and maps trajectories of cellular states: "SnapATAC dissects cellular heterogeneity in an unbiased manner and map the trajectories of cellular states."
- [intro] Sparsity motivation for clustering genome-wide accessibility instead of peaks: "the sparsity of signals in each individual profile due to low detection efficiency (5–15% of peaks detected per cell)"
- [readme] R version constraint from requirements: "* R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
