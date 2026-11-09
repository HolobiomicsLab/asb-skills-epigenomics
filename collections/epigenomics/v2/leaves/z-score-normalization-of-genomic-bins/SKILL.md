---
name: z-score-normalization-of-genomic-bins
description: Use when you have dcHiC quantile-normalized compartment scores for multiple
  ordered time points (e.g., an ESC-NPC-CN developmental course) and want to cluster
  bins by trajectory shape before time-series clustering.
when_to_use_negative:
- Comparing only two Hi-C maps or a single condition — standard dcHiC differential
  compartment analysis (Mahalanobis distance significance) is the right choice, not
  trajectory clustering
- Compartment scores have not been quantile-normalized jointly across all input Hi-C
  data, so score scales are not comparable and z-transformation would fold in batch/scale
  artifacts
- Data are unordered or non-temporal samples (e.g., a panel of unrelated LCLs) where
  TCseq trajectory clustering is inappropriate
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Produces the quantile-normalized compartment (PCA) scores at 100 kb resolution
    that are z-transformed per bin
  repo: https://github.com/ay-lab/dcHiC
- name: TCseq
  role: R package used to time-series cluster the z-scored compartment profiles into
    the 6 major trajectory clusters
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:303add40042ef596b8c6fa8e12790dac01eedd98bb7d0cef7da111c9ef86fa85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - TCseq
  - dcHiC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# z-score-normalization-of-genomic-bins

## Summary

Z-transform the quantile-normalized compartment scores of each 100 kb genomic bin across the six developmental time points (days 1, 7, 28, 56, 309, 347) so that downstream time-series clustering (TCseq) groups bins by relative compartmentalization dynamics rather than absolute A/B score scales.

## When to use

Use when you have dcHiC quantile-normalized compartment scores for multiple ordered time points (e.g., an ESC-NPC-CN developmental course) and want to cluster bins by trajectory shape before time-series clustering.

## When NOT to use

- Comparing only two Hi-C maps or a single condition — standard dcHiC differential compartment analysis (Mahalanobis distance significance) is the right choice, not trajectory clustering
- Compartment scores have not been quantile-normalized jointly across all input Hi-C data, so score scales are not comparable and z-transformation would fold in batch/scale artifacts
- Data are unordered or non-temporal samples (e.g., a panel of unrelated LCLs) where TCseq trajectory clustering is inappropriate

## Inputs

- dcHiC quantile-normalized compartment (PCA) scores at 100 kb resolution for each time point/stage of the ESC-NPC-CN time course
- Bin coordinate annotation (chromosome, start, end) for each 100 kb bin
- Ordered time-point/stage labels (Early: days 1/7; Mid: days 28/56; Late: days 309/347)

## Outputs

- Z-scored compartment-score profile per 100 kb bin across the six time points
- Cluster assignment table (bin coordinates, assigned cluster, z-scored profile) written to compartment_clusters.csv
- Per-cluster mean compartment-score profiles across Early/Mid/Late stages

## How to apply

Load the dcHiC output of quantile-normalized PCA compartment scores computed at 100 kb resolution across all six Hi-C datasets (dcHiC calculates principal components followed by quantile normalization of compartment scores across all input Hi-C data). For each 100 kb bin, transpose the six-point score vector and z-transform it (subtract the bin's mean, divide by its SD across days 1/7, 28/56, 309/347) so that clusters reflect relative dynamics rather than absolute score scales. Order the time points into Early (days 1/7), Mid (days 28/56), and Late (days 309/347) stages to match the paper's stage grouping. Feed the z-scored bin-by-time matrix into TCseq time-series clustering (the paper recovers 6 major clusters of compartmentalization score patterns). Finally compute per-cluster mean z-scored profiles across the ordered stages to summarize each trajectory.

## Related tools

- **dcHiC** (Produces the quantile-normalized compartment (PCA) scores at 100 kb resolution that are z-transformed per bin) — https://github.com/ay-lab/dcHiC
- **TCseq** (R package used to time-series cluster the z-scored compartment profiles into the 6 major trajectory clusters)

## Examples

```
Rscript -e 'scores <- read.csv("dchic_quantile_norm_100kb.csv"); z <- t(scale(t(scores[, time_columns])))'
```

## Evaluation signals

- compartment_clusters.csv exists and contains one row per 100 kb bin with a valid cluster label
- Each bin's z-scored vector across the six time points has mean ≈ 0 and SD ≈ 1, confirming per-bin standardization
- Per-cluster mean profiles show distinct trajectories across Early (days 1/7), Mid (days 28/56), and Late (days 309/347) stages
- Clusters recover ~6 major compartmentalization score patterns, recapitulating the published Early-to-Late compartment switching (Fig. 5C, Fig. 8D,F)

## Limitations

- Time-series clustering of compartment dynamics operates on pseudo-bulk profiles; single-cell Hi-C sparsity still limits compartment identification itself
- Clustering at 100 kb resolution reflects relative changes only — bins within the same A/B type can be separated or merged depending on trajectory, and absolute compartment strength is discarded by the z-transform

## Evidence

- [results] the quantile normalized PCA scores for each 100 kb bin are z-transformed and TC-seq is applied to identify 6 major clusters: [evidence span withheld — anchor status: not_found]
- [methods] Z-transform each 100 kb bin's compartment score vector across the six time points: [evidence span withheld — anchor status: not_found]
- [intro] dcHiC calculates the principal components followed by the quantile normalization: "dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [discussion] The time series clustering of the differential compartments across three stages: "The time series clustering of the differential compartments across three stages (‘Early’, ‘Mid’, and ‘Late’) of development"
