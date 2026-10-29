---
name: runtime-memory-benchmarking
description: Use when reproducing or auditing SCALE's scalability claims on a large
  scATAC-seq input (e.g. the ~80k-cell mouse_atlas.h5ad), where reported runtime (~1.5
  h), memory (~2 GB), and clustering F1 (0.419) must be measured and compared.
when_to_use_negative:
- Input is not a large-scale scATAC-seq count matrix but already a feature/embedding
  table — SCALE's VAE + Gaussian Mixture Model operates on peak-by-cell accessibility
  data
- 'The goal is batch-effect removal: the article states SCALE ''is not specifically
  designed to identify and remove these artifacts from the input data'''
- Small-scale runs where runtime/memory claims are not under test — use the standard
  Forebrain tutorial (k=8, 2088 cells) instead of the atlas benchmark
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCALE
  role: VAE + probabilistic Gaussian Mixture Model tool run on mouse_atlas.h5ad to
    produce Leiden clusters whose runtime, memory, and F1 are benchmarked
  repo: https://github.com/jsxlei/SCALE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:6ee7fcae3fb9a95e5316b382db7d6930c44ae767b573e998499b952049c323c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SCALE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# runtime-memory-benchmarking

## Summary

Benchmark the wall-clock runtime and peak memory of SCALE when clustering a large scATAC-seq dataset (the ~80,000-cell mouse sci-ATAC-seq atlas), and verify cluster-label agreement (F1 0.419 against 30 reference cell types) so computational efficiency claims can be reproduced.

## When to use

Use when reproducing or auditing SCALE's scalability claims on a large scATAC-seq input (e.g. the ~80k-cell mouse_atlas.h5ad), where reported runtime (~1.5 h), memory (~2 GB), and clustering F1 (0.419) must be measured and compared.

## When NOT to use

- Input is not a large-scale scATAC-seq count matrix but already a feature/embedding table — SCALE's VAE + Gaussian Mixture Model operates on peak-by-cell accessibility data
- The goal is batch-effect removal: the article states SCALE 'is not specifically designed to identify and remove these artifacts from the input data'
- Small-scale runs where runtime/memory claims are not under test — use the standard Forebrain tutorial (k=8, 2088 cells) instead of the atlas benchmark

## Inputs

- mouse_atlas.h5ad scATAC-seq dataset (~80,000 sci-ATAC-seq cells) from http://zhanglab.net/SCALE_SOURCE_DATA/mouse_atlas.h5ad
- Reference cell-type annotations (30 inferred reference clusters) for cluster-label matching
- CUDA-capable GPU (optional but recommended)

## Outputs

- adata.h5ad with Leiden cluster assignment, latent feature matrix and UMAP results
- model.pt saved model for reproduction with --pretrain
- umap.pdf 2D UMAP embedding visualization
- Measured wall-clock runtime and peak memory figures, and overall cluster-label F1 score

## How to apply

Download mouse_atlas.h5ad from the SCALE source-data repository and run SCALE.py with the defaults the README specifies: initial learning rate 0.002 (--lr), 30000 max iterations (--max_iter), random seed 18 (--seed), and cell filtering at --min_peaks 100, preferring CUDA if available since the PyTorch implementation is GPU-accelerated. Extract the Leiden cluster assignments and latent feature matrix from the output adata.h5ad, match the SCALE clusters to the atlas's 30 inferred reference cell types, and compute the overall cluster-label agreement F1 (target 0.419, with most major reference clusters having a corresponding SCALE cluster). Independently record wall-clock runtime and peak memory of the run and compare against the reported ~1.5 h and ~2 GB. Preserve the full output triplet (model.pt, adata.h5ad, umap.pdf) so the run can be re-seeded via --pretrain and the 2D UMAP embedding inspected.

## Related tools

- **SCALE** (VAE + probabilistic Gaussian Mixture Model tool run on mouse_atlas.h5ad to produce Leiden clusters whose runtime, memory, and F1 are benchmarked) — https://github.com/jsxlei/SCALE

## Examples

```
SCALE.py -d mouse_atlas.h5ad --lr 0.002 --max_iter 30000 --seed 18 --min_peaks 100 -o output/
```

## Evaluation signals

- Overall cluster-label agreement F1 against the 30 reference cell types is approximately 0.419, with most major reference clusters having a corresponding SCALE cluster
- Wall-clock runtime is on the order of ~1.5 h and peak memory ~2 GB for the ~80,000-cell atlas
- Output folder contains all three artifacts: model.pt, adata.h5ad (with Leiden assignments and latent features), and umap.pdf
- Re-running with --pretrain on the saved model.pt and --seed 18 yields comparable cluster assignments
- Runtime and memory plots can be compared against the README's runtime.png and memory.png references

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of technical variations is deferred to future work
- Results depend on random seed (default 18) and cluster number: without -k, Leiden clustering may not recover exactly 30 clusters
- Cell quality filtering (--min_peaks, default 100) and peak selection (--n_feature, default 100,000; --min_cells default 3) change the effective input size and hence runtime/memory

## Evidence

- [readme] Running SCALE on CUDA is recommended if available.: "Running SCALE on CUDA is recommended if available."
- [readme] default max_iter 30000: "[--max_iter] or [-i]: max iteration number, default is 30000"
- [readme] adata.h5ad output contents: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [readme] mouse_atlas dataset availability: "* [mouse_atlas](http://zhanglab.net/SCALE_SOURCE_DATA/mouse_atlas.h5ad)"
- [methods] model.pt pretrain reproduction: "**model.pt**:  saved model to reproduce results cooperated with option --pretrain"
- [discussion] SCALE scope as analysis tool: "SCALE is thus a powerful tool for scATAC-seq data analysis, including data visualization, clustering, and denoising and imputation."
