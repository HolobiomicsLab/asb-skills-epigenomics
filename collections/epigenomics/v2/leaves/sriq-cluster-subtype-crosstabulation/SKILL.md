---
name: sriq-cluster-subtype-crosstabulation
description: Use when you have SRIQ core/spiral cluster memberships for TCGA-LUAD
  samples and need to test whether they partition within, or refine, the Wilkerson
  TRU/PI/PP consensus subtypes via a cluster-by-subtype contingency table.
when_to_use_negative:
- The dataset is not LUAD expression data or lacks the samples needed to compute Wilkerson
  centroid correlations — the TRU/PI/PP labels are LUAD-specific
- No SRIQ cluster solution with established stability exists yet; select and validate
  a solution (e.g., diameter 0.63, 6 clusters) via Clusters_Frequencies before cross-tabulating
- The goal is de novo subtype discovery rather than comparison against a preexisting
  consensus classification
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: SRIQ
  role: Produces the cluster solution (EXTRACT at diameter 0.63, 6 clusters) whose
    memberships are cross-tabulated against TRU/PI/PP labels
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Generates the log2-transformed, median-centered, zero-filtered ('_mc_log2_nz')
    expression matrix used for centroid correlation
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: pandas is used to compute the cluster-by-subtype contingency table and write
    subtype_sriq_crosstab.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:ea79a7dec497e70963da830351f2f52ae357dad796fd3d93bef88060fb701311
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sriq-cluster-subtype-crosstabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-tabulates SRIQ cluster assignments from the 6-cluster TCGA-LUAD solution (diameter 0.63, BagSize 1200, 10,000 permutations) against Wilkerson centroid-based TRU/PI/PP molecular subtype labels to quantify how SRIQ refines existing transcriptional subtypes.

## When to use

Use when you have SRIQ core/spiral cluster memberships for TCGA-LUAD samples and need to test whether they partition within, or refine, the Wilkerson TRU/PI/PP consensus subtypes via a cluster-by-subtype contingency table.

## When NOT to use

- The dataset is not LUAD expression data or lacks the samples needed to compute Wilkerson centroid correlations — the TRU/PI/PP labels are LUAD-specific
- No SRIQ cluster solution with established stability exists yet; select and validate a solution (e.g., diameter 0.63, 6 clusters) via Clusters_Frequencies before cross-tabulating
- The goal is de novo subtype discovery rather than comparison against a preexisting consensus classification

## Inputs

- Log2-transformed, median-centered LUAD expression matrix ('_mc_log2_nz' tab-separated .txt from SRIQPreprocess)
- Wilkerson centroid gene lists defining TRU, PI, and PP subtypes
- SRIQ cluster assignments from the 6-cluster solution at diameter 0.63 (Clusters_Frequencies / EXTRACT output)

## Outputs

- Per-sample TRU/PI/PP subtype assignments based on maximum Pearson centroid correlation
- Cluster-by-subtype contingency table written to subtype_sriq_crosstab.csv, with unstable (non-core/non-spiral) samples flagged separately

## How to apply

Load the log2-transformed, median-centered LUAD expression matrix (the '_mc_log2_nz' output of SRIQPreprocess with method=PEARSON) and restrict it to the Wilkerson centroid genes for the terminal respiratory unit (TRU), proximal-inflammatory (PI), and proximal-proliferative (PP) subtypes. Compute Pearson correlations between each of the 434 samples and each subtype centroid and assign each sample to the subtype with the highest correlation. Load the SRIQ 6-cluster solution selected from the Clusters_Frequencies output at diameter 0.63, flagging the 52 unstable samples that are neither core nor spiral separately from the 382 core/spiral samples. Cross-tabulate SRIQ cluster membership against the TRU/PI/PP calls with pandas and write the result to subtype_sriq_crosstab.csv. The crosstab is correct if SRIQ subclusters partition preferentially within TRU and PP while PI stays comparatively homogeneous.

## Related tools

- **SRIQ** (Produces the cluster solution (EXTRACT at diameter 0.63, 6 clusters) whose memberships are cross-tabulated against TRU/PI/PP labels) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Generates the log2-transformed, median-centered, zero-filtered ('_mc_log2_nz') expression matrix used for centroid correlation) — https://github.com/StaafLab/SRIQ
- **Python** (pandas is used to compute the cluster-by-subtype contingency table and write subtype_sriq_crosstab.csv)

## Examples

```
crosstab = pd.crosstab(sriq_clusters['cluster'], samples['wilkerson_subtype']); crosstab.to_csv('subtype_sriq_crosstab.csv')
```

## Evaluation signals

- Each of the 434 samples receives exactly one TRU/PI/PP label (the subtype with the highest Pearson centroid correlation), and the labels sum to 434 in the crosstab
- The contingency table core cells reconcile with 382 core/spiral and 52 unstable samples reported for the LUAD run
- SRIQ subclusters partition preferentially within TRU (e.g., a core cluster that is 100% TRU) and split PP into nearly equal core clusters, while PI remains comparatively homogeneous
- The crosstab CSV parses correctly with pandas (pd.read_csv) and its row totals equal SRIQ cluster sizes from the EXTRACT output

## Limitations

- SRIQ cluster solutions are sensitive to the cluster diameter parameter, which must be considered in the context of stability and occurrence across iterations
- Of the 434 samples, 52 were considered unstable and cannot be confidently assigned to a cluster, so their treatment in the crosstab must be flagged separately
- SRIQ is computation and memory intensive with large bag size, high permutation counts, and large datasets, and cannot detect small or underrepresented subgroups in smaller datasets

## Evidence
