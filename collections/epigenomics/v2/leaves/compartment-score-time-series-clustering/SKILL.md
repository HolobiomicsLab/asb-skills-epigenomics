---
name: compartment-score-time-series-clustering
description: Use when you have compartment (PCA) scores for the same genomic bins
  across ≥3 ordered time points or developmental stages and want to cluster bins by
  trajectory shape rather than compare transitions one at a time.
when_to_use_negative:
- You only have two Hi-C conditions/time points — use dcHiC's pairwise differential
  compartment testing (Mahalanobis distance with chi-square significance) instead
  of time-series clustering
- Compartment scores have not been quantile-normalized across datasets, so score scales
  are not comparable and z-transformed trajectories would be confounded by batch/scale
  differences
- The goal is calling significant differential compartments or differential interactions
  rather than grouping bins by temporal trajectory
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: dcHiC
  role: Produces the principal components and quantile-normalized compartment scores
    across all input Hi-C datasets that feed the time-series clustering; also provides
    time-series clustering as a downstream feature
  repo: https://github.com/ay-lab/dcHiC
- name: TCseq
  role: R package performing the temporal clustering of z-scored compartment-score
    trajectories to group bins with similar Early-to-Late dynamics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:9a57e29bad8cd9043d4a95099ab2e0fccd7312b7073b660fa2e8bf1c74c8387c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - TCseq
  - dcHiC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# compartment-score-time-series-clustering

## Summary

Groups genomic bins by how their dcHiC-derived, quantile-normalized compartment scores evolve across a multi-point time course, using z-transformation followed by TCseq time-series clustering. This enables simultaneous analysis of compartmentalization dynamics across all stages rather than pairwise transition comparisons.

## When to use

Use when you have compartment (PCA) scores for the same genomic bins across ≥3 ordered time points or developmental stages and want to cluster bins by trajectory shape rather than compare transitions one at a time.

## When NOT to use

- You only have two Hi-C conditions/time points — use dcHiC's pairwise differential compartment testing (Mahalanobis distance with chi-square significance) instead of time-series clustering
- Compartment scores have not been quantile-normalized across datasets, so score scales are not comparable and z-transformed trajectories would be confounded by batch/scale differences
- The goal is calling significant differential compartments or differential interactions rather than grouping bins by temporal trajectory

## Inputs

- Quantile-normalized compartment (PCA) scores per 100 kb bin for each developmental stage, produced by dcHiC
- Ordered list of time points / stage assignments (e.g. Early: days 1/7; Mid: days 28/56; Late: days 309/347)

## Outputs

- Cluster assignment table (compartment_clusters.csv) with one row per 100 kb bin: bin coordinates, assigned cluster label, z-scored compartment profile
- Per-cluster mean compartment-score profiles across the ordered stages
- Six major clusters of compartmentalization score patterns (in the published ESC-NPC-CN analysis)

## How to apply

Start from the quantile-normalized PC1 compartment scores at 100 kb resolution that dcHiC produced across all input Hi-C maps (dcHiC calculates principal components followed by quantile normalization across datasets). Z-transform each 100 kb bin's score vector across the six postnatal time points (days 1, 7, 28, 56, 309, 347), which are grouped into Early (1/7), Mid (28/56), and Late (309/347) stages, so clusters reflect relative dynamics instead of absolute score scales. Run the TCseq package's temporal clustering workflow on the z-scored profiles — in the ESC-NPC-CN cortical analysis this identified 6 major clusters of compartmentalization score patterns. Compute the mean compartment-score profile per cluster across the ordered stages to summarize each trajectory, and write a cluster assignment table (bin coordinates, assigned cluster, z-scored profile) plus per-cluster mean profiles to a file such as compartment_clusters.csv. Validate that per-cluster mean trajectories are distinct across Early/Mid/Late and recapitulate published stage-transition patterns such as Early-to-Late compartment switching (Fig. 5C, Fig. 8D,F).

## Related tools

- **dcHiC** (Produces the principal components and quantile-normalized compartment scores across all input Hi-C datasets that feed the time-series clustering; also provides time-series clustering as a downstream feature) — https://github.com/ay-lab/dcHiC
- **TCseq** (R package performing the temporal clustering of z-scored compartment-score trajectories to group bins with similar Early-to-Late dynamics)

## Examples

```
library(TCseq); clusters <- timeclust(t(zscores), algo="cm", k=6)
```

## Evaluation signals

- compartment_clusters.csv exists and contains one row per 100 kb bin, each with a cluster label and z-scored profile
- Number of clusters matches the expected pattern (6 major clusters in the published postnatal cortex analysis)
- Per-cluster mean profiles show distinct trajectories across the Early (1/7), Mid (28/56), and Late (309/347) stage groupings
- Clusters recapitulate the published stage-transition patterns, e.g. Early-to-Late compartment switching shown in Fig. 5C and Fig. 8D,F
- Input scores are confirmed to be quantile-normalized across all Hi-C datasets before z-transformation

## Limitations

- Pseudo-bulk single-cell Hi-C compartment calls are limited by contact-map sparsity; dcHiC analysis was demonstrated from as few as ~80-100 cells per condition, but sparsity remains a major issue for compartment identification
- Stage grouping is discrete (Early/Mid/Late from days 1, 7, 28, 56, 309, 347); trajectories within a stage pair are collapsed during clustering
- Analysis demonstrated only on mouse (and human) Hi-C data; applicability to other organisms or very large numbers of conditions (thousands of cells, tens of conditions) was deferred to future dcHiC extensions

## Evidence

- [intro] dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data: "In step 1, dcHiC calculates the principal components followed by the quantile normalization of the compartment scores across all input Hi-C data"
- [discussion] Time-series clustering of differential compartments across development stages: [evidence span withheld — anchor status: not_found]
- [methods] Stage grouping of six time points into early/middle/late maps: "We separated these into early (1/7), middle (28/56), and late (309/347) stage maps for each cell type -- six different clusters in total."
- [intro] dcHiC downstream feature list including time-series clustering: "including standalone IGV browser37 visualization of results, detection of differential interactions involving significant differential compartments, time-series clustering of compartment scores, and"
- [readme] dcHiC capabilities summary from README: "dcHiC is a tool for differential compartment analysis of Hi-C datasets. It features many capabilities, including:"
