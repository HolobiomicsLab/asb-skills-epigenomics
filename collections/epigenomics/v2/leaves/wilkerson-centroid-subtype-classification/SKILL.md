---
name: wilkerson-centroid-subtype-classification
description: Use when you need per-sample TRU/PI/PP molecular subtype calls for a
  LUAD log2-transformed, median-centered expression matrix (e.g. an SRIQ '_mc_log2_nz'
  file), or when testing whether a new clustering solution partitions within existing
  consensus subtypes.
when_to_use_negative:
- Input expression data are raw counts or FPKM values that have not been offset, log2-transformed,
  and median-centered (SRIQ requires log2 median-centered values, so centroid correlations
  would be invalid)
- The cohort is not lung adenocarcinoma — the TRU/PI/PP centroids are LUAD-specific
  and should not be applied to other tumor types
- The goal is de novo subtype discovery rather than comparison against the preexisting
  consensus-defined TRU/PI/PP classifications
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SRIQ
  role: Produces the clustering solution (6 clusters, diameter 0.63) whose membership
    is cross-tabulated against the Wilkerson TRU/PI/PP calls; its '_mc_log2_nz' preprocessed
    matrix is the correlation input
  repo: https://github.com/StaafLab/SRIQ
- name: Python
  role: Pandas is used to load the median-centered matrix, compute Pearson correlations
    of each sample against the three subtype centroids, and write the crosstab CSV
- name: Java
  role: Runs SRIQPreprocess.jar and SRIQS.jar (preprocessing and EXTRACT of cluster
    assignments) via the command line
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:c1f815a00f00286b4cadf9b1be712942528bc9e95c9c7163f45cfefdca96a051
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# wilkerson-centroid-subtype-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each LUAD sample as terminal respiratory unit (TRU), proximal-inflammatory (PI), or proximal-proliferative (PP) by Pearson correlation against the Wilkerson et al. subtype centroid genes, then cross-tabulates the labels against an SRIQ cluster solution to test whether SRIQ refines existing transcriptional subtypes.

## When to use

Use when you need per-sample TRU/PI/PP molecular subtype calls for a LUAD log2-transformed, median-centered expression matrix (e.g. an SRIQ '_mc_log2_nz' file), or when testing whether a new clustering solution partitions within existing consensus subtypes.

## When NOT to use

- Input expression data are raw counts or FPKM values that have not been offset, log2-transformed, and median-centered (SRIQ requires log2 median-centered values, so centroid correlations would be invalid)
- The cohort is not lung adenocarcinoma — the TRU/PI/PP centroids are LUAD-specific and should not be applied to other tumor types
- The goal is de novo subtype discovery rather than comparison against the preexisting consensus-defined TRU/PI/PP classifications

## Inputs

- Log2-transformed, median-centered LUAD expression matrix ('_mc_log2_nz' output of SRIQPreprocess; tab-separated .txt with genes as rows, TCGA sample barcodes as columns)
- Wilkerson et al. centroid gene lists defining TRU, PI, and PP subtypes
- SRIQ cluster assignment file for the selected solution (e.g. 6 clusters at diameter 0.63, BagSize 1200, 10,000 permutations, extracted via SRIQS.jar EXTRACT)

## Outputs

- Per-sample TRU/PI/PP subtype assignments for all 434 LUAD samples
- Cluster-by-subtype contingency table (SRIQ cluster x TRU/PI/PP) written to subtype_sriq_crosstab.csv, with unstable samples flagged separately

## How to apply

Load the log2-transformed, median-centered expression matrix (the '_mc_log2_nz' output of SRIQPreprocess, computed with method=PEARSON) with pandas in Python. Restrict the matrix to the Wilkerson centroid genes defining TRU, PI, and PP, and compute the Pearson correlation between every sample and each of the three subtype centroids. Assign each of the 434 TCGA-LUAD samples to the subtype with the highest Pearson correlation to obtain per-sample labels. Load the SRIQ cluster solution (e.g. the 6-cluster solution at diameter 0.63 from the BagSize-1200, 10,000-permutation run in the Clusters_Frequencies output) and build a contingency table of SRIQ cluster membership versus TRU/PI/PP calls, flagging the 52 unstable (non-core/non-spiral) samples separately. Write per-sample assignments and the cluster-by-subtype crosstab to subtype_sriq_crosstab.csv.

## Related tools

- **SRIQ** (Produces the clustering solution (6 clusters, diameter 0.63) whose membership is cross-tabulated against the Wilkerson TRU/PI/PP calls; its '_mc_log2_nz' preprocessed matrix is the correlation input) — https://github.com/StaafLab/SRIQ
- **Python** (Pandas is used to load the median-centered matrix, compute Pearson correlations of each sample against the three subtype centroids, and write the crosstab CSV)
- **Java** (Runs SRIQPreprocess.jar and SRIQS.jar (preprocessing and EXTRACT of cluster assignments) via the command line)

## Examples

```
import pandas as pd; mat = pd.read_csv('GeneData_mc_log2_nz.txt', sep='\t', index_col=0).loc[wilkerson_genes]; labels = centroids.T.corr(mat).idxmax(); labels.to_csv('subtype_sriq_crosstab.csv')
```

## Evaluation signals

- Every one of the 434 samples receives exactly one subtype label, determined by the highest Pearson correlation among the three centroids
- The crosstab shows SRIQ subclusters partitioning preferentially within TRU and PP samples (e.g. cluster 5 is 100% TRU core samples) while PI remains comparatively homogeneous
- The PP subtype splits into three nearly equally large subsets across core clusters 2, 4, and 6, consistent with refinement of existing transcriptional subtypes
- Unstable samples (the 52 non-core/non-spiral samples out of 434; 382 were core or spiral) are flagged separately rather than force-assigned to a cluster
- The output file subtype_sriq_crosstab.csv contains both per-sample assignments and the cluster-by-subtype contingency table

## Limitations

- SRIQ cluster solutions are sensitive to the cluster diameter parameter, which must be considered in the context of stability and occurrence across iterations
- SRIQ cannot circumvent the general problem of smaller datasets limiting detection of small or underrepresented biological subgroups
- Centroid assignment is nearest-centroid only; borderline samples with near-equal correlations to two subtypes receive no ambiguity flag in this workflow

## Evidence

- [full_text] SRIQ stratifies the TRU TCGA subtype (e.g., cluster 5 is 100% TRU core samples) and separates the PP TCGA subtype into three nearly equally large subsets (core clusters 2, 4, and 6): [evidence span withheld — anchor status: not_found]
