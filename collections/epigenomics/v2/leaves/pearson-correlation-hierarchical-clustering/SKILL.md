---
name: pearson-correlation-hierarchical-clustering
description: Use when clustering a log2-transformed, median-centered gene expression
  matrix (samples × transcripts) with a Pearson-correlation-distance hierarchical
  method, and when you need stable cluster assignments across K values to compare
  against an alternative clustering method.
when_to_use_negative:
- The data are raw counts or FPKM not yet off-set, log2-transformed, and median-centered
  — SRIQ-compatible clustering requires the '_mc_log2_nz' preprocessing first
- You need to discover the number of clusters without predefining K — a stated limitation
  of consensus/bootstrap clustering is the requirement of a predefined parameter K
- The dataset is very high-dimensional without prior filtering — distance values become
  meaningless (curse of dimensionality), so transcripts should first be filtered (e.g.,
  removing non-expressed and very low-variation transcripts as done to retain >25000
  transcripts)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements the consensus clustering runs (Pearson distance, ward linkage,
    2000 iterations, 70% resampling) and the Jaccard pairwise comparison against SRIQ
    solutions
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQ
  role: Comparator clustering method whose diameter-0.63, 6-cluster core/spiral solution
    is converted to a binary pairwise matrix for Jaccard Pair Similarity scoring
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:81b17cb987afb7a747c918ab880835f134765d79e841e886eaa7442b2ae2dfc5
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pearson-correlation-hierarchical-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark clustering of tumor gene expression matrices using hierarchical clustering with Pearson correlation distance and ward's linkage under iterative item/feature resampling (consensus clustering), used in the SRIQ paper to generate reference K=3–7 solutions on 434 TCGA-LUAD samples for Jaccard-based Pair Similarity comparison.

## When to use

Use when clustering a log2-transformed, median-centered gene expression matrix (samples × transcripts) with a Pearson-correlation-distance hierarchical method, and when you need stable cluster assignments across K values to compare against an alternative clustering method.

## When NOT to use

- The data are raw counts or FPKM not yet off-set, log2-transformed, and median-centered — SRIQ-compatible clustering requires the '_mc_log2_nz' preprocessing first
- You need to discover the number of clusters without predefining K — a stated limitation of consensus/bootstrap clustering is the requirement of a predefined parameter K
- The dataset is very high-dimensional without prior filtering — distance values become meaningless (curse of dimensionality), so transcripts should first be filtered (e.g., removing non-expressed and very low-variation transcripts as done to retain >25000 transcripts)

## Inputs

- Preprocessed gene expression matrix (tab-separated, genes × samples), log2-transformed and median-centered, e.g., 434 TCGA-LUAD RNA-seq samples with >25000 retained transcripts
- Comparator cluster assignments (e.g., SRIQ solution with diameter 0.63, 6 clusters, core/spiral sample labels)
- Target K values (e.g., K = 3 to 7)

## Outputs

- Consensus cluster label assignments per sample for each K
- Binary pairwise co-clustering (sample × sample) similarity matrix per solution
- Jaccard Pair Similarity scores per K (e.g., written to pair_similarity_scores.csv)

## How to apply

Load the preprocessed expression matrix (e.g., the 434-sample TCGA-LUAD matrix with >25000 transcripts, off-set, log2-transformed, median-centered as SRIQ requires). Run consensus clustering using hierarchical clustering with Pearson correlation distance and ward's linkage, 2000 iterations, and 70% resampling of both items (samples) and features (genes), repeating for each K from 3 to 7. From the resulting consensus solution, build a binary pairwise co-clustering matrix where each entry indicates whether two samples occur in the same cluster. Compare this binary matrix to the pairwise matrix of a comparator solution (e.g., SRIQ output at diameter 0.63 with 6 clusters, core/spiral assignments) using the Jaccard index to yield a Pair Similarity score per K, then export the scores. The Pearson metric is also the distance metric SRIQ itself uses, which is why it is the appropriate basis for the head-to-head comparison.

## Related tools

- **Python** (Implements the consensus clustering runs (Pearson distance, ward linkage, 2000 iterations, 70% resampling) and the Jaccard pairwise comparison against SRIQ solutions) — https://github.com/StaafLab/SRIQ
- **SRIQ** (Comparator clustering method whose diameter-0.63, 6-cluster core/spiral solution is converted to a binary pairwise matrix for Jaccard Pair Similarity scoring) — https://github.com/StaafLab/SRIQ

## Examples

```
from scipy.cluster.hierarchy import linkage, fcluster; Z = linkage(X.T, method='ward', metric='correlation'); labels = fcluster(Z, t=3, criterion='maxclust')
```

## Evaluation signals

- Each sample in the 434-case matrix receives exactly one cluster label per K, and all K=3–7 runs use identical parameters (Pearson distance, ward's linkage, 2000 iterations, 70% item and feature resampling)
- The pairwise co-clustering matrix is symmetric, binary (0/1), with a diagonal of 1s and N(N-1)/2 comparable off-diagonal pairs
- Jaccard Pair Similarity scores fall in [0, 1]; the paper's K3 result (>60% similarity when consensus features >6000 genes) versus K6 (<60% similarity for any cluster) provides a sanity reference
- pair_similarity_scores.csv contains one score row per K from 3 to 7 with no missing values
- Expression matrix invariants hold before clustering: values log2-transformed, median-centered per transcript, and >25000 transcripts retained after filtering non-expressed/low-variation genes

## Limitations

- Consensus/bootstrap clustering requires a predefined number of clusters (K), limiting unsupervised subtype discovery
- Distance values of high-dimensional data become meaningless (curse of dimensionality), risking overfitting and unstable clusters if transcripts are not filtered beforehand
- Solutions can be sensitive to comparator parameters (e.g., SRIQ diameter must be evaluated against solution stability and occurrence across iterations), affecting Pair Similarity comparability
- Similarity outcomes depend on the consensus clustering feature set size; K3 similarity exceeded 60% only when the consensus feature set exceeded 6000 genes

## Evidence

- [results] The SRIQ and consensus clustering K3 solutions were relatively similar when the consensus clustering feature set exceeded 6000 genes: [evidence span withheld — anchor status: not_found]
