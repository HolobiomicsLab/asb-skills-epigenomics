---
name: scatac-seq-latent-feature-clustering
description: Use when the input is a raw scATAC-seq count matrix (h5ad, txt/tsv/csv,
  or mtx folder with count/peak/barcode files) and the goal is cell-type clustering,
  visualization, or denoising/imputation with benchmarked accuracy against scABC,
  scVI, cisTopic, and SC3.
when_to_use_negative:
- The task is to identify and remove batch effects or other technical artifacts from
  input data — the authors state SCALE is 'not specifically designed to identify and
  remove these artifacts from the input data'; explicit batch-effect modeling is deferred
  to future work (or use SCALEX for integration).
- The analysis needs single-peak motif resolution via grouped motif activity alone
  — chromVAR evaluates groups of peaks sharing motifs without individual peak resolution,
  and this skill is for whole-dataset latent-feature clustering instead.
- The input is scRNA-seq expression data — scRNA-seq tools like scVI apply a single
  Gaussian prior in that setting and are not designed for the close-to-binary, sparse
  scATAC-seq count matrix this skill expects.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3474
tools:
- name: SCALE
  role: 'Primary method: VAE with Gaussian Mixture Model prior learns latent features
    from the scATAC-seq count matrix and assigns Leiden clusters, UMAP embeddings,
    and binary imputations'
  repo: https://github.com/jsxlei/SCALE
- name: Homer
  role: 'Post-clustering motif discovery: run findMotifsGenome.pl on cluster-specific
    peak sets (e.g. top 1000 peaks by specificity score per cluster) to find enriched
    transcription factor motifs'
- name: chromVAR
  role: 'Alternative downstream motif analysis: evaluates groups of peaks sharing
    the same motifs or functional annotations, combined with SCALE''s cluster-specific
    peaks'
- name: scanpy
  role: Provides highly_variable_genes for filtering peaks and is used to read adata.obsm['binary']
    imputed data
- name: scVI
  role: Benchmark comparison tool (VAE for scRNA-seq) that SCALE outperforms in clustering
    accuracy metrics such as ARI, NMI, and micro-F1
- name: scABC
  role: Benchmark comparison tool (weighted K-medoid clustering) that SCALE outperforms
    in all comparisons
- name: cisTopic
  role: Benchmark comparison tool (latent Dirichlet allocation on scATAC-seq) against
    which SCALE's clustering is evaluated
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:5bd7eabfd7327ff921dd5ad3ad914cdd8894f4dc1ae71920092916cbcabe9343
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - Homer
  - SCALE
  - chromVAR
  - cisTopic
  - scABC
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# scatac-seq-latent-feature-clustering

## Summary

Cluster sparse single-cell ATAC-seq data by learning latent features with SCALE, a variational autoencoder (VAE) with a probabilistic Gaussian Mixture Model prior, then annotate clusters via Leiden and UMAP embeddings. SCALE is applied because the close-to-binary, high-dimensional scATAC-seq data suffer from 'missingness', making direct scRNA-seq methods unsuitable.

## When to use

Use when the input is a raw scATAC-seq count matrix (h5ad, txt/tsv/csv, or mtx folder with count/peak/barcode files) and the goal is cell-type clustering, visualization, or denoising/imputation with benchmarked accuracy against scABC, scVI, cisTopic, and SC3.

## When NOT to use

- The task is to identify and remove batch effects or other technical artifacts from input data — the authors state SCALE is 'not specifically designed to identify and remove these artifacts from the input data'; explicit batch-effect modeling is deferred to future work (or use SCALEX for integration).
- The analysis needs single-peak motif resolution via grouped motif activity alone — chromVAR evaluates groups of peaks sharing motifs without individual peak resolution, and this skill is for whole-dataset latent-feature clustering instead.
- The input is scRNA-seq expression data — scRNA-seq tools like scVI apply a single Gaussian prior in that setting and are not designed for the close-to-binary, sparse scATAC-seq count matrix this skill expects.

## Inputs

- scATAC-seq count matrix as .h5ad (e.g. Forebrain.h5ad, Breast_Tumor.h5ad from zhanglab.net/SCALE_SOURCE_DATA/)
- count matrix file in txt/tsv (tab-separated) or csv (comma-separated) format, rows=peaks, columns=barcodes
- mtx folder containing three files: count matrix, peak file (chr_start_end), and barcode file
- h5mu file (e.g. filename.h5mu/atac) for multiomics input
- Expected number of clusters (-k, e.g. 8 for the Forebrain dataset)

## Outputs

- model.pt: saved model for reproduction with --pretrain
- adata.h5ad: annotated AnnData with Leiden cluster assignments, latent feature matrix, and UMAP results
- umap.pdf: 2D UMAP embedding of cells
- binary imputed data at adata.obsm['binary'] when run with --binary
- Downstream: cluster-specific peak sets fed to Homer for known-motif enrichment (e.g. homer_known_motif_results.tsv)

## How to apply

Install SCALE from PyPI ('pip install scale') or from GitHub and run 'SCALE.py -d [input]' on the count matrix; SCALE's VAE with a Gaussian Mixture Model prior learns latent features that characterize the sparse scATAC-seq distribution. Filter low-quality cells with --min_peaks (default 100) and low-quality peaks with --min_cells (default 3), and optionally select highly variable features with --n_feature (default 100,000) or scanpy's highly_variable_genes. Supply the expected number of clusters via the -k option (e.g. 'SCALE.py -d Forebrain.h5ad -k 8' for the Forebrain dataset with 2088 cells); defaults include an initial learning rate of 0.002, max 30000 iterations, and random seed 18. Outputs land in the output folder: model.pt for pretraining reproduction via --pretrain, adata.h5ad holding Leiden cluster assignments, latent feature matrix, and binary imputed data (adata.obsm['binary'] with --binary), and umap.pdf of 2D UMAP embeddings. For downstream interpretation, combine the discovered cluster-specific peaks with Homer (findMotifsGenome.pl) or chromVAR to identify regulatory motifs enriched per cluster.

## Related tools

- **SCALE** (Primary method: VAE with Gaussian Mixture Model prior learns latent features from the scATAC-seq count matrix and assigns Leiden clusters, UMAP embeddings, and binary imputations) — https://github.com/jsxlei/SCALE
- **Homer** (Post-clustering motif discovery: run findMotifsGenome.pl on cluster-specific peak sets (e.g. top 1000 peaks by specificity score per cluster) to find enriched transcription factor motifs)
- **chromVAR** (Alternative downstream motif analysis: evaluates groups of peaks sharing the same motifs or functional annotations, combined with SCALE's cluster-specific peaks)
- **scanpy** (Provides highly_variable_genes for filtering peaks and is used to read adata.obsm['binary'] imputed data)
- **scVI** (Benchmark comparison tool (VAE for scRNA-seq) that SCALE outperforms in clustering accuracy metrics such as ARI, NMI, and micro-F1)
- **scABC** (Benchmark comparison tool (weighted K-medoid clustering) that SCALE outperforms in all comparisons)
- **cisTopic** (Benchmark comparison tool (latent Dirichlet allocation on scATAC-seq) against which SCALE's clustering is evaluated)

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8
```

## Evaluation signals

- Output folder contains all three artifacts: model.pt, adata.h5ad with Leiden cluster assignments and latent features, and umap.pdf
- Clustering accuracy computed with scikit-learn ARI, NMI, and micro-F1 against reference labels should reproduce SCALE's ranking from the paper's Fig. 2b benchmark (SCALE outperforms scABC, scVI, cisTopic, SC3; SC3 collapses most cells into one group and scVI misclassifies three subgroups on Forebrain)
- Reproducibility: rerun with the saved model.pt via --pretrain; defaults (lr=0.002, max 30000 iterations, seed 18) should give consistent cluster assignments
- Number of clusters in adata.h5ad matches the -k value supplied on the command line (e.g. 8 for Forebrain)
- Downstream validation: Homer known-motif enrichment on cluster-specific peaks recovers expected cell-type motifs (e.g. immune motifs Pu.1-Irf, Irf8, Runx1, SpiB in CD45+ cells; epithelial motifs Klf14, Mitf, Ets1 in Epcam+ cells of Breast Tumor)

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work
- scATAC-seq analysis suffers from the curse of 'missingness' and high dimensionality — each open chromatin site has only one or two capture opportunities, yielding only a few thousand distinct reads per cell — so results depend on the VAE+GMM prior to handle sparsity
- Direct applications of scRNA-seq analysis methods to scATAC-seq data may be unsuitable due to the close-to-binary nature and increased sparsity of the data

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [methods] SCALE.py command: [evidence span withheld — anchor status: not_found]
- [methods] filter options: "[--min_peaks]: filter low quality cells by valid peaks number, default 100"
- [methods] saved model: "**model.pt**:  saved model to reproduce results cooperated with option --pretrain"
- [discussion] motif discovery with Homer: "we can improve the discovery of cluster-specific peaks, and regulatory motifs as well, when combined with tools like Homer38 or chromVAR13."
- [discussion] batch effect caveat: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
- [readme] README output files: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [readme] README imputation: "Get binary imputed data in adata.h5ad file using scanpy **adata.obsm['binary']** with option **--binary**"
- [readme] README installation: "pip install git+https://github.com/jsxlei/SCALE.git"
