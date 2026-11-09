---
name: cross-method-clustering-comparison
description: Use when a scATAC-seq (or scRNA-seq) dataset with reference cell-type
  labels is available and you must quantify how accurately competing clustering tools
  recover those labels, including under graded data corruption.
when_to_use_negative:
- The dataset has no reference cell-type labels, so ARI/NMI/micro-F1 against a ground
  truth cannot be computed
- You only need to run and tune a single clustering method rather than compare several
  tools head-to-head on identical input
- SCALE is being used for batch-effect removal — the article notes it 'is not specifically
  designed to identify and remove these artifacts from the input data', so it is the
  wrong tool for that goal rather than for clustering comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SCALE
  role: 'Primary method under test: VAE+GMM latent feature extraction followed by
    clustering (-k 8, lr 0.002, max_iter 30000, seed 18)'
  repo: https://github.com/jsxlei/SCALE
- name: scABC
  role: Baseline clustering tool using sequencing-depth-weighted K-medoid clustering
    on the same Forebrain count matrix
- name: scVI
  role: 'Baseline: VAE-based latent representation originally developed for scRNA-seq
    data, applied to the same matrix'
- name: cisTopic
  role: 'Baseline: latent Dirichlet allocation applied to model scATAC-seq data and
    derive clusters'
- name: SC3
  role: Baseline clustering method applied to the same Forebrain count matrix
- name: scikit-learn
  role: Computes ARI, NMI, and micro-F1 between each method's cluster assignments
    and the reference labels
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:1939f311d7ebe2a89f16a7717755dc3507d4ad6a20ed3028957505f59649a3c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jsxlei/SCALE
  tools:
  - SC3
  - SCALE
  - cisTopic
  - scABC
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-method-clustering-comparison

## Summary

Benchmark a candidate single-cell clustering method (e.g. SCALE's VAE+GMM latent features clustered with K-means/Leiden) against baselines such as scABC, scVI, cisTopic, and SC3 by computing ARI, NMI, and micro-F1 against reference cell-type labels on the same input count matrix.

## When to use

Use when a scATAC-seq (or scRNA-seq) dataset with reference cell-type labels is available and you must quantify how accurately competing clustering tools recover those labels, including under graded data corruption.

## When NOT to use

- The dataset has no reference cell-type labels, so ARI/NMI/micro-F1 against a ground truth cannot be computed
- You only need to run and tune a single clustering method rather than compare several tools head-to-head on identical input
- SCALE is being used for batch-effect removal — the article notes it 'is not specifically designed to identify and remove these artifacts from the input data', so it is the wrong tool for that goal rather than for clustering comparison

## Inputs

- Dense count matrix in h5ad format (e.g. Forebrain.h5ad, 2088 cells, k=8 reference clusters, from zhanglab.net/SCALE_SOURCE_DATA/)
- Reference cell-type labels for each cell
- Optionally corrupted count matrices with graded random peak-dropping (10%–90%)
- Installed clustering tools: SCALE, scABC, scVI, cisTopic, SC3; scikit-learn for metric computation

## Outputs

- forebrain_benchmark_metrics.csv with ARI, NMI, and micro-F1 rows for all methods (SCALE, scABC, scVI, cisTopic, SC3)
- ari_robustness.csv tabulating ARI as a function of corruption level for each method
- Per-run artifacts: SCALE model.pt, adata.h5ad with Leiden cluster assignments and latent features, umap.pdf

## How to apply

Run each method on the identical input matrix — e.g. for SCALE, load Forebrain.h5ad (2088 cells, k=8) and run SCALE.py with -k 8, initial learning rate 0.002, max_iter 30000, random seed 18, collecting the Leiden cluster assignments from the saved adata.h5ad. Run the baselines with their canonical algorithms: scABC's sequencing-depth-weighted K-medoid clustering, scVI's VAE, cisTopic's latent Dirichlet allocation, and SC3. Compute Adjusted Rand Index (ARI), Normalized Mutual Information (NMI), and micro-F1 between each tool's assignments and the reference labels using scikit-learn, and assemble all metrics into one table (e.g. forebrain_benchmark_metrics.csv). For robustness analysis, repeat the protocol on randomly corrupted matrices (10%–90% peak dropping, including 30%) and tabulate ARI versus corruption level (ari_robustness.csv). Validate by checking that the ranking reproduces the paper's expectation that SCALE outperforms the other tools, e.g. Forebrain ARI degrading from 0.668 to 0.448 at 30% corruption while scABC falls 0.315→0.222 and scVI 0.448→0.388.

## Related tools

- **SCALE** (Primary method under test: VAE+GMM latent feature extraction followed by clustering (-k 8, lr 0.002, max_iter 30000, seed 18)) — https://github.com/jsxlei/SCALE
- **scABC** (Baseline clustering tool using sequencing-depth-weighted K-medoid clustering on the same Forebrain count matrix)
- **scVI** (Baseline: VAE-based latent representation originally developed for scRNA-seq data, applied to the same matrix)
- **cisTopic** (Baseline: latent Dirichlet allocation applied to model scATAC-seq data and derive clusters)
- **SC3** (Baseline clustering method applied to the same Forebrain count matrix)
- **scikit-learn** (Computes ARI, NMI, and micro-F1 between each method's cluster assignments and the reference labels)

## Examples

```
SCALE.py -d Forebrain.h5ad -k 8 --lr 0.002 --max_iter 30000 --seed 18
```

## Evaluation signals

- The metrics table contains ARI, NMI, and micro-F1 for all five methods (SCALE, scABC, scVI, cisTopic, SC3)
- SCALE reproduces the reported ranking: highest ARI at every corruption level, e.g. 0.668 on raw Forebrain data dropping to 0.448 at 30% corruption
- Baseline trajectories match reported values: scABC 0.315→0.222 and scVI 0.448→0.388 at 30% corruption
- Each SCALE run produces the expected artifacts (model.pt, adata.h5ad containing Leiden cluster assignments, umap.pdf) using the same seed (18) and hyperparameters across runs
- Known failure signatures of baselines are observed, e.g. SC3 collapsing most Forebrain cells into one group and scVI misclassifying three subgroups

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data
- Explicit modeling of batch effects and technical variations is deferred to future work, not implemented
- SC3 can collapse most cells into one group on the Forebrain dataset, making its metrics degenerate; scVI may misclassify subgroups since it was adapted from scRNA-seq analysis

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [discussion] In all comparisons, SCALE performs much more favorably than scABC and scRNA-seq tools.: "In all comparisons, SCALE performs much more favorably than scABC and scRNA-seq tools."
- [readme] adata.h5ad with Leiden cluster assignments: "**adata.h5ad**:  saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [readme] SCALE run options: "[--lr]: modify the initial learning rate, default is 0.002:"
- [intro] scABC baseline description: "Another method, scABC, weighs cells by sequencing depth and applies weighted K-medoid clustering to reduce the impact of missing values14."
