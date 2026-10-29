---
name: meta-cell-correlation-benchmarking
description: Use when you must quantify whether an imputed or denoised single-cell
  matrix (e.g. SCALE's obsm['imputed'] scATAC-seq peaks) recovers cluster-specific
  signal, by Pearson-correlating each cell against per-cluster meta-cells.
when_to_use_negative:
- 'Input data carry strong batch effects or other technical artifacts: SCALE is not
  specifically designed to identify and remove these from the input data, so meta-cell
  correlations can be confounded.'
- 'No reliable cluster assignments exist (k unknown or clusters of mixed cell types):
  averaging raw accessibility over the wrong groups destroys the cell-type-specific
  signal this benchmark measures.'
- The goal is clustering, UMAP visualization, or motif analysis rather than imputation-quality
  assessment — use SCALE's Leiden/UMAP outputs or the Homer/chromVAR motif workflow
  instead.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SCALE
  role: 'Produces the numerical imputation under test: run with --impute it writes
    the VAE-GMM imputed matrix to adata.obsm[''imputed''] in the output adata.h5ad'
  repo: https://github.com/jsxlei/SCALE
- name: scanpy
  role: Reads the output adata.h5ad, exposes adata.obsm['imputed'], and provides highly_variable_genes
    peak filtering
  repo: https://scanpy.readthedocs.io/en/stable/index.html
- name: scVI
  role: scRNA-seq VAE imputation baseline evaluated with the same cell-by-meta-cell
    correlation comparison
- name: scImpute
  role: scRNA-seq imputation baseline compared in the Forebrain meta-cell correlation
    evaluation
- name: SAVER
  role: scRNA-seq imputation baseline compared in the Forebrain meta-cell correlation
    evaluation
- name: MAGIC
  role: scRNA-seq imputation baseline compared in the Forebrain meta-cell correlation
    evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:9c90eec9953f263dd920adafa476e95bd277ba9d93b90c63cff9c5b8f44fc6ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - MAGIC
  - SAVER
  - SCALE
  - scImpute
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# meta-cell-correlation-benchmarking

## Summary

Benchmark single-cell imputation/denoising quality by computing cell-wise Pearson correlations between every imputed cell and per-cluster meta-cells built from raw accessibility — the Fig. 3a-style evaluation with which SCALE (VAE-GMM latent feature model) beat scImpute, SAVER, MAGIC, and scVI on the Forebrain scATAC-seq dataset (2088 cells, k=8).

## When to use

Use when you must quantify whether an imputed or denoised single-cell matrix (e.g. SCALE's obsm['imputed'] scATAC-seq peaks) recovers cluster-specific signal, by Pearson-correlating each cell against per-cluster meta-cells.

## When NOT to use

- Input data carry strong batch effects or other technical artifacts: SCALE is not specifically designed to identify and remove these from the input data, so meta-cell correlations can be confounded.
- No reliable cluster assignments exist (k unknown or clusters of mixed cell types): averaging raw accessibility over the wrong groups destroys the cell-type-specific signal this benchmark measures.
- The goal is clustering, UMAP visualization, or motif analysis rather than imputation-quality assessment — use SCALE's Leiden/UMAP outputs or the Homer/chromVAR motif workflow instead.

## Inputs

- Dense scATAC-seq peak-by-cell count matrix as .h5ad (e.g. Forebrain.h5ad from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad; 2088 cells, k=8)
- Per-cell cluster assignments (e.g. Leiden clusters from SCALE's adata.h5ad) defining the meta-cell groups
- Imputed/denoised matrices from each method under comparison (SCALE obsm['imputed'], plus scImpute, SAVER, MAGIC, scVI outputs)

## Outputs

- Cell-by-meta-cell Pearson correlation matrix (2088 cells x 8 meta-cells for Forebrain), one per imputation method
- Per-method diagnostic of whether each cell's maximum correlation lands on its assigned cluster's meta-cell (Fig. 3a-style recovery of cell-type-specific structure)
- SCALE adata.h5ad containing obsm['imputed'] numerical imputation (and obsm['binary'] if --binary was used)

## How to apply

Download the preprocessed Forebrain.h5ad (dense peak-by-cell matrix, 2088 cells, k=8 clusters) and run SCALE in imputation mode with its documented defaults — learning rate 0.002, max_iter 30000, seed 18, k=8 — via 'SCALE.py -d [input] --impute'. Read the output adata.h5ad and extract the numerical imputed matrix from adata.obsm['imputed'], verifying it has the same number of cells (2088) as the input. Build one meta-cell per cluster by averaging the RAW (unimputed) accessibility counts over all cells assigned to each of the 8 clusters, so the test measures how well imputation recovers shared cell-type-specific signal rather than the model's own output. Compute Pearson correlations between every imputed cell and every meta-cell (a 2088x8 matrix reproducing the Fig. 3a evaluation), and score a method by how often each cell's top-correlating meta-cell is its own assigned cluster. Repeat with baselines (scImpute, SAVER, MAGIC, scVI) on the same input and compare the recovered per-cluster correlation structure; SCALE's reported result is the highest cell-to-matching-meta-cell correlation for each cell type across scATAC-seq datasets.

## Related tools

- **SCALE** (Produces the numerical imputation under test: run with --impute it writes the VAE-GMM imputed matrix to adata.obsm['imputed'] in the output adata.h5ad) — https://github.com/jsxlei/SCALE
- **scanpy** (Reads the output adata.h5ad, exposes adata.obsm['imputed'], and provides highly_variable_genes peak filtering) — https://scanpy.readthedocs.io/en/stable/index.html
- **scVI** (scRNA-seq VAE imputation baseline evaluated with the same cell-by-meta-cell correlation comparison)
- **scImpute** (scRNA-seq imputation baseline compared in the Forebrain meta-cell correlation evaluation)
- **SAVER** (scRNA-seq imputation baseline compared in the Forebrain meta-cell correlation evaluation)
- **MAGIC** (scRNA-seq imputation baseline compared in the Forebrain meta-cell correlation evaluation)

## Examples

```
SCALE.py -d Forebrain.h5ad --impute --lr 0.002 --max_iter 30000 --seed 18 -k 8
```

## Evaluation signals

- Output adata.h5ad exists and adata.obsm['imputed'] has one row per input cell (2088 for Forebrain) — cell-count parity with the raw matrix.
- Each cell's highest Pearson correlation across the 8 meta-cells lands on its own assigned cluster's meta-cell (diagonal-dominant correlation structure).
- Meta-cells are computed by averaging RAW accessibility within each of the k=8 clusters (not over imputed values); verify exactly 8 meta-cells for Forebrain.
- Reproduction uses the documented defaults lr=0.002, max_iter=30000, seed=18; deviations change the imputation and break comparability with the published ranking.
- Expected qualitative result: SCALE attains the highest cell-to-matching-meta-cell Pearson correlation of the tested methods (scImpute, SAVER, MAGIC, scVI) on Forebrain.

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; batch confounding can distort meta-cell correlations, and explicit batch modeling is deferred to future work.
- The Forebrain benchmark presupposes a dense, preprocessed matrix with a known cluster number (k=8); with unknown or unreliable cluster assignments the meta-cell definition is ambiguous.
- Direct application of the scRNA-seq baselines (scImpute, SAVER, MAGIC, scVI) to scATAC-seq peaks may be unsuitable due to the close-to-binary nature and increased sparsity of the data, so method rankings partly reflect model-data mismatch rather than pure imputation quality.

## Evidence

- [readme] or get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**: "or get numerical imputed data in adata.h5ad file using scanpy **adata.obsm['imputed']** with option **--impute**"
- [readme] Run SCALE on dense matrix **Forebrain** dataset (k=8, 2088 cells): "Run SCALE on dense matrix **Forebrain** dataset (k=8, 2088 cells)"
- [other] Build a per-cluster meta-cell by averaging raw accessibility over cells within each of the 8 clusters, then compute cell-wise Pearson correlations between each imputed cell and each meta-cell: [evidence span withheld — anchor status: not_found]
- [abstract] SCALE substantially outperforms the other tools in all aspects of scATAC-seq data analysis, including visualization, clustering, and denoising and imputation.: "SCALE substantially outperforms the other tools in all aspects of scATAC-seq data analysis, including visualization, clustering, and denoising and imputation."
- [intro] Direct applications of these scRNA-seq analysis methods to scATAC-seq data, however, may not be suitable due to the close-to-binary nature and increased sparsity of the data: "Direct applications of these scRNA-seq analysis methods to scATAC-seq data, however, may not be suitable due to the close-to-binary nature and increased sparsity of the data"
- [discussion] SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation.: "SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation."
- [discussion] Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data.: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
