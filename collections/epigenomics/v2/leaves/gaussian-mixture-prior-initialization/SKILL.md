---
name: gaussian-mixture-prior-initialization
description: Use when implementing or reproducing SCALE's VAE+GMM architecture on
  scATAC-seq data (e.g. Forebrain.h5ad peak-by-barcode matrix) and a tight posterior
  estimate over latent features is needed for clustering.
when_to_use_negative:
- Input is a precomputed feature/latent table rather than a raw peak-by-barcode scATAC-seq
  count matrix
- The goal is batch integration of heterogeneous datasets — the article notes SCALE
  is 'not specifically designed to identify and remove these artifacts from the input
  data'; use SCALEX instead
- Data is scRNA-seq; scVI-style single-Gaussian priors were designed for that regime
  and the article warns direct applications to scATAC-seq may be unsuitable due to
  sparsity
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: SCALE
  role: Method that combines the VAE framework with the Gaussian Mixture Model prior;
    run via SCALE.py -d [input]
  repo: https://github.com/jsxlei/SCALE
- name: PyTorch
  role: Framework in which the SCALE neural network and GMM prior (mu_c, sigma_c)
    are implemented and optimized with Adam
- name: scanpy
  role: Used to load the Forebrain.h5ad peak-by-barcode matrix and provide highly_variable_genes
    peak filtering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:9291ad0a353ce580d295f2680be8039c8ad1457dd9e6a0d0cb4f5c090553bb23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - PyTorch
  - SCALE
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gaussian-mixture-prior-initialization

## Summary

Initialize a probabilistic Gaussian Mixture Model (GMM) as the prior over the 10-dimensional latent variables of SCALE's VAE, replacing a single Gaussian prior to better fit the sparse, close-to-binary scATAC-seq peak-by-barcode count data.

## When to use

Use when implementing or reproducing SCALE's VAE+GMM architecture on scATAC-seq data (e.g. Forebrain.h5ad peak-by-barcode matrix) and a tight posterior estimate over latent features is needed for clustering.

## When NOT to use

- Input is a precomputed feature/latent table rather than a raw peak-by-barcode scATAC-seq count matrix
- The goal is batch integration of heterogeneous datasets — the article notes SCALE is 'not specifically designed to identify and remove these artifacts from the input data'; use SCALEX instead
- Data is scRNA-seq; scVI-style single-Gaussian priors were designed for that regime and the article warns direct applications to scATAC-seq may be unsuitable due to sparsity

## Inputs

- Forebrain.h5ad scATAC-seq peak-by-barcode count matrix (Forebrain dataset, 2088 cells)
- h5ad / txt / csv / mtx count matrix files accepted by SCALE.py
- Trained encoder weights for the 10-dimensional latent layer

## Outputs

- Initialized GMM prior parameters mu_c and sigma_c over the 10 latent variables
- model.pt checkpoint of the trained VAE+GMM model
- adata.h5ad with latent feature matrix, Leiden cluster assignments, and UMAP results
- ELBO convergence curve over training iterations

## How to apply

Build the VAE in PyTorch with a four-layer encoder (3200-1600-800-400, ReLU activations) whose latent layer has 10 features, and a single-layer decoder connecting the 10 latent variables directly to all peaks with Sigmoid output. Initialize the GMM prior parameters mu_c and sigma_c over the latent variables rather than using a standard single-Gaussian VAE prior, since a tighter posterior estimate improves the power of the VAE on sparse data. Optimize the ELBO objective with Adam (weight decay 5e-4, learning rate 0.002, batch size 32, random seed 18) for up to 30,000 iterations. Record the ELBO per iteration and confirm it decreases and plateaus before the iteration limit without NaN/Inf divergence; then save the checkpoint as model.pt and latent features/cluster assignments as adata.h5ad.

## Related tools

- **SCALE** (Method that combines the VAE framework with the Gaussian Mixture Model prior; run via SCALE.py -d [input]) — https://github.com/jsxlei/SCALE
- **PyTorch** (Framework in which the SCALE neural network and GMM prior (mu_c, sigma_c) are implemented and optimized with Adam)
- **scanpy** (Used to load the Forebrain.h5ad peak-by-barcode matrix and provide highly_variable_genes peak filtering)

## Examples

```
SCALE.py -d Forebrain.h5ad --pretrain model.pt
```

## Evaluation signals

- ELBO decreases monotonically and plateaus before the 30,000-iteration limit
- No NaN/Inf losses at any iteration (no divergence during ELBO optimization)
- Reconstruction: the 10-dimensional latent layer connects through the single-layer Sigmoid decoder to all peaks
- model.pt can be loaded with --pretrain to reproduce results; adata.h5ad contains Leiden cluster assignments, latent features, and UMAP

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data
- Explicit modeling of batch effects and technical variations is deferred to future work, not implemented
- Hyperparameters (weight decay 5e-4, lr 0.002, batch size 32, seed 18) are dataset-tuned; other datasets may require adjustment

## Evidence

- [abstract] GMM prior over latent variables: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [intro] rationale for tighter posterior: "A tighter estimation of the posterior distribution could greatly improve the power of VAE in fitting and analyzing sparse data19."
- [readme] PyTorch implementation: "SCALE neural network is implemented in [Pytorch](https://pytorch.org/) framework."
- [readme] saved model reproducibility: "**model.pt**:  saved model to reproduce results cooperated with option --pretrain"
- [other] GMM initialization step: [evidence span withheld — anchor status: not_found]
