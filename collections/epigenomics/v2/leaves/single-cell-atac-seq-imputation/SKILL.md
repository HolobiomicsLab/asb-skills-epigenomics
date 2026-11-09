---
name: single-cell-atac-seq-imputation
description: Use when the input is a sparse, near-binary scATAC-seq peak-by-cell count
  matrix (h5ad, txt/tsv/csv, or mtx folder) and the goal is denoising/imputation of
  missing accessibility values for downstream correlation or clustering evaluation.
when_to_use_negative:
- When batch-effect removal is the primary goal — SCALE is 'not specifically designed
  to identify and remove these artifacts from the input data' and explicit batch modeling
  is deferred to future work
- When imputed values must be binary — use --binary (adata.obsm['binary']) instead;
  --impute yields the numerical matrix and is not the recommended storage-saving mode
- When peak grouping/motif-level resolution suffices and individual peak imputation
  is unnecessary (e.g., chromVAR-style motif analyses that evaluate peaks in groups)
edam_operation: http://edamontology.org/operation_3557
tools:
- name: SCALE
  role: VAE-GMM latent feature model that performs the scATAC-seq denoising and imputation;
    run via SCALE.py with --impute
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Used to read the output adata.h5ad and access the imputed matrix via adata.obsm['imputed'];
    also supplies highly_variable_genes for peak filtering
- name: PyTorch
  role: Framework implementing the SCALE neural network; CUDA recommended when available
- name: scVI
  role: scRNA-seq VAE-based comparison method for benchmarking imputation quality
    via cell-to-meta-cell correlation
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:1dedd9596a273458de6119c4fd09c392adcbefe733c6deb190307f6e1c7d67a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - PyTorch
  - SCALE
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3557
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# single-cell-atac-seq-imputation

## Summary

Impute and denoise sparse single-cell ATAC-seq accessibility matrices using SCALE, a VAE-GMM latent feature model, recovering a numerical imputed matrix stored as scanpy adata.obsm['imputed'] that can be evaluated against per-cell-type meta-cells (e.g., the Forebrain dataset, k=8, 2088 cells).

## When to use

Use when the input is a sparse, near-binary scATAC-seq peak-by-cell count matrix (h5ad, txt/tsv/csv, or mtx folder) and the goal is denoising/imputation of missing accessibility values for downstream correlation or clustering evaluation.

## When NOT to use

- When batch-effect removal is the primary goal — SCALE is 'not specifically designed to identify and remove these artifacts from the input data' and explicit batch modeling is deferred to future work
- When imputed values must be binary — use --binary (adata.obsm['binary']) instead; --impute yields the numerical matrix and is not the recommended storage-saving mode
- When peak grouping/motif-level resolution suffices and individual peak imputation is unnecessary (e.g., chromVAR-style motif analyses that evaluate peaks in groups)

## Inputs

- h5ad file (e.g., Forebrain.h5ad, dense peak-by-cell matrix, 2088 cells, k=8)
- count matrix file (txt/tsv/csv; rows are peaks, columns are barcodes)
- mtx folder containing count (mtx), peak (chr_start_end), and barcode files
- h5mu file (e.g., filename.h5mu/atac)

## Outputs

- adata.h5ad with adata.obsm['imputed'] numerical imputed matrix (same number of cells as input)
- Leiden cluster assignments and latent feature matrix in adata.h5ad
- model.pt saved model (for --pretrain reproduction)
- umap.pdf 2D UMAP embedding visualization

## How to apply

Run the SCALE VAE-GMM pipeline on the input with 'SCALE.py -d [input] --impute' using default parameters (learning rate 0.002, max 30000 iterations, seed 18, min_peaks 100 cell filter, min_cells 3 peak filter); the model learns latent features that fit the sparse distribution of scATAC-seq data, combining a deep generative framework with a probabilistic Gaussian Mixture Model. Extract the numerical imputed values from adata.obsm['imputed'] in the output adata.h5ad (a --binary option binarizes the imputation, but --impute is required for the numerical matrix used in correlation evaluations). To evaluate, build a per-cluster meta-cell by averaging raw accessibility over cells in each cluster (e.g., 8 clusters on Forebrain), then compute cell-wise Pearson correlations between each imputed cell and each meta-cell, confirming each cell correlates most strongly with its assigned cluster's meta-cell (Fig. 3a-style comparison against scImpute, SAVER, MAGIC, and scVI). Outputs are saved in the output folder as adata.h5ad (Leiden cluster assignment, latent feature matrix, UMAP results), model.pt for --pretrain reproduction, and umap.pdf.

## Related tools

- **SCALE** (VAE-GMM latent feature model that performs the scATAC-seq denoising and imputation; run via SCALE.py with --impute) — https://github.com/jsxlei/SCALE
- **scanpy** (Used to read the output adata.h5ad and access the imputed matrix via adata.obsm['imputed']; also supplies highly_variable_genes for peak filtering)
- **PyTorch** (Framework implementing the SCALE neural network; CUDA recommended when available)
- **scVI** (scRNA-seq VAE-based comparison method for benchmarking imputation quality via cell-to-meta-cell correlation)

## Examples

```
SCALE.py -d Forebrain.h5ad --impute -o output
```

## Evaluation signals

- adata.h5ad exists in the output folder and contains an obsm['imputed'] matrix with the same number of cells as the input dataset
- Each imputed cell correlates most strongly (Pearson) with the meta-cell of its assigned cluster, not other clusters' meta-cells
- Reproduces the Fig. 3a cell-type-specific correlation structure and outperforms scImpute, SAVER, MAGIC, and scVI on cell-wise meta-cell correlation
- Latent features and Leiden cluster assignments recover the expected number of clusters when -k is specified (e.g., k=8 for Forebrain)
- Saved model.pt can be reused with --pretrain to reproduce the results

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work
- Direct applications of scRNA-seq methods to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data
- scRNA-seq-based alternatives (scImpute, SAVER, MAGIC, scVI) were the comparison baseline — SCALE's advantage was shown across scATAC-seq datasets from different platforms and qualities, not for scRNA-seq imputation
- The --impute option produces larger output storage than the recommended --binary binarized mode

## Evidence

- [readme] Numerical imputation access and CLI flag: "get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [readme] Output artifact contents: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [discussion] Method scope includes imputation: "SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation."
- [abstract] Superiority claim over other tools: "SCALE substantially outperforms the other tools in all aspects of scATAC-seq data analysis, including visualization, clustering, and denoising and imputation."
- [abstract] Core method rationale: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
