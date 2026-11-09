---
name: neural-network-convergence-monitoring
description: Use when training the SCALE VAE (encoder 3200-1600-800-400, 10-dimensional
  latent layer, single-layer decoder) on scATAC-seq count data and you must verify
  the ELBO decreases and plateaus without divergence before using latent features
  or cluster assignments.
when_to_use_negative:
- Input is already a latent feature table or cluster assignment with no neural network
  training to monitor
- The goal is identifying and removing batch effects or other artifacts from input
  data — SCALE is not specifically designed for artifact removal
- Online integration of multiple heterogeneous single-cell datasets — SCALEX, not
  SCALE, projects datasets into a common cell-embedding space
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SCALE
  role: VAE+GMM method whose training run is monitored; combines the VAE framework
    with a probabilistic Gaussian Mixture Model over 10 latent features
  repo: https://github.com/jsxlei/SCALE
- name: PyTorch
  role: framework in which the SCALE neural network is implemented and the ELBO is
    optimized with Adam
  repo: https://pytorch.org/
- name: scanpy
  role: loads the h5ad peak-by-barcode count matrix and reads back adata.obsm['binary']
    imputed data from adata.h5ad
  repo: https://scanpy.readthedocs.io/en/stable/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:f574b8c8b4edd6caa07dfb8e69ed0585b02063f84698d56587d6940d63b6f156
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - PyTorch
  - SCALE
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# neural-network-convergence-monitoring

## Summary

Monitors the ELBO objective during SCALE's VAE+GMM training on sparse scATAC-seq peak-by-barcode data to confirm the deep generative model converges to a stable optimum, reproducing the paper's optimization behavior before trusting downstream clustering and imputation.

## When to use

Use when training the SCALE VAE (encoder 3200-1600-800-400, 10-dimensional latent layer, single-layer decoder) on scATAC-seq count data and you must verify the ELBO decreases and plateaus without divergence before using latent features or cluster assignments.

## When NOT to use

- Input is already a latent feature table or cluster assignment with no neural network training to monitor
- The goal is identifying and removing batch effects or other artifacts from input data — SCALE is not specifically designed for artifact removal
- Online integration of multiple heterogeneous single-cell datasets — SCALEX, not SCALE, projects datasets into a common cell-embedding space

## Inputs

- peak-by-barcode scATAC-seq count matrix in h5ad format (e.g., Forebrain.h5ad from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad)
- SCALE VAE+GMM training run with recorded per-iteration ELBO values
- training hyperparameters: Adam weight decay 5e-4, learning rate 0.002, batch size 32, random seed 18, max 30,000 iterations

## Outputs

- ELBO convergence curve plotted over iterations
- model.pt saved model checkpoint (reproducible with --pretrain)
- adata.h5ad containing Leiden cluster assignments, 10-dimensional latent feature matrix, and UMAP results
- umap.pdf visualization of 2D UMAP embeddings of each cell

## How to apply

Load the peak-by-barcode count matrix (e.g., Forebrain.h5ad, 2088 cells) with scanpy and build the VAE+GMM in PyTorch: encoder layers 3200-1600-800-400 with ReLU activations, a 10-dimensional latent layer connected to the output via a single-layer decoder with Sigmoid output, and GMM prior parameters mu_c and sigma_c initialized over the latent variables. Optimize the ELBO with the Adam optimizer (weight decay 5e-4, learning rate 0.002, batch size 32, random seed 18) for up to 30,000 iterations. Record the ELBO at every iteration and plot the convergence curve as the primary diagnostic. Convergence is judged by the ELBO decreasing and plateauing before the 30,000-iteration limit with no NaN/Inf losses; only then save the checkpoint as model.pt (usable with --pretrain) and the latent features / cluster assignments as adata.h5ad. Rationale: SCALE's Gaussian Mixture prior over latent variables is meant to give a tighter posterior estimate for sparse scATAC-seq data, so a divergent or non-converging ELBO invalidates the learned latent features.

## Related tools

- **SCALE** (VAE+GMM method whose training run is monitored; combines the VAE framework with a probabilistic Gaussian Mixture Model over 10 latent features) — https://github.com/jsxlei/SCALE
- **PyTorch** (framework in which the SCALE neural network is implemented and the ELBO is optimized with Adam) — https://pytorch.org/
- **scanpy** (loads the h5ad peak-by-barcode count matrix and reads back adata.obsm['binary'] imputed data from adata.h5ad) — https://scanpy.readthedocs.io/en/stable/index.html

## Examples

```
SCALE.py -d Forebrain.h5ad --pretrain model.pt
```

## Evaluation signals

- ELBO decreases and plateaus before the 30,000-iteration limit
- No NaN or Inf losses at any iteration (no divergence)
- Saved model.pt loads successfully and reproduces results with the --pretrain option
- adata.h5ad contains a 10-dimensional latent feature matrix and Leiden cluster assignments
- The reconstructed training run reproduces the paper's optimization behavior (convergence curve shape)

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data
- Explicit modeling of batch effects and technical variations is deferred to future work, not implemented
- Direct applications of scRNA-seq analysis methods to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [intro] A tighter estimation of the posterior distribution could greatly improve the power of VAE in fitting and analyzing sparse data19.: "A tighter estimation of the posterior distribution could greatly improve the power of VAE in fitting and analyzing sparse data19."
- [methods] model.pt: saved model to reproduce results cooperated with option --pretrain: "**model.pt**:  saved model to reproduce results cooperated with option --pretrain"
- [readme] SCALE neural network is implemented in Pytorch framework.: "SCALE neural network is implemented in [Pytorch](https://pytorch.org/) framework."
- [readme] adata.h5ad: saved data including Leiden cluster assignment, latent feature matrix and UMAP results.: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
