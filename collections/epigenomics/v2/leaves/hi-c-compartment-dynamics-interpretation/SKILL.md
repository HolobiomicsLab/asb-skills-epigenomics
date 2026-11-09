---
name: hi-c-compartment-dynamics-interpretation
description: Use when you have compartment scores (e.g., 100 kb bin PCA scores) for
  three or more ordered Hi-C time points and need to cluster bins by their relative
  trajectory across the full time course instead of comparing single stage transitions.
when_to_use_negative:
- You only have two Hi-C conditions — use dcHiC's Mahalanobis-distance differential
  compartment test instead of time-series clustering.
- Compartment scores have not been quantile-normalized across all input Hi-C maps
  — raw per-map PCA scores are not comparable and clustering would reflect scale artifacts
  rather than dynamics.
- The goal is detecting statistically significant differential compartments rather
  than grouping trajectory patterns; use dcHiC's Mahalanobis distance with chi-square
  testing and IHW/FDR correction.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3922
tools:
- name: dcHiC
  role: Computes principal components and quantile-normalizes compartment scores across
    all input Hi-C datasets, providing the normalized 100 kb bin score matrix for
    clustering
  repo: https://github.com/ay-lab/dcHiC
- name: TCseq
  role: R package used for temporal clustering of the z-scored compartment score profiles
    to identify major clusters of compartmentalization dynamics across Early/Mid/Late
    stages
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:c8bbe0526efcb0815757308740c5cff98744da605c30bdd9918a85f4b87cfc22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - TCseq
  - dcHiC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# hi-c-compartment-dynamics-interpretation

## Summary

Groups dynamic compartmentalization score patterns across a multi-point Hi-C time course by z-transforming dcHiC's quantile-normalized PCA scores per genomic bin and applying TCseq time-series clustering, so all developmental transitions are compared simultaneously rather than pairwise.

## When to use

Use when you have compartment scores (e.g., 100 kb bin PCA scores) for three or more ordered Hi-C time points and need to cluster bins by their relative trajectory across the full time course instead of comparing single stage transitions.

## When NOT to use

- You only have two Hi-C conditions — use dcHiC's Mahalanobis-distance differential compartment test instead of time-series clustering.
- Compartment scores have not been quantile-normalized across all input Hi-C maps — raw per-map PCA scores are not comparable and clustering would reflect scale artifacts rather than dynamics.
- The goal is detecting statistically significant differential compartments rather than grouping trajectory patterns; use dcHiC's Mahalanobis distance with chi-square testing and IHW/FDR correction.

## Inputs

- Quantile-normalized compartment (PCA) scores at 100 kb resolution per developmental stage from dcHiC
- Pseudo-bulk Hi-C maps from single-cell Hi-C time points (e.g., Tan et al. cortex and hippocampus days 1, 7, 28, 56, 309, 347)
- Stage grouping metadata (Early, Mid, Late)

## Outputs

- compartment_clusters.csv with one row per 100 kb bin containing bin coordinates, assigned cluster label, and z-scored profile
- Per-cluster mean compartment-score profiles across the ordered Early/Mid/Late stages
- Cluster assignments recapitulating published stage-transition patterns (e.g., Early-to-Late compartment switching)

## How to apply

Start from the quantile-normalized compartment scores at 100 kb resolution produced by dcHiC (dcHiC calculates principal components followed by quantile normalization of compartment scores across all input Hi-C data). Z-transform each 100 kb bin's compartment score vector across the six time points (e.g., days 1/7 = Early, 28/56 = Mid, 309/347 = Late for the Tan et al. mouse cortex/hippocampus scHi-C data) so clusters reflect relative dynamics rather than absolute score scales. Run time-series clustering with the TCseq package to group bins with similar Early-to-Late trajectories — the published ESC-NPC-CN analysis identified 6 major clusters of compartmentalization score patterns. Compute the mean compartment-score profile per cluster across the ordered stages to summarize each cluster's dynamics, and write a cluster assignment table (bin coordinates, assigned cluster, z-scored profile) plus per-cluster mean profiles to a file such as compartment_clusters.csv.

## Related tools

- **dcHiC** (Computes principal components and quantile-normalizes compartment scores across all input Hi-C datasets, providing the normalized 100 kb bin score matrix for clustering) — https://github.com/ay-lab/dcHiC
- **TCseq** (R package used for temporal clustering of the z-scored compartment score profiles to identify major clusters of compartmentalization dynamics across Early/Mid/Late stages)

## Examples

```
library(TCseq); clusters <- timeclust(t(zscores), algo='tpx', k=6)
```

## Evaluation signals

- compartment_clusters.csv exists and contains one row per 100 kb bin with a cluster label
- Per-cluster mean profiles show distinct trajectories across Early/Mid/Late stages
- Number of clusters is on the order of the published analysis (6 major clusters) for the ESC-NPC-CN time course
- Clusters recapitulate published stage-transition patterns such as Early-to-Late compartment switching shown in Fig. 5C and Fig. 8D,F
- Z-transformation applied per bin across the six time points so cluster distances reflect relative, not absolute, score scales

## Limitations

- Genomic compartment identification from single-cell Hi-C maps is limited by sparsity, even at coarse resolution; pseudo-bulk aggregation is needed and dcHiC works down to ~80-100 cells per condition
- Companion dcHiC analysis was limited to human and mouse Hi-C data, though the method is stated to be readily applicable to Hi-C data or its variants
- Clusters summarize relative compartment-score dynamics only; biological interpretation (e.g., linked gene regulation, chromatin states, replication timing) requires orthogonal integration

## Evidence

- [intro] In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [intro] Time-series clustering as a dcHiC downstream feature: "including standalone IGV browser37 visualization of results, detection of differential interactions involving significant differential compartments, time-series clustering of compartment scores, and"
- [methods] Stage grouping used in the paper: "We separated these into early (1/7), middle (28/56), and late (309/347) stage maps for each cell type -- six different clusters in total."
- [abstract] dcHiC enables time-series clustering capability: "dcHiC enables high-resolution compartment analysis as well as standalone browser visualization, differential interaction identification and time-series clustering"
