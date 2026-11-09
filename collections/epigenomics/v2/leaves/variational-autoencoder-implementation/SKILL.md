---
name: variational-autoencoder-implementation
description: Use when scATAC-seq peak-by-barcode count data (e.g. Forebrain.h5ad,
  2088 cells) are sparse and high-dimensional with severe missingness, and clustering/visualization/imputation
  requires learning low-dimensional latent features via a VAE with GMM prior.
when_to_use_negative:
- The goal is explicit identification and removal of batch effects or other technical
  artifacts — the paper states SCALE 'is not specifically designed to identify and
  remove these artifacts from the input data'
- Integrating heterogeneous scRNA-seq and scATAC-seq datasets online — use SCALEX,
  the dedicated integration tool, instead
- Input is already a latent/feature-reduced table, so dimensionality reduction via
  VAE would add no value
edam_operation: http://edamontology.org/operation_3935
tools:
- name: SCALE
  role: The VAE+GMM implementation itself; installed via pip and run as SCALE.py on
    h5ad/count-matrix inputs to produce model.pt, adata.h5ad, and umap.pdf
  repo: https://github.com/jsxlei/SCALE
- name: PyTorch
  role: Framework in which the SCALE neural network (encoder 3200-1600-800-400, 10-dim
    latent, single-layer decoder) is implemented and trained with Adam
  repo: https://pytorch.org/
- name: scanpy
  role: Used to load the h5ad peak-by-barcode matrix and to filter peaks via highly_variable_genes
  repo: https://scanpy.readthedocs.io/en/stable/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:9756427d84515e6fe97602bd9e33a27de764dafa49c0ecb3ec40c36dc6da2395
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - PyTorch
  - SCALE
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# variational-autoencoder-implementation

## Summary

Implement SCALE's VAE+GMM neural network for sparse scATAC-seq: a PyTorch variational autoencoder with a Gaussian Mixture Model prior over 10 latent features that learns representations for clustering, visualization, and imputation of peak-by-barcode data.

## When to use

Use when scATAC-seq peak-by-barcode count data (e.g. Forebrain.h5ad, 2088 cells) are sparse and high-dimensional with severe missingness, and clustering/visualization/imputation requires learning low-dimensional latent features via a VAE with GMM prior.

## When NOT to use

- The goal is explicit identification and removal of batch effects or other technical artifacts — the paper states SCALE 'is not specifically designed to identify and remove these artifacts from the input data'
- Integrating heterogeneous scRNA-seq and scATAC-seq datasets online — use SCALEX, the dedicated integration tool, instead
- Input is already a latent/feature-reduced table, so dimensionality reduction via VAE would add no value

## Inputs

- Forebrain.h5ad peak-by-barcode scATAC-seq count matrix (2088 cells)
- count matrix file in txt/tsv/csv format (rows=peaks, columns=barcodes)
- mtx folder with count, peak (chr_start_end), and barcode files
- h5ad or h5mu (multiomics, e.g. filename.h5mu/atac) anndata file

## Outputs

- model.pt: saved model checkpoint for reproduction with --pretrain
- adata.h5ad: data with Leiden cluster assignment, latent feature matrix, and UMAP results
- umap.pdf: 2d UMAP embeddings of each cell
- ELBO convergence curve across training iterations

## How to apply

Download the peak-by-barcode count matrix (e.g. Forebrain.h5ad, 2088 cells) and load it with scanpy, optionally filtering peaks with scanpy's highly_variable_genes and the --min_peaks/--min_cells/--n_feature options. Build the VAE in PyTorch: a four-layer encoder (3200-1600-800-400) with ReLU activations, a 10-dimensional latent layer, and a single-layer decoder connecting the 10 latent variables directly to all peaks with Sigmoid output; initialize the GMM prior parameters mu_c and sigma_c over the latent variables. Optimize the ELBO with Adam (learning rate 0.002, weight decay 5e-4, batch size 32, random seed 18) for up to 30,000 iterations, recording the ELBO each iteration to confirm convergence. The rationale is that a tighter posterior estimate from the GMM prior improves VAE fitting of sparse, close-to-binary scATAC-seq data compared to a standard single-Gaussian prior. Save the trained checkpoint as model.pt (reusable via --pretrain) and the latent features/Leiden cluster assignments as adata.h5ad, plus a 2d UMAP embedding (umap.pdf).

## Related tools

- **SCALE** (The VAE+GMM implementation itself; installed via pip and run as SCALE.py on h5ad/count-matrix inputs to produce model.pt, adata.h5ad, and umap.pdf) — https://github.com/jsxlei/SCALE
- **PyTorch** (Framework in which the SCALE neural network (encoder 3200-1600-800-400, 10-dim latent, single-layer decoder) is implemented and trained with Adam) — https://pytorch.org/
- **scanpy** (Used to load the h5ad peak-by-barcode matrix and to filter peaks via highly_variable_genes) — https://scanpy.readthedocs.io/en/stable/index.html

## Examples

```
SCALE.py -d Forebrain.h5ad --binary True
```

## Evaluation signals

- ELBO decreases monotonically and plateaus before the 30,000-iteration limit with no divergence (NaN/Inf losses)
- Trained model.pt is loadable and reproducible via the --pretrain option
- adata.h5ad contains a 10-dimensional latent feature matrix, Leiden cluster assignments, and UMAP results
- Latent features directly link to cell populations (interpretable per-cluster features)
- Binary imputed data recoverable from adata.obsm['binary'] when run with --binary

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work
- Results depend on fixed hyperparameters (lr 0.002, weight decay 5e-4, batch size 32, seed 18); GPU/CUDA execution is recommended and CPU may be slow
- Direct application of scRNA-seq analysis methods to scATAC-seq data is unsuitable due to the close-to-binary nature and increased sparsity of the data, so the VAE must model this sparsity (GMM prior)

## Evidence

- [intro] SCALE combines the VAE framework with the GMM prior over latent variables: "Here, we introduce SCALE (Single-Cell ATAC-seq analysis via Latent feature Extraction), a method that combines the VAE framework with the Gaussian Mixture Model"
- [methods] SCALE is implemented in PyTorch and installed from GitHub: "pip install git+https://github.com/jsxlei/SCALE.git"
- [readme] PyTorch framework and CUDA recommendation from the README: "SCALE neural network is implemented in [Pytorch](https://pytorch.org/) framework."
- [readme] Output artifacts of running SCALE: "**model.pt**:  saved model to reproduce results cooperated with option --pretrain"
- [intro] Motivation for modeling sparse scATAC-seq data: "Direct applications of these scRNA-seq analysis methods to scATAC-seq data, however, may not be suitable due to the close-to-binary nature and increased sparsity of the data"
- [discussion] Batch-effect limitation: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
