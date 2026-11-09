---
name: consensus-clustering-analysis
description: Use when benchmarking a novel clustering method against consensus clustering
  on the same gene expression matrix, or when you need to quantify cluster-solution
  agreement (e.g., Jaccard Pair Similarity per K) between two cluster assignments
  on shared samples.
when_to_use_negative:
- When the goal is subtype discovery without a predefined K — consensus/bootstrap
  clustering requires a predefined parameter, K, which is a stated limitation for
  subtype discovery
- When the reference method comparison is unnecessary and you only need cluster assignments
  — consensus clustering is a benchmarking comparator here, not the primary method
  being evaluated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the consensus clustering runs, pairwise co-clustering matrices,
    and Jaccard Pair Similarity scoring in the benchmarking pipeline
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQ
  role: Provides the reference cluster solution (diameter 0.63, 6 clusters) against
    which consensus clustering solutions are compared via Pair Similarity scores
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:3b511ce754928b10a365ac91d0ae70ee2876a74bbba8951c0a83c3fcb0b93aa2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/StaafLab/SRIQ
  repo_url: https://github.com/StaafLab/SRIQ
  tools:
  - Python
  - SRIQ
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# consensus-clustering-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs consensus clustering on the log2-transformed, median-centered TCGA-LUAD expression matrix (434 samples, >25000 transcripts) using hierarchical clustering with ward's linkage on Pearson correlation distances, 2000 iterations, and 70% resampling of items and features, then scores similarity to a reference (SRIQ) solution via a Jaccard-based Pair Similarity score. This is the benchmarking standard against which SRIQ's K3 and K6 solutions were evaluated.

## When to use

Use when benchmarking a novel clustering method against consensus clustering on the same gene expression matrix, or when you need to quantify cluster-solution agreement (e.g., Jaccard Pair Similarity per K) between two cluster assignments on shared samples.

## When NOT to use

- When the goal is subtype discovery without a predefined K — consensus/bootstrap clustering requires a predefined parameter, K, which is a stated limitation for subtype discovery
- When the reference method comparison is unnecessary and you only need cluster assignments — consensus clustering is a benchmarking comparator here, not the primary method being evaluated

## Inputs

- Preprocessed, log2-transformed, median-centered TCGA-LUAD gene expression matrix (434 samples, >25000 transcripts, tab-separated gene-by-sample format)
- Reference cluster assignments (e.g., SRIQ solution at diameter 0.63, 6 clusters, core/spiral sample labels)

## Outputs

- Consensus cluster solutions for K=3 to K=7
- Binary pairwise co-clustering similarity matrix per solution
- Jaccard-based Pair Similarity scores per K, written to pair_similarity_scores.csv

## How to apply

Load the preprocessed, log2-transformed, median-centered expression matrix (434 TCGA-LUAD samples, >25000 retained transcripts) with pandas. Run consensus clustering with hierarchical clustering using Pearson correlation distance and ward's linkage, 2000 iterations, and 70% resampling of both items (samples) and features (genes), repeating for each K from 3 to 7. Vary the feature set used by consensus clustering (e.g., exceeding 6000 genes versus smaller sets) since feature-set size materially affects agreement. Convert each solution to a binary pairwise co-clustering matrix (1 if two samples share a cluster), compute the Jaccard index against the reference solution's pairwise matrix (SRIQ diameter 0.63, 6 clusters, core/spiral assignments) to get a Pair Similarity score per K, and write scores for K=3-7 to pair_similarity_scores.csv.

## Related tools

- **Python** (Implements the consensus clustering runs, pairwise co-clustering matrices, and Jaccard Pair Similarity scoring in the benchmarking pipeline) — https://github.com/StaafLab/SRIQ
- **SRIQ** (Provides the reference cluster solution (diameter 0.63, 6 clusters) against which consensus clustering solutions are compared via Pair Similarity scores) — https://github.com/StaafLab/SRIQ

## Examples

```
from sklearn.cluster import AgglomerativeClustering; import pandas as pd; X = pd.read_csv('expression_mc_log2.txt', sep='\t', index_col=0).T; labels = AgglomerativeClustering(n_clusters=3, linkage='ward', metric='euclidean').fit_predict(X)
```

## Evaluation signals

- K3 solutions show relatively high Pair Similarity (>60% for clusters) when the consensus clustering feature set exceeds 6000 genes
- K6 solutions yield <60% similarity for any cluster irrespective of feature-set size, reproducing the paper's reported result
- pair_similarity_scores.csv contains one Pair Similarity score for each K from 3 to 7
- Consensus clustering settings match the benchmark: hierarchical clustering, ward's linkage, Pearson correlation, 2000 iterations, 70% resampling of both items and features
- Pairwise matrices are symmetric binary matrices over the same 434-sample set for both methods being compared

## Limitations

- Consensus clustering requires a predefined K, so the comparison must sweep K=3 to K=7 rather than discovering the number of clusters
- Similarity of solutions is strongly dependent on the consensus clustering feature set size (K3 similar only when feature set exceeds 6000 genes)
- SRIQ, the comparator, is computation- and memory-intensive with large bag size, high permutation counts, and large datasets (samples and features)

## Evidence

- [results] Similarity finding: [evidence span withheld — anchor status: not_found]
