---
name: clustering-agreement-evaluation-ari-nmi
description: Use when cluster or cell-type assignments from one or more methods must
  be scored against reference labels across multiple conditions (e.g., coverage levels),
  producing ARI/NMI rows suitable for paired significance testing.
when_to_use_negative:
- Predicted and reference labels are not matched cell-by-cell (no shared barcodes/cell
  IDs), since ARI/NMI require paired per-cell assignments
- The goal is to evaluate trajectory or pseudotime orderings rather than discrete
  cluster recovery, for which ARI/NMI agreement metrics are inappropriate
- The input is already a feature/peak table and no clustering has been run — ARI/NMI
  evaluation presupposes completed cluster assignments from a pipeline such as makeBinary
  → runDimReduct → runCluster
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SnapATAC
  role: Generates the cluster assignments being scored — makeBinary, runDimReduct,
    and Leiden runCluster produce the per-cell cluster labels compared against reference
    labels via ARI/NMI.
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python preprocessing module that builds the snap files (cell-by-bin matrices)
    downsampled to each coverage level, which are the inputs to clustering and hence
    to ARI/NMI scoring.
  repo: https://github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:72d0887cfbad52215aa508a5da56326013f4cb177c63fa15f255511b80844826
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - R
  - SnapATAC
  - SnapTools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Clustering agreement evaluation (ARI/NMI)

## Summary

Quantifies how faithfully a clustering method recovers reference cell-type labels by computing Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) per condition, as used to benchmark SnapATAC against LSA and cisTopic across simulated scATAC-seq coverages of 1,000–10,000 reads per cell.

## When to use

Use when cluster or cell-type assignments from one or more methods must be scored against reference labels across multiple conditions (e.g., coverage levels), producing ARI/NMI rows suitable for paired significance testing.

## When NOT to use

- Predicted and reference labels are not matched cell-by-cell (no shared barcodes/cell IDs), since ARI/NMI require paired per-cell assignments
- The goal is to evaluate trajectory or pseudotime orderings rather than discrete cluster recovery, for which ARI/NMI agreement metrics are inappropriate
- The input is already a feature/peak table and no clustering has been run — ARI/NMI evaluation presupposes completed cluster assignments from a pipeline such as makeBinary → runDimReduct → runCluster

## Inputs

- Per-method cluster/cell-type assignments (Leiden cluster labels from SnapATAC runCluster or LSA-runCluster output)
- Reference cell-type labels for the same cells (from downsampling 10 published bulk ATAC-seq datasets)
- Simulated coverage variants (1,000–10,000 reads per cell) of a 10X_PBMC_15K snap dataset

## Outputs

- ari_nmi_comparison.csv with one row per (method, coverage level) pair containing ARI and NMI scores
- Wilcoxon signed-rank test results comparing ARI/NMI distributions between methods (e.g., SnapATAC vs LSA/cisTopic, p < 0.01)

## How to apply

For each condition (here, simulated scATAC-seq datasets downsampled to 1,000–10,000 reads per cell from 10 bulk ATAC-seq datasets), obtain each method's cluster assignments: run the SnapATAC pipeline (makeBinary on the bin matrix, runDimReduct with the specified pc.num, then Leiden runCluster with tmp.folder, louvain.lib, resolution, seed.use) and the LSA comparator (runLSA(obj, input.mat, pc.num, logTF, scale.factor, min.cell, seed.use) followed by the same Leiden clustering). Compute ARI and NMI between each method's assignments and the reference cell labels at every coverage level. Assemble one row per (method, coverage level) pair into ari_nmi_comparison.csv. Judge significance by comparing paired score distributions across coverage levels with a Wilcoxon signed-rank test; the benchmark passes only if the claimed winner (SnapATAC) attains the highest ARI/NMI across the full 1,000–10,000 reads/cell range with p < 0.01.

## Related tools

- **SnapATAC** (Generates the cluster assignments being scored — makeBinary, runDimReduct, and Leiden runCluster produce the per-cell cluster labels compared against reference labels via ARI/NMI.) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python preprocessing module that builds the snap files (cell-by-bin matrices) downsampled to each coverage level, which are the inputs to clustering and hence to ARI/NMI scoring.) — https://github.com/r3fang/SnapTools

## Examples

```
devtools::install_github("r3fang/SnapATAC")
```

## Evaluation signals

- All ARI and NMI values fall in [0, 1] for every (method, coverage) pair in ari_nmi_comparison.csv.
- The table contains exactly one row per (method, coverage level) pair covering the full 1,000–10,000 reads/cell range.
- SnapATAC attains the highest ARI and NMI across the entire coverage range, consistent with the paper's claim that it outperforms current methods in accuracy and reproducibility for cell-type identification.
- Wilcoxon signed-rank tests of paired score differences yield p < 0.01 favoring SnapATAC over LSA and cisTopic.
- Each cluster-assignment vector has the same length and cell ordering as the reference label vector before scoring.

## Limitations

- SnapATAC (R) requires R >= 3.4.0 & < 3.6.0 because R 3.6 does not work for the rhdf5 package used to read snap files.
- SnapTools preprocessing is run under Python (>= 2.7 & < 3.0), with 2.7 highly recommended, so the benchmark pipeline is version-constrained.
- scATAC-seq signal sparsity (only 5–15% of peaks detected per cell) is the underlying difficulty ARI/NMI benchmarking addresses; very low coverage variants may degrade all methods' scores.
- Findings apply to the original (v1) r3fang/SnapATAC; the active successor is SnapATAC2, whose APIs differ.

## Evidence

- [intro] we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues.: "we show that SnapATAC outperforms current methods in accuracy, sensitivity, scalability and reproducibility for cell-type identification from complex tissues."
- [methods] SnapATAC has two components: [Snaptools](https://github.com/r3fang/SnapTools) and [SnapATAC](https://github.com/r3fang/SnapATAC).: "SnapATAC has two components: [Snaptools](https://github.com/r3fang/SnapTools) and [SnapATAC](https://github.com/r3fang/SnapATAC)."
- [readme] R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
