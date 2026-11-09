---
name: pairwise-co-clustering-matrix-construction
description: Use when two clustering solutions (e.g. SRIQ and consensus clustering)
  have been derived on the same sample set and you need a per-K quantitative similarity
  metric between their cluster partitions.
when_to_use_negative:
- The two clustering solutions were run on different sample sets — the Jaccard comparison
  requires identical sample IDs in both pairwise matrices.
- You only need to match cluster labels between solutions (label alignment) rather
  than quantify overall partition similarity.
- The clustering solution has not been assessed for stability — in SRIQ, 52 of 434
  samples were considered unstable and unstable samples should not be treated as reliable
  co-clustering members.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SRIQ
  role: Provides one of the two cluster solutions (systematic random forest integration
    to QT clustering) whose pairwise co-clustering matrix is compared against consensus
    clustering
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Used to load the expression data, build the binary pairwise similarity matrices,
    compute the Jaccard index, and write pair_similarity_scores.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:74f348408f916881564540f7f90c9f451ff0ec474034a5140f01a244f7a7c6b7
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pairwise-co-clustering-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds binary sample-by-sample co-clustering (pairwise similarity) matrices from two cluster solutions (SRIQ and consensus clustering) and compares them via a Jaccard-based Pair Similarity score to benchmark clustering methods on the same cohort, e.g. 434 TCGA-LUAD cases.

## When to use

Use when two clustering solutions (e.g. SRIQ and consensus clustering) have been derived on the same sample set and you need a per-K quantitative similarity metric between their cluster partitions.

## When NOT to use

- The two clustering solutions were run on different sample sets — the Jaccard comparison requires identical sample IDs in both pairwise matrices.
- You only need to match cluster labels between solutions (label alignment) rather than quantify overall partition similarity.
- The clustering solution has not been assessed for stability — in SRIQ, 52 of 434 samples were considered unstable and unstable samples should not be treated as reliable co-clustering members.

## Inputs

- Cluster label assignments from consensus clustering for each K (3–7) on the same sample IDs
- SRIQ cluster solution assignments (diameter 0.63, 6 clusters, core/spiral) on the same 434 samples
- Optionally, the preprocessed log2-transformed median-centered TCGA-LUAD expression matrix (434 samples, >25000 transcripts) used to generate both solutions

## Outputs

- Binary pairwise co-clustering similarity matrix per clustering solution (n x n, 1 = samples co-clustered)
- Jaccard-based Pair Similarity score per K (K=3–7)
- pair_similarity_scores.csv containing the Pair Similarity scores for K=3–7

## How to apply

First run both methods on the identical preprocessed (log2-transformed, median-centered) expression matrix, e.g. 434 TCGA-LUAD samples with >25000 transcripts: consensus clustering with hierarchical clustering, ward's linkage, Pearson correlation, 2000 iterations, and 70% resampling of both samples and genes for each K from 3 to 7, versus an SRIQ solution (diameter 0.63, 6 clusters, core/spiral assignments). For each K, convert each solution's cluster labels into a binary n x n pairwise matrix where entry (i,j)=1 if samples i and j co-occur in the same cluster. Compute the Jaccard index between the consensus pairwise matrix and the SRIQ pairwise matrix, which is reported as the Pair Similarity score per K. Write the scores for K=3–7 to pair_similarity_scores.csv. Interpretation: in the paper, the K3 solutions were relatively similar when the consensus clustering feature set exceeded 6000 genes, whereas K6 solutions never reached 60% similarity for any cluster, indicating method divergence at higher K.

## Related tools

- **SRIQ** (Provides one of the two cluster solutions (systematic random forest integration to QT clustering) whose pairwise co-clustering matrix is compared against consensus clustering) — https://github.com/StaafLab/SRIQ
- **Python** (Used to load the expression data, build the binary pairwise similarity matrices, compute the Jaccard index, and write pair_similarity_scores.csv)

## Examples

```
from sklearn.metrics import jaccard_score; score = jaccard_score(consensus_pairs.ravel(), sriq_pairs.ravel())
```

## Evaluation signals

- The pairwise matrix is n x n, symmetric, with 1s on the diagonal, where n matches the number of samples in both solutions (434 TCGA-LUAD cases).
- Pair Similarity scores are produced for every K from 3 to 7 and written to pair_similarity_scores.csv.
- Expected paper result: K3 similarity is relatively high when the consensus clustering feature set exceeds 6000 genes, while every K6 cluster shows <60% similarity — deviations warrant checking the consensus clustering parameters (2000 iterations, 70% resampling, ward's linkage, Pearson correlation).
- Sanity check: the Jaccard index between a solution and itself is 1.0, and Jaccard scores fall in [0, 1].
- The SRIQ assignments used must correspond to the reported solution parameters (diameter 0.63, 6 clusters, spiral mode, core/spiral members only).

## Limitations

- The comparison requires a predefined K for consensus clustering, whereas SRIQ evaluates solution stability without user-supplied K; scores are therefore only defined for fixed K values.
- SRIQ solutions are sensitive to the cluster diameter parameter, which must be considered in the context of the stability and occurrence of a solution across iterations; the Jaccard score depends on the chosen diameter (0.63 here).
- Unstable samples (52 of 434 in the paper) reduce the comparability of the two partitions and may deflate similarity scores.

## Evidence

- [results] K3 vs K6 divergence: [evidence span withheld — anchor status: not_found]
