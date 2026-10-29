---
name: temporal-cluster-profile-summarization
description: Use when you have compartment scores for the same genomic bins across
  ≥3 ordered conditions/time points (e.g. dcHiC quantile-normalized PCA scores at
  100 kb for days 1/7/28/56/309/347) and want to cluster bins by trajectory shape
  rather than compare pairs of transitions.
when_to_use_negative:
- Only two time points/conditions are available — pairwise differential compartment
  testing with dcHiC's Mahalanobis-distance significance is the appropriate analysis,
  not trajectory clustering
- The scores are not comparable across conditions (no quantile normalization has been
  applied), so clusters would reflect scale differences rather than dynamics
- Input is raw Hi-C contact maps rather than per-bin compartment scores — compartment
  identification/PCA must be run first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Produces the quantile-normalized PCA compartment scores per bin across all
    input Hi-C maps that are the input to the clustering
  repo: https://github.com/ay-lab/dcHiC
- name: TCseq
  role: R package whose temporal clustering workflow groups z-scored compartment-score
    trajectories into clusters
- name: IGV
  role: dcHiC outputs standalone IGV-based HTML browser views that can be used to
    inspect clustered/differential compartments along the genome
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:dff12c2fc94a646e78533769150c53fa46fde8ce9d793e7476f81e06c961eada
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - IGV
  - TCseq
  - dcHiC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# temporal-cluster-profile-summarization

## Summary

Groups genomic bins with similar compartment-score trajectories across a multi-point time course by z-transforming dcHiC's quantile-normalized PCA scores and running time-series clustering with the TCseq workflow, then summarizing each cluster's mean Early-to-Late profile.

## When to use

Use when you have compartment scores for the same genomic bins across ≥3 ordered conditions/time points (e.g. dcHiC quantile-normalized PCA scores at 100 kb for days 1/7/28/56/309/347) and want to cluster bins by trajectory shape rather than compare pairs of transitions.

## When NOT to use

- Only two time points/conditions are available — pairwise differential compartment testing with dcHiC's Mahalanobis-distance significance is the appropriate analysis, not trajectory clustering
- The scores are not comparable across conditions (no quantile normalization has been applied), so clusters would reflect scale differences rather than dynamics
- Input is raw Hi-C contact maps rather than per-bin compartment scores — compartment identification/PCA must be run first

## Inputs

- Quantile-normalized compartment (PCA) scores from dcHiC for each 100 kb bin across all time points of the Hi-C time course
- Ordered time-point labels / stage groupings (e.g. Early: days 1 and 7; Mid: days 28 and 56; Late: days 309 and 347)

## Outputs

- Cluster assignment table (compartment_clusters.csv) with one row per 100 kb bin: bin coordinates, assigned cluster label, and z-scored profile
- Per-cluster mean compartment-score profiles across the ordered stages

## How to apply

First obtain quantile-normalized compartment scores per bin from dcHiC, which calculates principal components and quantile-normalizes compartment scores across all input Hi-C data. Z-transform each 100 kb bin's score vector across the six developmental time points (grouped into Early days 1/7, Mid days 28/56, Late days 309/347) so that clustering reflects relative dynamics instead of absolute score scale. Run time-series clustering on the z-scored profiles with the TCseq package, which groups bins with similar Early-to-Late trajectories (the paper resolved 6 clusters for the ESC-NPC-CN mouse brain time course). Compute the mean compartment-score profile per cluster across the ordered stages to summarize cluster dynamics, and write a cluster assignment table (bin coordinates, cluster label, z-scored profile) plus per-cluster mean profiles (e.g. compartment_clusters.csv). The rationale is that dcHiC's differential-compartment calls flag outlier bins via Mahalanobis distance, but time-series clustering simultaneously captures how scores evolve over all transitions at once.

## Related tools

- **dcHiC** (Produces the quantile-normalized PCA compartment scores per bin across all input Hi-C maps that are the input to the clustering) — https://github.com/ay-lab/dcHiC
- **TCseq** (R package whose temporal clustering workflow groups z-scored compartment-score trajectories into clusters)
- **IGV** (dcHiC outputs standalone IGV-based HTML browser views that can be used to inspect clustered/differential compartments along the genome)

## Evaluation signals

- compartment_clusters.csv exists and contains exactly one row per 100 kb bin with a non-empty cluster label
- Per-cluster mean profiles are visually and numerically distinct across the Early/Mid/Late stage groupings
- The number of clusters (e.g. 6 in the paper's ESC-NPC-CN analysis) is reported and each cluster has non-trivial membership
- Clusters recapitulate published stage-transition patterns such as Early-to-Late compartment switching (Fig. 5C, Fig. 8D,F)
- Z-transformation was applied per bin across time points, so each bin's profile has comparable scale before clustering

## Limitations

- Compartment identification from single-cell Hi-C maps is limited by data sparsity, so pseudo-bulk aggregation (as few as ~80–100 cells per condition) is needed before scoring and clustering
- Clustering reflects relative (z-scored) dynamics, not absolute compartment strength; clusters do not by themselves assign A/B compartment labels
- The published application analyzed human and mouse Hi-C data only; scalability to thousands of cells and tens of conditions was deferred to future extensions of dcHiC

## Evidence

- [abstract] dcHiC enables high-resolution compartment analysis as well as standalone browser visualization, differential interaction identification and time-series clustering: "dcHiC enables high-resolution compartment analysis as well as standalone browser visualization, differential interaction identification and time-series clustering"
- [methods] Stage grouping into six clusters: "We separated these into early (1/7), middle (28/56), and late (309/347) stage maps for each cell type -- six different clusters in total."
- [discussion] Time-series clustering across three developmental stages: "The time series clustering of the differential compartments across three stages (‘Early’, ‘Mid’, and ‘Late’) of development"
- [intro] Quantile normalization of compartment scores: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [readme] dcHiC tool definition: "dcHiC is a tool for differential compartment analysis of Hi-C datasets."
