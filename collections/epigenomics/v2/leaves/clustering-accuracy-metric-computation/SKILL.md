---
name: clustering-accuracy-metric-computation
description: Use when comparing cluster assignments from a scATAC-seq method (e.g.,
  SCALE latent-feature clustering with K-means) against reference cell-type labels,
  and a quantitative ranking of competing tools (ARI/NMI/F1) is required.
when_to_use_negative:
- The data need batch-effect identification or removal — the authors state 'SCALE
  is not specifically designed to identify and remove these artifacts from the input
  data', so batch-corrected benchmarking is out of scope.
- The input is scRNA-seq data — scVI was adapted for scRNA-seq, whereas this benchmark
  targets close-to-binary, sparse scATAC-seq profiles.
- Only one clustering method was run and there is no reference label set — no ground-truth
  comparison for ARI/NMI/F1 is possible.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: SCALE
  role: 'primary method: VAE+GMM latent feature extraction followed by Leiden/K-means
    clustering on Forebrain.h5ad with -k 8; its output labels are the benchmarked
    assignments'
  repo: https://github.com/jsxlei/SCALE
- name: scABC
  role: 'baseline clustering method: weighs cells by sequencing depth and applies
    weighted K-medoid clustering'
- name: scVI
  role: 'baseline: VAE adapted for scRNA-seq, applied to the scATAC-seq count matrix
    for comparison'
- name: cisTopic
  role: 'baseline: latent Dirichlet allocation applied to model scATAC-seq data'
- name: SC3
  role: baseline clustering method included in the Fig. 2b comparison
- name: scikit-learn
  role: computes ARI, NMI, and micro-F1 between each method's assignments and the
    Forebrain reference labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12630-7
    title: SCALE method for single-cell ATAC-seq analysis via latent feature extraction
  source_content_hash: sha256:a57f19fcae7cea54d9fc1e11ad4d1bec1ff7f40d03077a236301d52a932ba245
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clustering-accuracy-metric-computation

## Summary

Benchmark the accuracy of single-cell cluster assignments against reference cell-type labels by computing ARI, NMI, and micro-F1 with scikit-learn, as done when comparing SCALE against scABC, scVI, cisTopic, and SC3 on the Forebrain scATAC-seq dataset.

## When to use

Use when comparing cluster assignments from a scATAC-seq method (e.g., SCALE latent-feature clustering with K-means) against reference cell-type labels, and a quantitative ranking of competing tools (ARI/NMI/F1) is required.

## When NOT to use

- The data need batch-effect identification or removal — the authors state 'SCALE is not specifically designed to identify and remove these artifacts from the input data', so batch-corrected benchmarking is out of scope.
- The input is scRNA-seq data — scVI was adapted for scRNA-seq, whereas this benchmark targets close-to-binary, sparse scATAC-seq profiles.
- Only one clustering method was run and there is no reference label set — no ground-truth comparison for ARI/NMI/F1 is possible.

## Inputs

- Forebrain.h5ad dense-matrix scATAC-seq dataset (2088 cells, k=8 reference clusters) from http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad
- Cluster assignments (Leiden labels from adata.h5ad for SCALE; cluster vectors from scABC, scVI, cisTopic, SC3)
- Reference cell-type label vector for the Forebrain dataset

## Outputs

- forebrain_benchmark_metrics.csv: table of ARI, NMI, and micro-F1 for SCALE, scABC, scVI, cisTopic, and SC3
- SCALE artifacts: model.pt (saved model) and adata.h5ad (Leiden cluster assignment, latent feature matrix, UMAP results)

## How to apply

Run each clustering tool on the same Forebrain count matrix (2088 cells, k=8): SCALE via its VAE+GMM latent features with SCALE.py -k 8 using defaults (lr 0.002, max_iter 30000, seed 18), then scABC (weighted K-medoid), scVI (VAE), cisTopic (latent Dirichlet allocation), and SC3 on identical inputs. Extract each method's cluster assignments and the Forebrain reference labels, then compute adjusted Rand index, normalized mutual information, and micro-F1 with scikit-learn's metrics module for every method. Assemble all metrics into a single table (forebrain_benchmark_metrics.csv) so methods are ranked on identical label sets. Validate by confirming SCALE's scores reproduce the paper's clustering comparison (Fig. 2b), where SCALE achieved the most accurate clustering and SC3 collapsed most cells into one group while scVI misclassified three subgroups.

## Related tools

- **SCALE** (primary method: VAE+GMM latent feature extraction followed by Leiden/K-means clustering on Forebrain.h5ad with -k 8; its output labels are the benchmarked assignments) — https://github.com/jsxlei/SCALE
- **scABC** (baseline clustering method: weighs cells by sequencing depth and applies weighted K-medoid clustering)
- **scVI** (baseline: VAE adapted for scRNA-seq, applied to the scATAC-seq count matrix for comparison)
- **cisTopic** (baseline: latent Dirichlet allocation applied to model scATAC-seq data)
- **SC3** (baseline clustering method included in the Fig. 2b comparison)
- **scikit-learn** (computes ARI, NMI, and micro-F1 between each method's assignments and the Forebrain reference labels)

## Examples

```
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score, f1_score; ari = adjusted_rand_score(forebrain_ref, scale_pred); nmi = normalized_mutual_info_score(forebrain_ref, scale_pred); f1 = f1_score(forebrain_ref, scale_pred, average='micro')
```

## Evaluation signals

- The metrics table contains ARI, NMI, and micro-F1 rows for all five methods (SCALE, scABC, scVI, cisTopic, SC3).
- SCALE's ARI/NMI scores are the highest of the five methods, reproducing the ranking reported in the paper's clustering comparison (Fig. 2b).
- Sanity check of baselines: SC3 collapses most Forebrain cells into one group and scVI misclassifies three subgroups.
- SCALE run artifacts exist: model.pt (for --pretrain reproduction) and adata.h5ad containing Leiden cluster assignment, latent features, and UMAP.
- All methods were clustered on the same Forebrain count matrix with k=8 so metrics are computed on identical label sets.

## Limitations

- SCALE is not specifically designed to identify and remove batch effects or other artifacts from the input data; explicit modeling of batch effects is deferred to future work.
- Direct applications of scRNA-seq analysis methods (e.g., scVI) to scATAC-seq data may not be suitable due to the close-to-binary nature and increased sparsity of the data.
- Each method has caveats: chromVAR lacks individual peak resolution and scABC heavily depends on landmark samples with high sequencing depth.

## Evidence

- [abstract] SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data.: "SCALE combines a deep generative framework and a probabilistic Gaussian Mixture Model to learn latent features that accurately characterize scATAC-seq data."
- [abstract] SCALE substantially outperforms the other tools in all aspects of scATAC-seq data analysis, including visualization, clustering, and denoising and imputation.: "SCALE substantially outperforms the other tools in all aspects of scATAC-seq data analysis, including visualization, clustering, and denoising and imputation."
- [readme] README: SCALE options for reproducible clustering: "[--max_iter] or [-i]: max iteration number, default is 30000"
- [readme] README: random seed: "[--seed]: random seed for parameter initialization, default is 18"
- [readme] README: cluster number option: "* [-k]: if cluster number is known"
- [readme] README: output artifacts: "**adata.h5ad**: saved data including Leiden cluster assignment, latent feature matrix and UMAP results."
- [intro] methods: scABC baseline description: "Another method, scABC, weighs cells by sequencing depth and applies weighted K-medoid clustering to reduce the impact of missing values14."
- [methods] methods: dataset download: "[Forebrain](http://zhanglab.net/SCALE_SOURCE_DATA/Forebrain.h5ad)"
- [discussion] discussion: SCALE limitation: "Nevertheless, SCALE is not specifically designed to identify and remove these artifacts from the input data."
