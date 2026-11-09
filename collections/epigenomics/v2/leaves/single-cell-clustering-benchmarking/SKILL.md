---
name: single-cell-clustering-benchmarking
description: Use when clustering scATAC-seq data (e.g. the Forebrain.h5ad dataset,
  2088 cells, k=8) and you need to compare SCALE against scABC, scVI, cisTopic, or
  SC3 using ARI/NMI/F1 metrics against reference labels.
when_to_use_negative:
- The input is scRNA-seq data requiring online multi-dataset integration — use SCALEX
  instead, which is designed for projecting heterogeneous datasets into a common cell-embedding
  space.
- You need explicit identification and removal of batch effects from the input data
  — SCALE is not specifically designed to identify and remove these artifacts.
- chromVAR-style motif-level analysis of peak groups rather than whole-dataset cell
  clustering is the goal.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SCALE
  role: 'Primary method: VAE combined with a probabilistic Gaussian Mixture Model
    to learn latent features from sparse scATAC-seq data and assign Leiden clusters;
    run via SCALE.py with -k for known cluster number.'
  repo: https://github.com/jsxlei/SCALE
- name: scABC
  role: Baseline that weighs cells by sequencing depth and applies weighted K-medoid
    clustering to reduce the impact of missing values.
- name: scVI
  role: Baseline scRNA-seq tool that learns latent representation via a hierarchical
    Bayesian VAE; applied to the Forebrain count matrix for comparison.
- name: cisTopic
  role: Baseline that applies latent Dirichlet allocation to scATAC-seq data to identify
    cis-regulatory topics.
- name: SC3
  role: Baseline scRNA-seq clustering tool included in the Forebrain benchmark comparison.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:500fa3635e832b81e1c065c1f95caa9db960638ebe5773c933c557b26b776dce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SC3
  - SCALE
  - cisTopic
  - scABC
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-clustering-benchmarking

## Summary

Benchmarks SCALE — a VAE+Gaussian Mixture Model method for single-cell ATAC-seq latent feature extraction — against scABC, scVI, cisTopic, and SC3 clustering tools, scoring cluster assignments against reference cell-type labels with ARI, NMI, and micro-F1.

## When to use

Use when clustering scATAC-seq data (e.g. the Forebrain.h5ad dataset, 2088 cells, k=8) and you need to compare SCALE against scABC, scVI, cisTopic, or SC3 using ARI/NMI/F1 metrics against reference labels.

## When NOT to use

- The input is scRNA-seq data requiring online multi-dataset integration — use SCALEX instead, which is designed for projecting heterogeneous datasets into a common cell-embedding space.
- You need explicit identification and removal of batch effects from the input data — SCALE is not specifically designed to identify and remove these artifacts.
- chromVAR-style motif-level analysis of peak groups rather than whole-dataset cell clustering is the goal.

## Inputs

- Forebrain.h5ad dense-matrix scATAC-seq dataset (2088 cells, k=8 reference clusters) from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad
- Count matrix with peaks as rows and barcodes as columns (txt/tsv/csv or mtx folder with count, peak, and barcode files)
- Reference cell-type labels for the Forebrain dataset
- Optionally, corrupted versions of the count matrix with randomly dropped peak values (10–90%)

## Outputs

- model.pt — saved SCALE model for reproduction with --pretrain
- adata.h5ad — annotated data with Leiden cluster assignment, latent feature matrix, and UMAP results
- umap.pdf — 2D UMAP embedding of each cell
- forebrain_benchmark_metrics.csv — ARI, NMI, and micro-F1 for SCALE, scABC, scVI, cisTopic, SC3
- ari_robustness.csv — ARI as a function of corruption level for SCALE, scABC, and scVI

## How to apply

Download the dense-matrix Forebrain.h5ad (2088 cells, k=8) from the SCALE source-data repository and load it with scanpy/anndata. Run SCALE.py on the h5ad input with -k 8 using package defaults (initial learning rate 0.002, max_iter 30000, random seed 18) so the VAE+GMM model learns latent features and assigns Leiden clusters, saving model.pt and adata.h5ad. Run the baselines — scABC (weighted K-medoid clustering weighted by sequencing depth), scVI (VAE adapted for scRNA-seq), cisTopic (latent Dirichlet allocation), and SC3 — on the same Forebrain count matrix. Compute ARI, NMI, and micro-F1 between each method's assignments and the Forebrain reference labels with scikit-learn, then assemble a metrics table (e.g. forebrain_benchmark_metrics.csv). For robustness analysis, randomly drop peak values at graded corruption levels (10–90%), rerun SCALE and baselines per level, and tabulate ARI as a function of corruption (SCALE 0.668 raw → 0.448 at 30% corruption; scABC 0.315 → 0.222; scVI 0.448 → 0.388).

## Related tools

- **SCALE** (Primary method: VAE combined with a probabilistic Gaussian Mixture Model to learn latent features from sparse scATAC-seq data and assign Leiden clusters; run via SCALE.py with -k for known cluster number.) — https://github.com/jsxlei/SCALE
- **scABC** (Baseline that weighs cells by sequencing depth and applies weighted K-medoid clustering to reduce the impact of missing values.)
- **scVI** (Baseline scRNA-seq tool that learns latent representation via a hierarchical Bayesian VAE; applied to the Forebrain count matrix for comparison.)
- **cisTopic** (Baseline that applies latent Dirichlet allocation to scATAC-seq data to identify cis-regulatory topics.)
- **SC3** (Baseline scRNA-seq clustering tool included in the Forebrain benchmark comparison.)

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --lr 0.002 --max_iter 30000 --seed 18 -o forebrain_output
```

## Evaluation signals

- Metrics table contains ARI, NMI, and micro-F1 rows for all five methods (SCALE, scABC, scVI, cisTopic, SC3).
- SCALE's ARI on raw Forebrain data reproduces the reported value of 0.668, the highest among compared tools (scABC 0.315, scVI 0.448).
- SCALE's ranking in the clustering comparison (Fig. 2b) matches the paper: SCALE outperforms all other tools; SC3 collapses most cells into one group and scVI misclassifies three subgroups.
- Robustness trajectory matches reported degradation: SCALE ARI 0.668 → 0.448 at 30% corruption, scABC 0.315 → 0.222, scVI 0.448 → 0.388, with SCALE retaining highest ARI at each level.
- SCALE outputs model.pt, adata.h5ad (with Leiden cluster assignment, latent features, UMAP), and umap.pdf are present for each run.

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work.
- scABC heavily depends on landmark samples with high sequencing depths; Spearman rank correlation can be ill-defined for data with many missing values.
- Direct applications of scRNA-seq methods (scVI, SC3) to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data.
- Results depend on random seed (default 18) and defaults such as min_peaks=100, min_cells=3, n_feature=100,000, lr=0.002, max_iter=30000.

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [intro] scABC weighting and K-medoid: "Another method, scABC, weighs cells by sequencing depth and applies weighted K-medoid clustering to reduce the impact of missing values"
- [readme] SCALE.py options and defaults: "[--max_iter] or [-i]: max iteration number, default is 30000"
- [discussion] Benchmark outcome vs scABC and scRNA-seq tools: "In all comparisons, SCALE performs much more favorably than scABC and scRNA-seq tools."
- [discussion] SCALE batch-effect limitation: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
