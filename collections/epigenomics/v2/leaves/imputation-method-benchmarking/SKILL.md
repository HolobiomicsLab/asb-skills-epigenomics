---
name: imputation-method-benchmarking
description: Use when evaluating whether an imputation method recovers cell-type-specific
  signal in sparse scATAC-seq data, e.g. comparing SCALE's adata.obsm['imputed'] matrix
  against scImpute, SAVER, MAGIC or scVI via meta-cell correlation.
when_to_use_negative:
- The goal is explicit identification and removal of batch effects or other technical
  artifacts — the paper states SCALE 'is not specifically designed to identify and
  remove these artifacts from the input data'
- Input is scRNA-seq data without an scATAC-seq sparsity/binary structure concern;
  the benchmark rationale rests on scATAC-seq 'missingness' and close-to-binary data
- Only a small preview of denoised values is needed — use the storage-efficient --binary
  option instead of numerical --impute imputation
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SCALE
  role: Primary imputation method; run with --impute to write numerical imputed values
    to adata.obsm['imputed'] in the output adata.h5ad
  repo: https://github.com/jsxlei/SCALE
- name: PyTorch
  role: Framework in which SCALE's VAE-GMM neural network is implemented
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Reads the output adata.h5ad and provides adata.obsm['imputed'] access and
    highly_variable_genes peak filtering
  repo: https://scanpy.readthedocs.io/en/stable/index.html
- name: scVI
  role: Comparison imputation method (scRNA-seq VAE tool) benchmarked against SCALE's
    imputed cells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:e69240f037c808e4ef099394e71a1a9c6c3842d9b712f704713773737380ee8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - PyTorch
  - SCALE
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# imputation-method-benchmarking

## Summary

Benchmark denoising/imputation of sparse single-cell data by running SCALE's VAE-GMM imputation (--impute) on a scATAC-seq dataset such as Forebrain and evaluating cell-wise Pearson correlations against per-cell-type meta-cells, in comparison with scRNA-seq methods (scImpute, SAVER, MAGIC, scVI).

## When to use

Use when evaluating whether an imputation method recovers cell-type-specific signal in sparse scATAC-seq data, e.g. comparing SCALE's adata.obsm['imputed'] matrix against scImpute, SAVER, MAGIC or scVI via meta-cell correlation.

## When NOT to use

- The goal is explicit identification and removal of batch effects or other technical artifacts — the paper states SCALE 'is not specifically designed to identify and remove these artifacts from the input data'
- Input is scRNA-seq data without an scATAC-seq sparsity/binary structure concern; the benchmark rationale rests on scATAC-seq 'missingness' and close-to-binary data
- Only a small preview of denoised values is needed — use the storage-efficient --binary option instead of numerical --impute imputation

## Inputs

- Forebrain.h5ad scATAC-seq count matrix (dense matrix, 2088 cells, k=8 clusters) from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad
- Output adata.h5ad containing adata.obsm['imputed'] numerical imputation matrix
- Imputed matrices from alternative methods (scImpute, SAVER, MAGIC, scVI)

## Outputs

- Cell-wise Pearson correlation matrix between imputed single cells and per-cluster meta-cells
- Per-cell-type correlation comparison showing SCALE vs. scImpute/SAVER/MAGIC/scVI (Fig. 3a-style result)
- SCALE output adata.h5ad with Leiden cluster assignments, latent feature matrix, and imputed data

## How to apply

Download the dense Forebrain.h5ad dataset (k=8, 2088 cells) and run SCALE with 'SCALE.py -d [input] --impute' using default parameters (learning rate 0.002, 30000 max iterations, seed 18) so the VAE-GMM latent feature model writes numerical imputed values to adata.obsm['imputed'] in the output adata.h5ad. Build a per-cluster meta-cell by averaging raw accessibility over cells within each of the 8 clusters, then compute cell-wise Pearson correlations between each imputed cell and each meta-cell, reproducing the Fig. 3a evaluation. Run the same meta-cell correlation protocol on alternative imputations (e.g. scVI) and compare the recovered cell-type-specific correlation structure. A run is successful when each imputed cell correlates most strongly with the meta-cell of its assigned cluster, and SCALE attains the highest correlation across cell types relative to the competing methods.

## Related tools

- **SCALE** (Primary imputation method; run with --impute to write numerical imputed values to adata.obsm['imputed'] in the output adata.h5ad) — https://github.com/jsxlei/SCALE
- **PyTorch** (Framework in which SCALE's VAE-GMM neural network is implemented) — https://github.com/jsxlei/SCALE
- **scanpy** (Reads the output adata.h5ad and provides adata.obsm['imputed'] access and highly_variable_genes peak filtering) — https://scanpy.readthedocs.io/en/stable/index.html
- **scVI** (Comparison imputation method (scRNA-seq VAE tool) benchmarked against SCALE's imputed cells)

## Examples

```
SCALE.py -d Forebrain.h5ad --impute --lr 0.002 -i 30000 --seed 18 -k 8
```

## Evaluation signals

- adata.h5ad exists in the output folder and adata.obsm['imputed'] has the same number of cells (2088) as the input Forebrain dataset
- Each imputed cell correlates most strongly with the meta-cell of its assigned cluster (one of the 8 clusters)
- SCALE's per-cell-type meta-cell correlations exceed those of scImpute, SAVER, MAGIC, and scVI across the Forebrain dataset
- Default run parameters were respected: lr 0.002, max_iter 30000, seed 18
- Reproduced correlation structure matches the Fig. 3a pattern of highest correlation of single cells with the corresponding meta-cell for each cell type

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit batch-effect modeling is deferred to future work
- Numerical imputation via --impute consumes storage; the README recommends --binary for saving storage
- Evaluation depends on known cluster structure (k=8 for Forebrain); imputed values are cell-to-meta-cell correlations rather than absolute accessibility recovery

## Evidence

- [readme] Run SCALE with --impute to obtain numerical imputed data: "get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [abstract] SCALE outperforms other tools in denoising and imputation: "SCALE substantially outperforms the other tools in all aspects of scATAC-seq data analysis, including visualization, clustering, and denoising and imputation."
- [discussion] Comparison against scRNA-seq tools: "In all comparisons, SCALE performs much more favorably than scABC and scRNA-seq tools."
- [intro] Motivation: missingness in scATAC-seq data: [evidence span withheld — anchor status: not_found]
- [readme] Default parameters for reproducibility: "[--seed]: random seed for parameter initialization, default is 18"
- [abstract] SCALE combines VAE framework with Gaussian Mixture Model: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
