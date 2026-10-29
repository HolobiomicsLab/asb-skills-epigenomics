---
name: nystrom-approximation-large-scale-simulation
description: Use when the goal is to benchmark or extrapolate dimensionality-reduction
  scalability of scATAC-seq data beyond the available cell count (e.g., 20k-80k measured
  cells up to a simulated 1M cells) using the ensemble Nyström approximation.
when_to_use_negative:
- The dataset is small enough (<~100k cells) that an exact eigendecomposition of the
  Jaccard matrix is feasible and approximation error is not worth trading off
- Input is already a peak-based feature matrix or the analysis requires population-level
  peak annotation prior to clustering, which is the design SnapATAC's bin-based Jaccard/Nyström
  path avoids
- The environment cannot satisfy the original SnapATAC runtime constraints (Python
  >= 2.7 & < 3.0 for SnapTools; R >= 3.4.0 & < 3.6.0 because 3.6 breaks the rhdf5
  package) — use SnapATAC2 instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SnapATAC
  role: Runs the ensemble Nyström-approximated dimensionality reduction (runDimReduct)
    over the normalized Jaccard similarity matrix with a fixed seed
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Preprocesses reads into snap-format HDF5 files and generates the cell-by-bin
    count matrices (snap-pre, snap-add-bmat) that feed the Jaccard computation
  repo: https://github.com/r3fang/SnapTools
- name: cisTopic
  role: Baseline topic-model method benchmarked against the Nyström-based SnapATAC
    runtime scaling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:3c82a98f53155aee614e8de9127d7db302439bc151eeeb47faa2a2228a44ab7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Python
  - R
  - SnapATAC
  - SnapTools
  - cisTopic
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# nystrom-approximation-large-scale-simulation

## Summary

Applies the ensemble Nyström method within SnapATAC to approximate dimensionality reduction of the cell-by-cell Jaccard similarity matrix, enabling runtime-scaling benchmarks and simulated 1M-cell scATAC-seq analysis on regular hardware.

## When to use

Use when the goal is to benchmark or extrapolate dimensionality-reduction scalability of scATAC-seq data beyond the available cell count (e.g., 20k-80k measured cells up to a simulated 1M cells) using the ensemble Nyström approximation.

## When NOT to use

- The dataset is small enough (<~100k cells) that an exact eigendecomposition of the Jaccard matrix is feasible and approximation error is not worth trading off
- Input is already a peak-based feature matrix or the analysis requires population-level peak annotation prior to clustering, which is the design SnapATAC's bin-based Jaccard/Nyström path avoids
- The environment cannot satisfy the original SnapATAC runtime constraints (Python >= 2.7 & < 3.0 for SnapTools; R >= 3.4.0 & < 3.6.0 because 3.6 breaks the rhdf5 package) — use SnapATAC2 instead

## Inputs

- snap (HDF5) file with cell-by-bin accessibility matrix (AM session) from SnapTools
- Binary cell-by-bin matrices (5kb / 10kb bins) for the ~81k-cell mouse 13-tissue atlas (GSE100033)
- Random cell subsamples (20k, 40k, 60k, 80k cells) and a simulated 1M-cell dataset via ensemble Nyström

## Outputs

- Normalized Jaccard similarity matrix (runNormJaccard output)
- Low-dimensional eigenvector embeddings ( PCs) from Nyström-approximated runDimReduct
- Runtime-versus-cell-number benchmark table (e.g., benchmark_results.csv) comparing SnapATAC, LSA, cisTopic

## How to apply

First preprocess the scATAC-seq data with SnapTools (snap-pre with --min-mapq=30, --max-flen=1000, then snap-add-bmat at 5kb/10kb bin sizes) to build cell-by-bin matrices in snap (HDF5) format. Compute the normalized cell-by-cell Jaccard similarity with runNormJaccard (chunked, fixed seed.use), since SnapATAC resolves heterogeneity by comparing genome-wide accessibility profiles directly without prior peak calling. Instead of the exact eigendecomposition, invoke the ensemble Nyström method (runDimReduct on the Jaccard matrix with a fixed seed and matched number of eigenvectors/PCs) to approximate the spectrum from a sampled landmark subset. Generate random subsamples (e.g., 20k, 40k, 60k, 80k cells from the ~81k-cell Cusanovich 13-mouse-tissue atlas, GEO GSE100033) plus the Nyström-simulated 1M-cell dataset, log CPU runtime at each size, and compare against LSA (runLSA) and cisTopic topic models on the same subsets. Judge success by near-linear runtime growth with cell number — at 100k cells SnapATAC should be ~10x faster than LSA and >100x faster than cisTopic.

## Related tools

- **SnapATAC** (Runs the ensemble Nyström-approximated dimensionality reduction (runDimReduct) over the normalized Jaccard similarity matrix with a fixed seed) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Preprocesses reads into snap-format HDF5 files and generates the cell-by-bin count matrices (snap-pre, snap-add-bmat) that feed the Jaccard computation) — https://github.com/r3fang/SnapTools
- **cisTopic** (Baseline topic-model method benchmarked against the Nyström-based SnapATAC runtime scaling)

## Examples

```
runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)
```

## Evaluation signals

- CPU runtime of dimensionality reduction grows approximately linearly with cell number (20k → 1M) rather than super-linearly
- At 100k cells, SnapATAC runtime is nearly 10x lower than LSA and >100x lower than cisTopic on the same subsets
- Eigenvector embeddings from the Nyström approximation reproduce clustering (e.g., Leiden with fixed seed and resolution) consistent with full-depth runs
- Downsampling reads to 10% depth (90% reduction) still yields reproducible cell-type assignments versus full-depth clustering
- Fixed seed.use across runNormJaccard and runDimReduct produces deterministic embeddings

## Limitations

- The 1M-cell capability is an approximation via the ensemble Nyström method, not an exact decomposition, and the original r3fang/SnapATAC package is superseded by SnapATAC2 (scverse), which claims scaling to more than 10 million cells
- Original SnapATAC requires R >= 3.4.0 & < 3.6.0 (R 3.6 breaks the rhdf5 package) and SnapTools under Python 2.7 (< 3.0 recommended)
- Iterative clustering strategies that aggregate and re-call peaks require multiple rounds, hindering application to large-scale datasets and underrepresenting peaks in rare populations

## Evidence

- [abstract] Using the Nyström method, SnapATAC can process data from up to a million cells.: "Using the Nyström method, SnapATAC can process data from up to a million cells."
- [intro] ensemble Nyström method: "We also adopt a technique, ensemble Nyström method20,21, that improves the"
- [intro] outperforms current methods in scalability: "we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues."
- [discussion] runs on standard hardware: "The open-source software runs on standard hardware"
- [readme] SnapATAC2 scale: [evidence span withheld — anchor status: external_doc]
