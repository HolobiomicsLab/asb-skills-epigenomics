---
name: ari-clustering-agreement-evaluation
description: Use when a clustering method's output assignments must be compared against
  known cell-type labels on single-cell data (e.g., scATAC-seq Forebrain, k=8), or
  when ARI degradation under graded input corruption must be tabulated across tools.
when_to_use_negative:
- When no ground-truth cell labels exist for the dataset — ARI cannot be computed
  against unknown clusters
- When the goal is batch-effect removal or data integration rather than clustering
  accuracy assessment — SCALE is 'not specifically designed to identify and remove
  batch effects' and ARI alone does not measure integration quality
edam_operation: http://edamontology.org/operation_2939
edam_topics: []
tools:
- name: SCALE
  role: Produces Leiden cluster assignments in adata.h5ad that are scored by ARI;
    run per corruption level with -k 8, lr 0.002, max_iter 30000, seed 18
  repo: https://github.com/jsxlei/SCALE
- name: scABC
  role: Baseline clustering tool whose assignments are scored by ARI for comparison
    under sparsity corruption
- name: scVI
  role: Baseline latent-representation clustering tool whose assignments are scored
    by ARI for comparison under sparsity corruption
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:bd27395d276df48d5e03510cfa6bae3380faa142d60448fe16e4e5ca65d9e7e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - scABC
  - scVI
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ari-clustering-agreement-evaluation

## Summary

Compute the Adjusted Rand Index (ARI) between algorithm cluster assignments (e.g., Leiden clusters stored in SCALE's adata.h5ad output) and known cell labels to quantify clustering accuracy, optionally across graded corruption levels to measure robustness to data sparsity.

## When to use

Use when a clustering method's output assignments must be compared against known cell-type labels on single-cell data (e.g., scATAC-seq Forebrain, k=8), or when ARI degradation under graded input corruption must be tabulated across tools.

## When NOT to use

- When no ground-truth cell labels exist for the dataset — ARI cannot be computed against unknown clusters
- When the goal is batch-effect removal or data integration rather than clustering accuracy assessment — SCALE is 'not specifically designed to identify and remove batch effects' and ARI alone does not measure integration quality

## Inputs

- adata.h5ad file from SCALE containing Leiden cluster assignments per cell
- Known Forebrain cell labels (ground-truth cell-type annotations, k=8 clusters, 2088 cells)
- Corrupted count matrices at graded peak-dropping levels (10%–90% removal)

## Outputs

- ARI value per clustering run (cluster assignments vs. known labels)
- Tabulated ARI-vs-corruption-level comparison across tools (ari_robustness.csv)

## How to apply

For each run, load the adata.h5ad output and extract the Leiden cluster assignment per cell, then compute the Adjusted Rand Index between these assignments and the known Forebrain cell labels. When evaluating robustness, generate random peak-dropping corruptions of the dense count matrix at graded levels (10%–90%, including the 30% level used in the reference result), rerun clustering per level with fixed parameters (-k 8, lr 0.002, max_iter 30000, seed 18), and compute ARI at each level. Repeat identically for baselines (scABC, scVI) so ARI trajectories are comparable, then tabulate ARI vs. corruption level (e.g., ari_robustness.csv). Correct application is judged by reproducing the reference trajectory: SCALE ARI 0.668 on raw Forebrain dropping to 0.448 at 30% corruption, with SCALE retaining the highest ARI at each level versus scABC (0.315 → 0.222) and scVI (0.448 → 0.388).

## Related tools

- **SCALE** (Produces Leiden cluster assignments in adata.h5ad that are scored by ARI; run per corruption level with -k 8, lr 0.002, max_iter 30000, seed 18) — https://github.com/jsxlei/SCALE
- **scABC** (Baseline clustering tool whose assignments are scored by ARI for comparison under sparsity corruption)
- **scVI** (Baseline latent-representation clustering tool whose assignments are scored by ARI for comparison under sparsity corruption)

## Examples

```
SCALE.py -d Forebrain_corrupt30.h5ad -k 8 --lr 0.002 --max_iter 30000 --seed 18 -o run_30pct; # then compute ARI(adata.obs['leiden'], true_labels)
```

## Evaluation signals

- Raw-data SCALE ARI on Forebrain should approximate 0.668 (k=8, 2088 cells)
- At 30% peak corruption, SCALE ARI should be ~0.448, scABC ~0.222, scVI ~0.388
- SCALE should retain the highest ARI at every corruption level relative to scABC and scVI
- ARI is computed between Leiden assignments in each adata.h5ad and the known Forebrain labels — verify cell counts (2088) match between assignments and labels
- Identical corruption-and-clustering protocol must be applied to all tools before ARI comparison

## Limitations

- ARI drops substantially as input sparsity increases (SCALE 0.668 → 0.448 at 30% corruption), so results are sensitive to the corruption level applied
- Comparative robustness conclusions depend on applying the identical corruption and clustering protocol to every tool
- SCALE itself is not designed to remove batch effects or technical artifacts from input data, which can affect clustering assignments

## Evidence

- [results] SCALE's ARI dropped from 0.668 (raw data) to 0.448 at 30% corruption: [evidence span withheld — anchor status: not_found]
- [readme] Leiden cluster assignments stored in adata.h5ad: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [readme] SCALE clustering hyperparameters for reproduction: "[--lr]: modify the initial learning rate, default is 0.002: 
* [--max_iter] or [-i]: max iteration number, default is 30000  
* [--seed]: random seed for parameter initialization, default is 18"
- [readme] Forebrain dataset description: "**[Tutorial Forebrain](https://github.com/jsxlei/SCALE/wiki/Forebrain)**   Run SCALE on dense matrix **Forebrain** dataset (k=8, 2088 cells)"
- [discussion] SCALE outperforms comparators in clustering: "In all comparisons, SCALE performs much more favorably than scABC and scRNA-seq tools."
