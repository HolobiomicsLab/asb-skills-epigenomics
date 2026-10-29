---
name: contingency-table-subtype-refinement-analysis
description: Use when you have an unsupervised cluster solution (e.g., SRIQ core/spiral
  labels) and an established molecular subtype scheme (TRU/PI/PP centroid calls) and
  need to quantify subtype-level enrichment or refinement per cluster.
when_to_use_negative:
- You only have raw FPKM counts — centroid correlations require the log2-transformed,
  median-centered '_mc_log2_nz' data, not unnormalized expression
- You lack a preexisting subtype scheme or centroid gene lists — the analysis is a
  comparison against a reference classification, not de novo subtype discovery
- Your clustering output has no core/spiral/stable sample designation, so unstable
  samples cannot be separated before tabulation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Python
  role: Loads the _mc_log2_nz expression matrix with pandas, computes Pearson correlations
    to Wilkerson centroids, and builds the cluster-by-subtype contingency table
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQ
  role: Produces the cluster solution (BagSize 1200, 10,000 permutations; EXTRACT
    at diameter 0.63, 6 clusters) whose membership is cross-tabulated against TRU/PI/PP
    calls
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Generates the log2-transformed, median-centered, non-zero-filtered '_mc_log2_nz'
    input matrix used for both clustering and centroid correlation
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:13b3f7ba9ed731a4a7dfb0fcb656197cc5e799fece4938450d7d1954623d1528
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
  - Java
  - Python
  - SRIQ
  - SRIQPreprocess.jar
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# contingency-table-subtype-refinement-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the Wilkerson TRU/PI/PI centroid classification of TCGA-LUAD samples and cross-tabulates it against an SRIQ cluster solution to test whether SRIQ refines existing transcriptional subtypes into biologically meaningful subgroups.

## When to use

Use when you have an unsupervised cluster solution (e.g., SRIQ core/spiral labels) and an established molecular subtype scheme (TRU/PI/PP centroid calls) and need to quantify subtype-level enrichment or refinement per cluster.

## When NOT to use

- You only have raw FPKM counts — centroid correlations require the log2-transformed, median-centered '_mc_log2_nz' data, not unnormalized expression
- You lack a preexisting subtype scheme or centroid gene lists — the analysis is a comparison against a reference classification, not de novo subtype discovery
- Your clustering output has no core/spiral/stable sample designation, so unstable samples cannot be separated before tabulation

## Inputs

- Log2-transformed, median-centered expression matrix ('_mc_log2_nz' output of SRIQPreprocess.jar), tab-separated gene-by-sample txt (434 TCGA LUAD samples, ~25,785 transcripts)
- Wilkerson et al. subtype centroid gene lists for TRU, PI, and PP
- SRIQ cluster assignment file extracted from the Clusters_Frequencies solution (e.g., diameter 0.63, 6 clusters, spiral=false, from a 10,000-permutation BagSize-1200 run)

## Outputs

- Per-sample TRU/PI/PP subtype labels from maximum Pearson correlation to centroids
- Cluster-by-subtype contingency table (SRIQ cluster x TRU/PI/PP) with unstable samples flagged
- subtype_sriq_crosstab.csv containing per-sample assignments and the contingency table

## How to apply

Load the log2-transformed, median-centered LUAD expression matrix (the SRIQPreprocess '_mc_log2_nz' output, computed with method=PEARSON) in pandas, restrict it to the Wilkerson centroid genes defining TRU, PI, and PP, and compute Pearson correlations between each of the 434 samples and each subtype centroid. Assign each sample to the subtype with the highest correlation to obtain per-sample TRU/PI/PP labels. Load the SRIQ solution from the Clusters_Frequencies output (e.g., the 6-cluster solution at diameter 0.63 from a 10,000-permutation, BagSize-1200 run), then cross-tabulate cluster membership against subtype calls in a contingency table, flagging the unstable (non-core/non-spiral) samples separately (of 434 samples, 382 were core/spiral and 52 unstable). Validation: SRIQ clusters should partition preferentially within TRU and PP samples — e.g., a 100% TRU core cluster and PP split into three nearly equal core clusters — while PI stays comparatively homogeneous.

## Related tools

- **Python** (Loads the _mc_log2_nz expression matrix with pandas, computes Pearson correlations to Wilkerson centroids, and builds the cluster-by-subtype contingency table) — https://github.com/StaafLab/SRIQ
- **SRIQ** (Produces the cluster solution (BagSize 1200, 10,000 permutations; EXTRACT at diameter 0.63, 6 clusters) whose membership is cross-tabulated against TRU/PI/PP calls) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Generates the log2-transformed, median-centered, non-zero-filtered '_mc_log2_nz' input matrix used for both clustering and centroid correlation) — https://github.com/StaafLab/SRIQ

## Examples

```
java -jar SRIQS.jar EXTRACT "F:/test/LUAD_test/test.properties" false 0.63 6 "F:/test/LUAD_test/newFiltered_35k.txt"
```

## Evaluation signals

- Contingency table shows SRIQ subclusters partitioning preferentially within TRU and PP while PI remains comparatively homogeneous
- At least one core cluster is nearly or fully TRU (paper example: cluster 5 is 100% TRU core samples)
- PP samples split into roughly three equally large core clusters (paper example: clusters 2, 4, and 6)
- Every one of the 434 samples receives exactly one TRU/PI/PP label (highest Pearson correlation wins)
- 52 samples are flagged unstable and reported separately, consistent with 382 core/spiral + 52 unstable of 434

## Limitations

- Cluster solutions are sensitive to the cluster diameter parameter, which must be considered in the context of stability and occurrence across iterations
- SRIQ cannot detect small or underrepresented biological subgroups in smaller datasets
- Only Pearson and Euclidean distance metrics are implemented; additional metrics are not yet available

## Evidence

- [other] SRIQ subclusters partitioning preferentially within TRU and PP samples while PI remains comparatively homogeneous: [evidence span withheld — anchor status: not_found]
