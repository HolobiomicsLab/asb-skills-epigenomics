---
name: vae-gmm-model-training
description: Use when you need to cluster or embed a sparse, near-binary scATAC-seq
  peak-by-cell matrix and want a VAE+GMM latent representation instead of scRNA-seq
  tools like scVI or weighted K-medoid methods like scABC.
when_to_use_negative:
- The input is already a latent/feature table or a scRNA-seq expression matrix analyzed
  with batch-corrected integration — use SCALEX or scRNA-seq tools instead
- The task explicitly requires identifying and removing batch effects or technical
  artifacts, since SCALE is not specifically designed for this (deferred to future
  work)
- The dataset lacks the minimum quality threshold (fewer than 100 valid peaks per
  cell with default --min_peaks), indicating low-quality cells unsuitable for training
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SCALE
  role: Trains the VAE+GMM model on scATAC-seq data to learn latent features and assign
    Leiden clusters; installed via pip from github.com/jsxlei/SCALE
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Provides highly_variable_genes used to filter peaks and anndata for loading
    the h5ad input
- name: SCALEX
  role: Alternative online integration tool for scRNA-seq and scATAC-seq when projecting
    heterogeneous datasets into a common cell-embedding space
  repo: https://github.com/jsxlei/SCALEX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:c154c0235a11458a511b7a88e6a222e2304ef3c58589ef049f735ab4f3b1cc8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  - SCALEX
  - cisTopic
  - scABC
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# vae-gmm-model-training

## Summary

Train SCALE, a VAE combined with a probabilistic Gaussian Mixture Model, on sparse scATAC-seq count data to learn latent features that support clustering, visualization, and imputation. The skill reproduces the paper's Forebrain benchmark workflow (h5ad input, k=8, seed 18).

## When to use

Use when you need to cluster or embed a sparse, near-binary scATAC-seq peak-by-cell matrix and want a VAE+GMM latent representation instead of scRNA-seq tools like scVI or weighted K-medoid methods like scABC.

## When NOT to use

- The input is already a latent/feature table or a scRNA-seq expression matrix analyzed with batch-corrected integration — use SCALEX or scRNA-seq tools instead
- The task explicitly requires identifying and removing batch effects or technical artifacts, since SCALE is not specifically designed for this (deferred to future work)
- The dataset lacks the minimum quality threshold (fewer than 100 valid peaks per cell with default --min_peaks), indicating low-quality cells unsuitable for training

## Inputs

- scATAC-seq h5ad file (e.g., Forebrain.h5ad, 2088 cells, k=8 clusters)
- peak-by-cell count matrix in txt/tsv/csv (row=peak, column=barcode) or mtx folder with count, peak (chr_start_end), and barcode files
- h5mu multiomics file (e.g., filename.h5mu/atac)
- known cluster number k (e.g., -k 8)

## Outputs

- model.pt saved model for reproduction with --pretrain
- adata.h5ad with Leiden cluster assignments, latent feature matrix, and UMAP results
- umap.pdf 2D UMAP embedding of each cell
- imputed data in adata.obsm['binary'] or adata.obsm['imputed'] when --binary/--impute is set

## How to apply

Install SCALE via pip (`pip install scale`) or from GitHub (github.com/jsxlei/SCALE), which is implemented in PyTorch (CUDA recommended). Download the dataset as an h5ad file (e.g., Forebrain.h5ad, 2088 cells, k=8 clusters) or supply a peak-by-cell count matrix in txt/tsv/csv or mtx folder format, then run `SCALE.py -d [input] -k 8` using package defaults (initial learning rate 0.002, max 30000 iterations, random seed 18). SCALE applies quality filtering (`--min_peaks` default 100 cells, `--min_cells` default 3, `--n_feature` default 100,000 highly variable peaks selected with scanpy's highly_variable_genes) before training the VAE with a Gaussian Mixture Model prior on the latent space. Outputs are written to an output folder: model.pt (for reproduction with --pretrain), adata.h5ad containing Leiden cluster assignments, the latent feature matrix, and UMAP results, and umap.pdf. Add `--binary` or `--impute` to obtain binary or numerical imputed data in adata.obsm['binary'] / adata.obsm['imputed'].

## Related tools

- **SCALE** (Trains the VAE+GMM model on scATAC-seq data to learn latent features and assign Leiden clusters; installed via pip from github.com/jsxlei/SCALE) — https://github.com/jsxlei/SCALE
- **scanpy** (Provides highly_variable_genes used to filter peaks and anndata for loading the h5ad input)
- **SCALEX** (Alternative online integration tool for scRNA-seq and scATAC-seq when projecting heterogeneous datasets into a common cell-embedding space) — https://github.com/jsxlei/SCALEX

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --outdir output
```

## Evaluation signals

- Output folder contains model.pt, adata.h5ad, and umap.pdf as documented in the README
- adata.h5ad includes Leiden cluster assignment, latent feature matrix, and UMAP results
- With -k 8 on Forebrain.h5ad (2088 cells), cluster assignments should match the reference cell types better than scABC, scVI, cisTopic, and SC3 by ARI, NMI, and micro-F1 (SCALE ranks first as in Fig. 2b)
- Saved model.pt can be reloaded with --pretrain to reproduce results
- Default hyperparameters (lr=0.002, max_iter=30000, seed=18) are respected in the run configuration

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work
- chromVAR lacks individual peak resolution and scABC depends on landmark samples with high sequencing depths — motivation for SCALE, but also a reminder that alternative methods may be needed for motif-level analysis
- Direct application of scRNA-seq methods to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data
- Recent versions replaced the --min_cells default (0.01) with --n_feature (default 100,000 highly variable peaks); use --n_feature -1 to disable peak filtering

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [readme] Run command and options: "[--lr]: modify the initial learning rate, default is 0.002: [--max_iter] or [-i]: max iteration number, default is 30000 [--seed]: random seed for parameter initialization, default is 18"
- [readme] Filtering parameters: "[--min_peaks]: filter low quality cells by valid peaks number, default 100 [--min_cells]: filter low quality peaks by valid cells number, default 3"
- [readme] Output artifacts: "**model.pt**: saved model to reproduce results cooperated with option --pretrain **adata.h5ad**: saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [methods] Forebrain dataset and tutorial parameters: "[Forebrain](http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad)"
