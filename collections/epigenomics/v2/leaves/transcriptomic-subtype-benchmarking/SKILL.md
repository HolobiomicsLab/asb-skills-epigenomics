---
name: transcriptomic-subtype-benchmarking
description: Use when a newly developed unsupervised clustering method for tumor gene
  expression data must be benchmarked against an established method (e.g., consensus
  clustering) by quantifying solution concordance per number of clusters K.
when_to_use_negative:
- The clustering solutions being compared are not defined over the same sample set
  — Jaccard on pairwise co-clustering matrices requires identical sample indices across
  methods
- You need an absolute quality assessment of a single clustering solution rather than
  a relative comparison between two methods on the same cohort
- The dataset is too small to support resampling-based consensus clustering (2000
  iterations at 70% item/feature resampling)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
tools:
- name: SRIQ
  role: Novel clustering method under evaluation; its core/spiral cluster assignments
    (diameter 0.63, 6 clusters) form one side of the pairwise Jaccard comparison
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Implements consensus clustering setup, builds binary pairwise co-clustering
    matrices, computes Jaccard Pair Similarity scores, and writes pair_similarity_scores.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:0b2d123c2eac0ac90b026043cd5cf753f99ab11e88e8b7fb21e47e76f8b64e9d
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
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# transcriptomic-subtype-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a novel unsupervised transcriptomic clustering method (SRIQ) against a consensus clustering baseline on the same gene expression cohort by comparing cluster solutions with a Jaccard-based Pair Similarity score across K values (here K=3–7 on 434 TCGA-LUAD cases).

## When to use

Use when a newly developed unsupervised clustering method for tumor gene expression data must be benchmarked against an established method (e.g., consensus clustering) by quantifying solution concordance per number of clusters K.

## When NOT to use

- The clustering solutions being compared are not defined over the same sample set — Jaccard on pairwise co-clustering matrices requires identical sample indices across methods
- You need an absolute quality assessment of a single clustering solution rather than a relative comparison between two methods on the same cohort
- The dataset is too small to support resampling-based consensus clustering (2000 iterations at 70% item/feature resampling)

## Inputs

- Preprocessed, log2-transformed, median-centered gene expression matrix (tab-separated .txt, genes x samples; 434 TCGA-LUAD samples, >25,000 transcripts)
- Cluster assignments from the method under test (e.g., SRIQ output for diameter 0.63, 6 clusters, core/spiral sample labels)
- Consensus clustering solutions for each K in the evaluated range (K=3–7)

## Outputs

- Binary pairwise co-clustering similarity matrices for each clustering solution
- Pair Similarity scores (Jaccard index) per K value
- pair_similarity_scores.csv with Pair Similarity scores for K=3–7

## How to apply

Load the preprocessed, log2-transformed, median-centered expression matrix (here 434 TCGA-LUAD samples, >25,000 transcripts after removing non-expressed and low-variation transcripts) with pandas. Run the reference method — consensus clustering with hierarchical clustering, ward's linkage, Pearson correlation distance, 2000 iterations, and 70% resampling of both items (samples) and features (genes) — for each K from 3 to 7. Build a binary pairwise co-clustering similarity matrix for each solution, where an entry indicates whether two samples occur in the same cluster; do the same for the method under test (SRIQ solution at diameter 0.63, 6 clusters, core/spiral assignments). Compute the Jaccard index between the two pairwise matrices per K to yield a Pair Similarity score, and write the scores (K=3–7) to a CSV. Interpret concordance in light of the feature set size: in the source study the K3 solutions were relatively similar when the consensus clustering feature set exceeded 6000 genes, whereas the K6 solutions differed with <60% similarity for any cluster.

## Related tools

- **SRIQ** (Novel clustering method under evaluation; its core/spiral cluster assignments (diameter 0.63, 6 clusters) form one side of the pairwise Jaccard comparison) — https://github.com/StaafLab/SRIQ
- **Python** (Implements consensus clustering setup, builds binary pairwise co-clustering matrices, computes Jaccard Pair Similarity scores, and writes pair_similarity_scores.csv)

## Examples

```
java -jar SRIQS.jar EXTRACT "F:/test/LUAD_test/test.properties" false 0.63 6 "F:/test/LUAD_test/newFiltered_35k.txt"
```

## Evaluation signals

- Pair Similarity score for K3 exceeds ~60% only when the consensus clustering feature set exceeds 6000 genes; below this the K3 scores drop
- All K6 Pair Similarity scores remain <60% for any consensus clustering feature set size, confirming method discordance at higher K
- Consensus clustering run reproduces the exact parameters: hierarchical clustering, ward's linkage, Pearson correlation, 2000 iterations, 70% resampling of both samples and genes
- Each Jaccard score is computed between binary pairwise matrices of identical dimension (434 x 434 sample pairs) across the two methods
- pair_similarity_scores.csv contains one row per K value from 3 to 7

## Limitations

- Consensus/bootstrap clustering requires a predefined number of clusters (K), limiting subtype discovery; SRIQ instead evaluates solution stability internally
- SRIQ solutions are sensitive to the cluster diameter parameter, which must be evaluated against solution stability and occurrence across iterations
- Consensus clustering resampling (2000 iterations, 70% of items and features) is computationally expensive, and SRIQ with large bag size, high permutations, and large datasets is computation and memory intensive
- High-dimensional distance values become meaningless (curse of dimensionality), which can drive overfitting and unstable clusters in distance-based clustering

## Evidence

- [results] K3 similarity depends on feature set size: [evidence span withheld — anchor status: not_found]
