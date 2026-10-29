---
name: clustering-reproducibility-evaluation
description: Use when benchmarking clustering robustness of scATAC-seq methods across
  sequencing depths or cell counts — e.
when_to_use_negative:
- Only a single sequencing depth is available with no downsampled/full-depth pair,
  so no assignment-consistency score can be computed
- The goal is biological interpretation (peak calling, motif analysis with chromVAR,
  gene–peak linkage) rather than method benchmarking
- Input is already a pre-computed feature/clustering table from one method only —
  no cross-method runtime or reproducibility comparison is possible
edam_operation: http://edamontology.org/operation_3432
tools:
- name: SnapATAC
  role: 'Primary method under test: Jaccard-normalized dimensionality reduction (runNormJaccard,
    runDimReduct) and Leiden clustering (runCluster) with fixed seeds for reproducibility
    scoring'
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python module used to downsample sequencing reads to 10% depth and preprocess
    into snap (hdf5) cell-by-bin matrices
  repo: https://github.com/r3fang/SnapTools
- name: cisTopic
  role: Alternative topic-model method benchmarked for runtime scaling and clustering
    reproducibility against SnapATAC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:9bf0710624c772fc8148e830d5b0d83b0fed26747f829b22e1e24c67e0243cb1
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# clustering-reproducibility-evaluation

## Summary

Benchmarks how reproducible and scalable a scATAC-seq dimensionality-reduction/clustering method is by re-clustering sequencing-downsampled data (e.g., 10% depth via SnapTools) with a fixed Leiden seed and scoring cell-type-assignment consistency against full-depth runs, alongside CPU-runtime scaling across 20k–1M cells for SnapATAC vs. LSA vs. cisTopic.

## When to use

Use when benchmarking clustering robustness of scATAC-seq methods across sequencing depths or cell counts — e.g., scoring consistency of Leiden cell-type assignments between 10%-depth downsampled profiles and full-depth runs, or comparing CPU runtime scaling of SnapATAC, LSA and cisTopic from 20k to 1M cells.

## When NOT to use

- Only a single sequencing depth is available with no downsampled/full-depth pair, so no assignment-consistency score can be computed
- The goal is biological interpretation (peak calling, motif analysis with chromVAR, gene–peak linkage) rather than method benchmarking
- Input is already a pre-computed feature/clustering table from one method only — no cross-method runtime or reproducibility comparison is possible

## Inputs

- snap-format hdf5 file with cell-by-bin accessibility matrix (AM session) from SnapTools (e.g., ~81,000-cell 13-mouse-tissue atlas, GSE100033)
- Random cell subsamples of 20k/40k/60k/80k cells and a 1M-cell set via ensemble Nyström approximation
- Fragment/BAM files for 10%-depth downsampling with SnapTools
- Full-depth Leiden cluster (cell-type) assignments as the reference labeling

## Outputs

- benchmark_results.csv with runtime-versus-cell-number rows per method (SnapATAC, LSA, cisTopic)
- Per-method reproducibility scores (consistency of downsampled vs. full-depth cell-type assignments)
- Leiden cluster assignments at 10% depth per method, produced with a fixed seed

## How to apply

Obtain the snap-format (hdf5) dataset — e.g., the Cusanovich et al. 13-mouse-tissue atlas (~81,000 cells, GEO GSE100033) preprocessed with SnapTools into binary cell-by-bin matrices — and generate random subsamples of 20k, 40k, 60k and 80k cells plus a 1M-cell set via the ensemble Nyström approximation. For each method (SnapATAC runDimReduct, LSA runLSA, cisTopic topic models) log CPU runtime with a fixed seed.use and a matched number of eigenvectors so the runtime-vs-cell-number slope is comparable. Downsample sequencing reads to 10% of original depth with SnapTools, re-run each method's full pipeline, then re-cluster with the Leiden algorithm via runCluster(obj, tmp.folder, louvain.lib, resolution, seed.use) using the same fixed seed and resolution as the full-depth run. Score reproducibility as the consistency of cell-type assignments between downsampled and full-depth runs, and compile runtime and reproducibility rows into benchmark_results.csv. Rationale: with sparsity of 5–15% of peaks detected per cell, only methods whose cluster assignments survive 90% read reduction are trustworthy for downstream cis-element discovery in rare cell types.

## Related tools

- **SnapATAC** (Primary method under test: Jaccard-normalized dimensionality reduction (runNormJaccard, runDimReduct) and Leiden clustering (runCluster) with fixed seeds for reproducibility scoring) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python module used to downsample sequencing reads to 10% depth and preprocess into snap (hdf5) cell-by-bin matrices) — https://github.com/r3fang/SnapTools
- **cisTopic** (Alternative topic-model method benchmarked for runtime scaling and clustering reproducibility against SnapATAC)

## Examples

```
runCluster(obj, tmp.folder, louvain.lib="Leiden", resolution=0.5, seed.use=1)
```

## Evaluation signals

- Reproducibility score: consistency of Leiden cell-type assignments between 10%-depth downsampled runs and full-depth runs must be computed with identical seed.use and resolution settings
- Runtime table must show a near-linear increase of CPU time with cell number, with SnapATAC ~10x faster than LSA and >100x faster than cisTopic at 100k cells
- benchmark_results.csv must contain one row per method x cell count (20k, 40k, 60k, 80k, 1M) x sequencing depth (100% and 10%)
- Number of eigenvectors/pcs must be matched across SnapATAC, LSA and cisTopic runs for a fair comparison
- Re-running with the same fixed seed should reproduce identical cluster assignments (seed-controlled determinism)

## Limitations

- 1M-cell analysis relies on the ensemble Nyström approximation rather than exact computation on all cells
- Version constraints: SnapTools requires Python >= 2.7 & < 3.0 (recommended), and SnapATAC requires R >= 3.4.0 & < 3.6.0 because 3.6 does not work for the rhdf5 package
- Iterative clustering strategies require multiple rounds of clustering, aggregation, and peak calling, hindering application to large-scale datasets; peaks in rare populations may be underrepresented
- Runtime and reproducibility results are hardware- and parameter-dependent; slopes are only meaningful with matched eigenvector counts and fixed seeds

## Evidence

- [intro] we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues: "we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues."
- [methods] runNormJaccard signature: "runNormJaccard(obj, tmp.folder, method, row.center, row.scale, low.threshold, high.threshold, do.par, ncell.chunk, num.cores, seed.use)"
- [abstract] Nyström scalability claim: "Using the Nyström method, SnapATAC can process data from up to a million cells."
- [discussion] iterative clustering limitation: "the strategy of iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, thus hindering its application to large-scale datasets."
- [readme] Leiden clustering in README: "SnapATAC enables clustering using leiden algorithm"
