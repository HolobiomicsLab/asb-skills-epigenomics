---
name: pseudo-bulk-schic-development-stage-analysis
description: Use when you have pseudo-bulk single-cell Hi-C maps at multiple developmental
  time points and need to group genomic bins by dynamic compartmentalization patterns
  rather than comparing transitions pairwise.
when_to_use_negative:
- When the goal is pairwise differential compartment calling between only two Hi-C
  conditions rather than multi-point time-course trajectory clustering
- When single-cell maps are too sparse to form pseudo-bulk profiles with at least
  ~80-100 cells per condition, since genomic compartment identification from single-cell
  Hi-C is limited by sparsity
- When the input is already a cluster assignment or feature table rather than raw/pseudo-bulk
  Hi-C contact maps
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: dcHiC
  role: Computes principal components and quantile-normalized compartment scores across
    all input pseudo-bulk Hi-C stage maps for differential compartment analysis
  repo: https://github.com/ay-lab/dcHiC
- name: TCseq
  role: Time-series clustering of z-scored 100 kb bin compartment-score trajectories
    into 6 clusters across Early/Mid/Late stages
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:2f2f6b8982e7ea1f062643f0021141534c67c0a07ad174230e8683f5d427c566
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - TCseq
  - dcHiC
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pseudo-bulk-schic-development-stage-analysis

## Summary

Analyzes pseudo-bulk single-cell Hi-C maps across post-natal mouse brain development (days 1, 7, 28, 56, 309, 347) by using dcHiC to compute quantile-normalized compartment scores at 100 kb resolution and TCseq to time-series-cluster z-scored compartment trajectories across Early/Mid/Late stages into 6 clusters.

## When to use

Use when you have pseudo-bulk single-cell Hi-C maps at multiple developmental time points and need to group genomic bins by dynamic compartmentalization patterns rather than comparing transitions pairwise.

## When NOT to use

- When the goal is pairwise differential compartment calling between only two Hi-C conditions rather than multi-point time-course trajectory clustering
- When single-cell maps are too sparse to form pseudo-bulk profiles with at least ~80-100 cells per condition, since genomic compartment identification from single-cell Hi-C is limited by sparsity
- When the input is already a cluster assignment or feature table rather than raw/pseudo-bulk Hi-C contact maps

## Inputs

- Pseudo-bulk single-cell Hi-C contact maps for postnatal mouse cortex/hippocampus at days 1, 7, 28, 56, 309, 347 (at least ~80-100 cells per condition)
- Quantile-normalized compartment scores at 100 kb resolution per stage computed by dcHiC
- Stage grouping of time points into Early (1/7), Mid (28/56), Late (309/347)

## Outputs

- compartment_clusters.csv: one row per 100 kb bin with bin coordinates, assigned cluster label, and z-scored compartment profile
- Per-cluster mean compartment-score profiles across the ordered developmental stages
- 6 major clusters of compartmentalization score patterns recapitulating stage-transition dynamics

## How to apply

First create pseudo-bulk maps from the postnatal neuronal scHi-C data and separate the six time points into early (days 1/7), middle (28/56), and late (309/347) stage maps for each cell type. Run dcHiC, which calculates principal components followed by quantile normalization of compartment scores across all input Hi-C maps, yielding comparable compartment scores per 100 kb bin at each stage. Z-transform each bin's compartment-score vector across the six time points so clustering reflects relative dynamics rather than absolute score scales, then run time-series clustering with the TCseq package to identify 6 major clusters of compartmentalization score patterns. Compute the mean compartment-score profile per cluster across the ordered Early/Mid/Late stages to summarize each cluster's trajectory, and write a cluster assignment table (bin coordinates, cluster label, z-scored profile) with per-cluster mean profiles to compartment_clusters.csv. dcHiC supports this workflow from pseudo-bulk profiles of as few as ~80-100 cells per condition.

## Related tools

- **dcHiC** (Computes principal components and quantile-normalized compartment scores across all input pseudo-bulk Hi-C stage maps for differential compartment analysis) — https://github.com/ay-lab/dcHiC
- **TCseq** (Time-series clustering of z-scored 100 kb bin compartment-score trajectories into 6 clusters across Early/Mid/Late stages)

## Examples

```
library(TCseq); cl <- timeclust(x = t(zscored_compartment_profiles), k = 6, metric = "euclidean", method = "pam")
```

## Evaluation signals

- compartment_clusters.csv exists and contains exactly one row per 100 kb bin, each with a cluster label and z-scored profile
- Exactly 6 major clusters are recovered, matching the paper's reported clustering of differential compartments across Early, Mid, and Late stages
- Per-cluster mean profiles show distinct trajectories across Early/Mid/Late stages comparable to the paper's Fig. 5C and Fig. 8D,F stage-transition patterns
- Pseudo-bulk maps were built from at least ~80-100 cells per condition; fewer cells risks unreliable compartment calls
- Z-scored profiles have zero mean and unit variance per bin across the six time points, confirming clustering reflects relative dynamics

## Limitations

- Genomic compartment identification from single-cell Hi-C maps remains limited by sparsity of single-cell contact maps, even at coarse resolution
- dcHiC's demonstrated use cases and this analysis were limited to human and mouse Hi-C data
- Compartment comparison across thousands of cells and tens of conditions is deferred to future extensions of dcHiC

## Evidence

- [intro] dcHiC calculates principal components followed by quantile normalization of compartment scores across all input Hi-C data: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [methods] Stage grouping of time points: "We separated these into early (1/7), middle (28/56), and late (309/347) stage maps for each cell type -- six different clusters in total."
- [methods] Pseudo-bulk map creation: "Using postnatal neuronal scHi-C data, we created pseudo-bulk maps for analysis."
- [discussion] Minimum cells per condition: "dcHiC is able to call compartments and perform differential analysis from pseudo-bulk profiles of as low as 80 cells per condition"
- [readme] dcHiC pseudo-bulk capability in README: "Comprehensive identification of significant compartment changes between any number of cell lines (with replicates), including with pseudo-bulk single cell data"
