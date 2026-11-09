---
name: variational-autoencoder-latent-feature-extraction
description: Use when input is a scATAC-seq count matrix (h5ad, txt/tsv/csv, or mtx
  folder of peak/barcode files) that is sparse and close-to-binary, and the goal is
  cell clustering, visualization, or imputation via a learned latent feature space.
when_to_use_negative:
- Input is scRNA-seq data or multi-dataset integration/batch-removal is the primary
  goal — use SCALEX (github.com/jsxlei/SCALEX) instead, since SCALE is not specifically
  designed to identify and remove batch effects or other artifacts.
- The task requires single-peak motif resolution only — chromVAR-style grouped motif
  analysis may be preferred, since chromVAR only analyzes peaks in groups; SCALE's
  imputed output is meant to be combined with tools like Homer or chromVAR for downstream
  motif discovery.
- The data are high-coverage bulk ATAC-seq or already a feature/embedding table rather
  than a per-cell peak count matrix.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALE
  role: Core VAE-GMM tool that learns latent features from scATAC-seq data and performs
    clustering, visualization, denoising and imputation; invoked via SCALE.py
  repo: https://github.com/jsxlei/SCALE
- name: SCALEX
  role: Online integration tool for scRNA-seq and scATAC-seq used when projecting
    heterogeneous datasets into a common cell-embedding space instead of single-dataset
    SCALE runs
  repo: https://github.com/jsxlei/SCALEX
- name: chromVAR
  role: Downstream tool for regulatory motif analysis on SCALE's cluster-specific
    peaks / imputed data
- name: Homer
  role: Combined with SCALE to improve discovery of cluster-specific peaks and regulatory
    motifs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:2ad9384304164c18b87eecfcae39b41bdc868d2e584d9ea7e4af9406509aac33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - Homer
  - SCALE
  - SCALEX
  - chromVAR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# variational-autoencoder-latent-feature-extraction

## Summary

Apply SCALE, a VAE-based deep generative model with a probabilistic Gaussian Mixture Model prior, to extract latent features from sparse, near-binary single-cell ATAC-seq count matrices and use them for clustering, UMAP visualization, and denoising/imputation of open-chromatin profiles.

## When to use

Use when input is a scATAC-seq count matrix (h5ad, txt/tsv/csv, or mtx folder of peak/barcode files) that is sparse and close-to-binary, and the goal is cell clustering, visualization, or imputation via a learned latent feature space.

## When NOT to use

- Input is scRNA-seq data or multi-dataset integration/batch-removal is the primary goal — use SCALEX (github.com/jsxlei/SCALEX) instead, since SCALE is not specifically designed to identify and remove batch effects or other artifacts.
- The task requires single-peak motif resolution only — chromVAR-style grouped motif analysis may be preferred, since chromVAR only analyzes peaks in groups; SCALE's imputed output is meant to be combined with tools like Homer or chromVAR for downstream motif discovery.
- The data are high-coverage bulk ATAC-seq or already a feature/embedding table rather than a per-cell peak count matrix.

## Inputs

- scATAC-seq count matrix in h5ad format (e.g. mouse_atlas.h5ad from http://zhanglab.net/SCALE_SOURCE_DATA/mouse_atlas.h5ad)
- count matrix file with peaks as rows and barcodes as columns in txt/tsv (tab-separated) or csv (comma-separated) format
- mtx folder containing count file (mtx format), peak file (chr_start_end), and barcode file
- h5mu multiomics file (e.g. filename.h5mu/atac)

## Outputs

- model.pt — saved trained model for reproduction with --pretrain
- adata.h5ad — data including Leiden cluster assignment, latent feature matrix, and UMAP results
- umap.pdf — visualization of 2D UMAP embeddings of each cell
- adata.obsm['binary'] (with --binary) or adata.obsm['imputed'] (with --impute) for denoised/imputed peak data

## How to apply

Install SCALE from PyPI (pip install scale) or GitHub, then run SCALE.py on the input (e.g. a mouse_atlas.h5ad or dense matrix). SCALE first filters low-quality cells by valid peak count (--min_peaks, default 100) and selects highly variable peaks (--n_feature, default 100,000; --min_cells default 3 for the older peak filter), then trains the VAE-GMM model with initial learning rate 0.002, up to 30000 iterations, and random seed 18, preferably on CUDA. The decision point on cluster number is set with -k when the number of clusters is known; otherwise Leiden clustering is applied in the latent space. Outputs land in the output folder: model.pt for reproducible pretraining via --pretrain, adata.h5ad carrying Leiden cluster assignments, the latent feature matrix and UMAP coordinates, and umap.pdf for 2D UMAP embeddings. Optionally obtain binary imputed data via --binary (adata.obsm['binary']) or numerical imputation via --impute. The latent features are expected to be interpretable and linked to cell populations, which can be checked by mapping clusters onto known reference cell types.

## Related tools

- **SCALE** (Core VAE-GMM tool that learns latent features from scATAC-seq data and performs clustering, visualization, denoising and imputation; invoked via SCALE.py) — https://github.com/jsxlei/SCALE
- **SCALEX** (Online integration tool for scRNA-seq and scATAC-seq used when projecting heterogeneous datasets into a common cell-embedding space instead of single-dataset SCALE runs) — https://github.com/jsxlei/SCALEX
- **chromVAR** (Downstream tool for regulatory motif analysis on SCALE's cluster-specific peaks / imputed data)
- **Homer** (Combined with SCALE to improve discovery of cluster-specific peaks and regulatory motifs)

## Examples

```
SCALE.py -d mouse_atlas.h5ad --lr 0.002 --max_iter 30000 --seed 18 --min_peaks 100
```

## Evaluation signals

- Output folder contains all three artifacts: model.pt, adata.h5ad with Leiden cluster assignments and latent feature matrix, and umap.pdf.
- On the ~80,000-cell mouse atlas, clusters matched to the 30 reference cell types should yield an overall F1 score near 0.419, with most major reference clusters having a corresponding SCALE cluster.
- Runtime and peak memory stay in the reported range (~1.5 h wall-clock, ~2 GB memory) for the atlas-scale run on CUDA.
- Filtering invariants: cells below --min_peaks 100 valid peaks are dropped, and peak selection respects --n_feature (default 100,000 highly variable features) or --min_cells 3.
- Reproducibility: rerunning with --pretrain on model.pt and --seed 18 with --lr 0.002 and --max_iter 30000 should recover consistent clustering.

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects was deferred to future work.
- Direct application of scRNA-seq methods to scATAC-seq data is problematic due to the close-to-binary nature and increased sparsity — models must be validated on datasets from different platforms, protocols, and data qualities.
- Performance depends on quality filtering defaults; changing --min_cells/--n_feature behavior across versions (min_cells default changed from 0.01 to 3, now replaced by --n_feature) can alter results.

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [readme] min_peaks filter: "[--min_peaks]: filter low quality cells by valid peaks number, default 100"
- [readme] max_iter option: "[--max_iter] or [-i]: max iteration number, default is 30000"
- [methods] install command: "pip install git+https://github.com/jsxlei/SCALE.git"
- [readme] n_feature filter: "[--n_feature]: filter peaks by selecting highly variable features, default 100,000; use [--n_feature] -1 to disable."
- [discussion] batch effects caveat: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
