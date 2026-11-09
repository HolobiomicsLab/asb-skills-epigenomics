---
name: vae-latent-feature-learning
description: Use when the input is a raw scATAC-seq peak-by-cell count matrix (h5ad,
  txt/tsv/csv, or mtx folder with count/peak/barcode files) suffering from 'missingness',
  and the goal is clustering, 2D embedding, or imputation of individual peaks.
when_to_use_negative:
- The task requires explicit identification and removal of batch effects or other
  technical artifacts — the article states SCALE is not specifically designed to identify
  and remove these artifacts from the input data
- The input is already a feature/latent embedding table rather than a raw peak-by-cell
  count matrix
- Peak-group-level analysis without individual peak resolution is sufficient (chromVAR
  may be more appropriate)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3173
tools:
- name: SCALE
  role: Implements the VAE-GMM latent feature model trained on scATAC-seq count matrices;
    provides clustering, UMAP embedding, and imputation via SCALE.py
  repo: https://github.com/jsxlei/SCALE
- name: PyTorch
  role: Framework in which the SCALE neural network is implemented; CUDA recommended
  repo: https://pytorch.org/
- name: scanpy
  role: Provides highly_variable_genes for peak filtering and the anndata obsm access
    to imputed matrices in adata.h5ad
  repo: https://scanpy.readthedocs.io/en/stable/index.html
- name: scVI
  role: Baseline VAE-based scRNA-seq tool adapted for comparison; uses a standard
    single Gaussian prior rather than SCALE's GMM prior
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:777b05c82d985efa9cf1c5a400642dc93dc3e49d16e029989534f1798dc9f8e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - PyTorch
  - SCALE
  - scABC
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# vae-latent-feature-learning

## Summary

Learn low-dimensional latent features from sparse, high-dimensional scATAC-seq count matrices using a VAE whose latent variables are given a probabilistic Gaussian Mixture Model prior (SCALE), enabling visualization, Leiden clustering, and denoising/imputation of missing accessibility values.

## When to use

Use when the input is a raw scATAC-seq peak-by-cell count matrix (h5ad, txt/tsv/csv, or mtx folder with count/peak/barcode files) suffering from 'missingness', and the goal is clustering, 2D embedding, or imputation of individual peaks.

## When NOT to use

- The task requires explicit identification and removal of batch effects or other technical artifacts — the article states SCALE is not specifically designed to identify and remove these artifacts from the input data
- The input is already a feature/latent embedding table rather than a raw peak-by-cell count matrix
- Peak-group-level analysis without individual peak resolution is sufficient (chromVAR may be more appropriate)

## Inputs

- scATAC-seq peak-by-cell count matrix as h5ad file
- count matrix file in txt/tsv (tab-separated) or csv format with peaks as rows and barcodes as columns
- mtx folder containing a count file (mtx format), a peak file (chr_start_end), and a barcode file
- h5mu multiomics file (e.g. filename.h5mu/atac)

## Outputs

- model.pt saved model for --pretrain reproduction
- adata.h5ad containing Leiden cluster assignment, latent feature matrix, and UMAP results
- umap.pdf 2D UMAP embedding visualization
- numerical imputed data in adata.obsm['imputed'] (with --impute) or binary imputed data in adata.obsm['binary'] (with --binary)

## How to apply

Install SCALE via pip (or from github.com/jsxlei/SCALE) and run SCALE.py on the input count matrix; the model combines a VAE with a Gaussian Mixture Model prior to fit the close-to-binary, sparse scATAC-seq distribution. Set the cluster number with -k when known (e.g. k=8 for the Forebrain dataset with 2088 cells), and keep default hyperparameters (initial learning rate 0.002, max_iter 30000, seed 18) for reproducibility; peaks can be pre-filtered with --min_peaks (default 100 valid peaks per cell), --min_cells (default 3 valid cells per peak) or --n_feature (highly variable features, default 100,000). The output folder contains model.pt (for --pretrain reuse), adata.h5ad with Leiden cluster assignments, latent features and UMAP results, and umap.pdf. For imputation, add --impute to obtain numerical imputed values in adata.obsm['imputed'] (or --binary for binarized values recommended to save storage), then evaluate by averaging raw accessibility per cluster into meta-cells and computing cell-wise Pearson correlations against them.

## Related tools

- **SCALE** (Implements the VAE-GMM latent feature model trained on scATAC-seq count matrices; provides clustering, UMAP embedding, and imputation via SCALE.py) — https://github.com/jsxlei/SCALE
- **PyTorch** (Framework in which the SCALE neural network is implemented; CUDA recommended) — https://pytorch.org/
- **scanpy** (Provides highly_variable_genes for peak filtering and the anndata obsm access to imputed matrices in adata.h5ad) — https://scanpy.readthedocs.io/en/stable/index.html
- **scVI** (Baseline VAE-based scRNA-seq tool adapted for comparison; uses a standard single Gaussian prior rather than SCALE's GMM prior)

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --impute --lr 0.002 --max_iter 30000 --seed 18
```

## Evaluation signals

- adata.h5ad exists and contains an obsm['imputed'] matrix with the same number of cells as the input
- With known cluster number (e.g. Forebrain k=8, 2088 cells), Leiden cluster assignments recover the expected number of clusters
- Each imputed cell correlates most strongly with the meta-cell (cluster-averaged raw accessibility) of its assigned cluster (Fig. 3a-style Pearson correlation structure)
- ARI against known cell labels matches reported trajectories (e.g. Forebrain SCALE ARI 0.668 raw, 0.448 at 30% peak corruption)
- model.pt from a run successfully reproduces results when used with the --pretrain option

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work
- scATAC-seq 'missingness' means each open chromatin site has only one or two capture opportunities per diploid cell, yielding only a few thousand distinct reads per cell
- Default --min_cells filtering (3, previously 0.01) was replaced by --n_feature (default 100,000 highly variable features); use --n_feature -1 to disable

## Evidence

- [abstract] VAE-GMM prior motivation: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [intro] Method definition: "Here, we introduce SCALE (Single-Cell ATAC-seq analysis via Latent feature Extraction), a method that combines the VAE framework with the Gaussian Mixture Model"
- [readme] Imputation output location: "or get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [readme] Default hyperparameters: "[--lr]: modify the initial learning rate, default is 0.002:"
- [methods] Output artifacts: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [discussion] Batch effect caveat: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
