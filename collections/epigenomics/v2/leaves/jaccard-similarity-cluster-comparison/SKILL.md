---
name: jaccard-similarity-cluster-comparison
description: Use when two clustering solutions (e.g., SRIQ vs consensus clustering)
  exist on the same sample set and you need a quantitative, per-K similarity metric
  for the derived cluster partitions.
when_to_use_negative:
- When only one clustering solution exists and there is no second partition to compare
  against
- When you need a biological validity assessment (e.g., differential expression via
  SAM or Cibersort immune fractions) rather than a technical similarity benchmark
  between methods
- When cluster assignments use non-overlapping sample sets, making a sample-level
  pairwise co-occurrence comparison undefined
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0622
tools:
- name: SRIQ
  role: Provides the reference cluster solution (diameter 0.63, 6 clusters, core/spiral
    assignments) via SRIQS.jar EXTRACT, whose assignments are converted to a binary
    pairwise matrix for Jaccard comparison
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Used with pandas to load the TCGA-LUAD expression matrix, build the binary
    co-occurrence matrices, and compute Jaccard Pair Similarity scores per K
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:205fbd36c56c9c23e2d5dd3faf65bb2ac3e0f6b5f142348a546237e3b9db3db8
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# jaccard-similarity-cluster-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark an SRIQ cluster solution against a consensus clustering solution on the same cohort (e.g., 434 TCGA-LUAD samples) by comparing binary sample-co-occurrence matrices with a Jaccard-based Pair Similarity score per K.

## When to use

Use when two clustering solutions (e.g., SRIQ vs consensus clustering) exist on the same sample set and you need a quantitative, per-K similarity metric for the derived cluster partitions.

## When NOT to use

- When only one clustering solution exists and there is no second partition to compare against
- When you need a biological validity assessment (e.g., differential expression via SAM or Cibersort immune fractions) rather than a technical similarity benchmark between methods
- When cluster assignments use non-overlapping sample sets, making a sample-level pairwise co-occurrence comparison undefined

## Inputs

- Preprocessed, log2-transformed, median-centered gene expression matrix (434 TCGA-LUAD samples, >25000 transcripts, tab-separated .txt)
- Consensus clustering cluster assignments per K (K=3..7)
- SRIQ cluster assignments extracted at a given diameter (e.g., 0.63), number of clusters (e.g., 6), and spiral setting

## Outputs

- Binary sample-by-sample co-occurrence (pairwise similarity) matrix per method
- Pair Similarity (Jaccard) score per K value
- pair_similarity_scores.csv listing Pair Similarity scores for K=3-7

## How to apply

First generate both solutions on identical preprocessed input: consensus clustering with hierarchical clustering, ward's linkage, Pearson correlation, 2000 iterations and 70% resampling of both items (samples) and features (genes) for each K (e.g., 3-7), and the SRIQ solution extracted with the chosen diameter (e.g., 0.63) and cluster count (e.g., 6, core/spiral assignments). Convert each solution into a binary pairwise similarity matrix whose entry (i,j) indicates whether samples i and j co-occur in the same cluster. Compute the Jaccard index between the consensus pairwise matrix and the SRIQ pairwise matrix to yield one Pair Similarity score per K, then write the scores for K=3-7 to a CSV (e.g., pair_similarity_scores.csv). Interpret scores against the paper's observation: SRIQ and consensus K3 solutions were relatively similar when the consensus feature set exceeded 6000 genes, whereas K6 solutions differed (<60% similarity for any cluster) irrespective of feature count.

## Related tools

- **SRIQ** (Provides the reference cluster solution (diameter 0.63, 6 clusters, core/spiral assignments) via SRIQS.jar EXTRACT, whose assignments are converted to a binary pairwise matrix for Jaccard comparison) — https://github.com/StaafLab/SRIQ
- **Python** (Used with pandas to load the TCGA-LUAD expression matrix, build the binary co-occurrence matrices, and compute Jaccard Pair Similarity scores per K)

## Examples

```
from sklearn.metrics import jaccard_score; ps_k = jaccard_score(sriq_pairwise.ravel(), consensus_pairwise.ravel())
```

## Evaluation signals

- Each solution yields a symmetric binary co-occurrence matrix with ones on the diagonal and matching dimensions across methods (sample count 434)
- One Pair Similarity (Jaccard) score is produced per K, and scores are written to pair_similarity_scores.csv covering K=3-7
- K6 comparison should reproduce the paper's expectation of <60% similarity for any cluster, while K3 similarity rises when the consensus feature set exceeds 6000 genes
- SRIQ assignments correspond to a valid extracted solution (e.g., output path Results_log_0.63_6) with 382 core/spiral samples and 52 unstable samples excluded or labeled consistently
- Jaccard score falls within [0, 1], with 1.0 only if the two partitions produce identical co-occurrence matrices

## Limitations

- Comparison depends on SRIQ's cluster diameter parameter, which must be considered in the context of the stability and occurrence of a solution across iterations
- SRIQ's computational and memory intensity with large bag size, high permutation counts, and large datasets limits scalability of repeated benchmarking
- Similarity of partitions does not by itself establish biological relevance; the article additionally assessed core-cluster biology via SAM one-versus-rest analysis and Cibersort immune fractions

## Evidence

- [results] SRIQ vs consensus K3/K6 similarity: [evidence span withheld — anchor status: not_found]
- [other] Jaccard workflow from source card: [evidence span withheld — anchor status: not_found]
