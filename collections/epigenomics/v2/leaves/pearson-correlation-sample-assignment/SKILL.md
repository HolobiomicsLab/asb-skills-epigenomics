---
name: pearson-correlation-sample-assignment
description: Use when each sample in a log2-transformed, median-centered tumor expression
  matrix (e.g., the SRIQ '_mc_log2_nz' output) must be labeled with a nearest-centroid
  TRU/PI/PP call to compare against an independent clustering solution.
when_to_use_negative:
- The input is raw FPKM/counts rather than the log2-transformed, median-centered '_mc_log2_nz'
  expression format SRIQ requires (off-set by 0.1 or values <1 set to 1, log2-transformed,
  median-centered)
- No Wilkerson centroid gene definitions are available, so TRU/PI/PP nearest-centroid
  correlation cannot be computed
- The goal is clustering itself rather than labeling samples against existing subtype
  centroids — use the SRIQ clustering skill instead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python (pandas)
  role: Loads the '_mc_log2_nz' expression matrix, computes Pearson correlations against
    TRU/PI/PP centroids, and writes the contingency table to subtype_sriq_crosstab.csv
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQPreprocess.jar
  role: Produces the log2-transformed, median-centered '_mc_log2_nz' input matrix
    on which the centroid correlation assignment is performed
  repo: https://github.com/StaafLab/SRIQ
- name: SRIQ
  role: Supplies the cluster solution (e.g., 6 clusters at diameter 0.63, BagSize
    1200, 10,000 permutations) that is cross-tabulated against the TRU/PI/PP labels
  repo: https://github.com/StaafLab/SRIQ
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:991716d0cfdc78384aa1b53ec6147cce3510905fa683b589335bbaf96fe8db61
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
  - Python (pandas)
  - SRIQ
  - SRIQPreprocess.jar
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pearson-correlation-sample-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each RNA-seq tumor sample to the Wilkerson terminal respiratory unit (TRU), proximal-inflammatory (PI), or proximal-proliferative (PP) molecular subtype by taking the argmax of Pearson correlations between the sample's expression vector (restricted to centroid genes) and each subtype centroid, enabling cross-tabulation against SRIQ cluster solutions in TCGA-LUAD.

## When to use

Use when each sample in a log2-transformed, median-centered tumor expression matrix (e.g., the SRIQ '_mc_log2_nz' output) must be labeled with a nearest-centroid TRU/PI/PP call to compare against an independent clustering solution.

## When NOT to use

- The input is raw FPKM/counts rather than the log2-transformed, median-centered '_mc_log2_nz' expression format SRIQ requires (off-set by 0.1 or values <1 set to 1, log2-transformed, median-centered)
- No Wilkerson centroid gene definitions are available, so TRU/PI/PP nearest-centroid correlation cannot be computed
- The goal is clustering itself rather than labeling samples against existing subtype centroids — use the SRIQ clustering skill instead

## Inputs

- Log2-transformed, median-centered gene expression matrix of TCGA-LUAD samples (tab-separated .txt '_mc_log2_nz' output of SRIQPreprocess.jar; 434 samples x >25,000 transcripts)
- Wilkerson et al. subtype centroid gene lists defining TRU, PI, and PP
- SRIQ cluster solution assignment per sample (selected from Clusters_Frequencies output, e.g., 6 clusters at diameter 0.63), including core/spiral/unstable flags

## Outputs

- Per-sample TRU/PI/PP subtype assignments (one label per each of the 434 samples)
- Cluster-by-subtype contingency table written to subtype_sriq_crosstab.csv with unstable (non-core/non-spiral) samples flagged separately

## How to apply

Load the log2-transformed, median-centered LUAD expression matrix (the '_mc_log2_nz' file produced by SRIQPreprocess.jar) with pandas in Python. Restrict the matrix to the Wilkerson et al. centroid genes defining the TRU, PI, and PP subtypes, then compute the Pearson correlation between every sample and each of the three subtype centroids. Assign each of the 434 TCGA-LUAD samples to the subtype with the highest Pearson correlation (per-sample argmax). Cross-tabulate these per-sample TRU/PI/PP labels against the SRIQ cluster solution (e.g., the 6-cluster solution at diameter 0.63 from the 10,000-permutation, BagSize-1200 run in Clusters_Frequencies), flagging the unstable non-core/non-spiral samples separately, and write the cluster-by-subtype contingency table to subtype_sriq_crosstab.csv. The rationale is that nearest-centroid correlation on published centroid genes provides an independent, standard subtype ground truth against which SRIQ's refinement of existing transcriptional subtypes can be assessed.

## Related tools

- **Python (pandas)** (Loads the '_mc_log2_nz' expression matrix, computes Pearson correlations against TRU/PI/PP centroids, and writes the contingency table to subtype_sriq_crosstab.csv) — https://github.com/StaafLab/SRIQ
- **SRIQPreprocess.jar** (Produces the log2-transformed, median-centered '_mc_log2_nz' input matrix on which the centroid correlation assignment is performed) — https://github.com/StaafLab/SRIQ
- **SRIQ** (Supplies the cluster solution (e.g., 6 clusters at diameter 0.63, BagSize 1200, 10,000 permutations) that is cross-tabulated against the TRU/PI/PP labels) — https://github.com/StaafLab/SRIQ

## Examples

```
import pandas as pd; M = pd.read_csv('GeneData_mc_log2_nz.txt', sep='\t', index_col=0); r = M.loc[centroid_genes].corrwith(centroid_tru, axis=1)
```

## Evaluation signals

- Every one of the 434 samples receives exactly one TRU/PI/PP label (the argmax correlation is well-defined per sample)
- The crosstab shows SRIQ subclusters partitioning preferentially within TRU and PP samples while PI remains comparatively homogeneous
- Cluster 5 contains 100% TRU core samples, and the PP subtype splits into three nearly equally large core subsets (clusters 2, 4, and 6)
- Unstable samples (52 of 434 classified neither core nor spiral) are flagged separately in the contingency table
- subtype_sriq_crosstab.csv exists and its row/column sums reconcile with the 434-sample cohort and the SRIQ cluster sizes

## Limitations

- The SRIQ cluster solution being cross-tabulated is sensitive to the cluster diameter parameter, which must be evaluated against stability and occurrence across iterations
- Samples classified as unstable (52 of 434) lie outside core/spiral clusters and cannot be cleanly attributed to a subcluster in the crosstab
- Small datasets limit the possibility to detect small or underrepresented biological subgroups, constraining subtype-refinement conclusions in smaller cohorts

## Evidence

- [results] SRIQ stratifies the TRU TCGA subtype and separates the PP TCGA subtype into three nearly equally large subsets: [evidence span withheld — anchor status: not_found]
