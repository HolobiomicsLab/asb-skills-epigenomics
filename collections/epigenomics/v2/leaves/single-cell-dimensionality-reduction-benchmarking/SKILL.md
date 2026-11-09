---
name: single-cell-dimensionality-reduction-benchmarking
description: Use when comparing feature-reduction/embedding methods for scATAC-seq
  (or similar sparse single-cell) cell-type clustering, where robustness to per-cell
  coverage must be quantified with ARI/NMI against known labels.
when_to_use_negative:
- Input is already a reduced/embedded representation or a feature table rather than
  raw fragments or a snap file
- No reference cell labels exist, so ARI/NMI cannot be computed (the benchmark metric
  is undefined)
- Goal is single-method exploratory clustering rather than comparative robustness
  evaluation across methods
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: SnapATAC
  role: 'Candidate method under test: bin-based dimensionality reduction (runDimReduct)
    plus Leiden clustering (runCluster) for cell-type identification'
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python 2.7 preprocessing module used to build snap files (snap-pre, snap-add-bmat)
    that feed the benchmark
  repo: https://github.com/r3fang/SnapTools
- name: LSA
  role: Comparator method run via runLSA(obj, input.mat, pc.num, logTF, scale.factor,
    min.cell, seed.use) followed by identical Leiden clustering
- name: cisTopic
  role: Comparator method evaluated at every simulated coverage level for ARI/NMI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:b16f904c941eb1c1b16a38d05d63f3e7b3ed4cc4ba903d1b28d2c19fc2b87320
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - LSA
  - R
  - SnapATAC
  - SnapTools
  - cisTopic
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-dimensionality-reduction-benchmarking

## Summary

Benchmarks single-cell dimensionality-reduction/clustering methods (e.g., SnapATAC vs LSA vs cisTopic) for cell-type identification by measuring ARI/NMI against reference labels across simulated coverage regimes.

## When to use

Use when comparing feature-reduction/embedding methods for scATAC-seq (or similar sparse single-cell) cell-type clustering, where robustness to per-cell coverage must be quantified with ARI/NMI against known labels.

## When NOT to use

- Input is already a reduced/embedded representation or a feature table rather than raw fragments or a snap file
- No reference cell labels exist, so ARI/NMI cannot be computed (the benchmark metric is undefined)
- Goal is single-method exploratory clustering rather than comparative robustness evaluation across methods

## Inputs

- Snap-format (.snap) scATAC-seq dataset with cell-by-bin matrix (e.g., 10X_PBMC_15K example)
- Published bulk ATAC-seq datasets for downsampling-based coverage simulation (1,000-10,000 reads per cell)
- Reference cell-type labels per cell for ARI/NMI computation

## Outputs

- ari_nmi_comparison.csv with one row per (method, coverage level) pair
- Wilcoxon signed-rank test p-values comparing methods across coverage levels

## How to apply

Generate simulated datasets by downsampling published bulk or single-cell ATAC-seq profiles to a range of per-cell coverages (e.g., 1,000-10,000 reads per cell). For each coverage level run each candidate method end-to-end on identical inputs: for SnapATAC run makeBinary on the bin matrix, runDimRe­duct for dimensionality reduction with the chosen pc.num, then Leiden/Louvain clustering via runCluster (tmp.folder, louvain.lib, resolution, seed.use); for the LSA comparator run runLSA(obj, input.mat, pc.num, logTF, scale.factor, min.cell, seed.use) followed by the same Leiden clustering. Compute ARI and NMI between each method's cluster assignments and the reference cell labels at every coverage level, assemble one row per (method, coverage) pair, and assess significance of pairwise differences with a Wilcoxon signed-rank test (the source study reports p < 0.01 favoring SnapATAC).

## Related tools

- **SnapATAC** (Candidate method under test: bin-based dimensionality reduction (runDimReduct) plus Leiden clustering (runCluster) for cell-type identification) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python 2.7 preprocessing module used to build snap files (snap-pre, snap-add-bmat) that feed the benchmark) — https://github.com/r3fang/SnapTools
- **LSA** (Comparator method run via runLSA(obj, input.mat, pc.num, logTF, scale.factor, min.cell, seed.use) followed by identical Leiden clustering)
- **cisTopic** (Comparator method evaluated at every simulated coverage level for ARI/NMI)

## Examples

```
install_github("r3fang/SnapATAC")  # R < 3.6; then per coverage: runDimReduct(obj, pc.num=...); runCluster(obj, louvain.lib, resolution, seed.use); ARI/NMI vs reference labels
```

## Evaluation signals

- Ari_nmi_comparison.csv contains one row per (method, coverage) pair with non-null ARI and NMI values
- Winning method attains the highest ARI/NMI across the full 1,000-10,000 reads/cell range, not just at one coverage extreme
- Pairwise method differences are confirmed by Wilcoxon signed-rank test with p < 0.01
- Identical clustering settings (resolution, seed.use, pc.num) are used across methods so differences are attributable to the embedding, not clustering parameters
- Clustering at each coverage is reproducible given seed.use is fixed

## Limitations

- Simulated coverage range only spans 1,000-10,000 reads per cell; conclusions may not extrapolate to sparser or deeper data
- SnapATAC requires R (>= 3.4.0 & < 3.6.0) because R 3.6 does not work with the rhdf5 package, and SnapTools is Python 2.7-oriented
- Iterative clustering approaches hinder application to large-scale datasets, and rare-population peaks may be underrepresented in iterative peak-based comparators

## Evidence

- [results] SnapATAC showed the highest robustness and accuracy in identifying the original cell types, measured by ARI and NMI, significantly outperforming LSA and cis-Topic (Wilcoxon signed-rank test, p < 0.01).: [evidence span withheld — anchor status: not_found]
- [intro] we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues.: "we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues."
- [readme] R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [methods] makeBinary(obj, mat, outlier.filter): "makeBinary(obj, mat, outlier.filter)"
