---
name: tad-boundary-delineation-from-cluster-labels
description: Use when you have per-bin cluster labels from HDBSCAN on TOAST graph
  auto-encoder embeddings plus the source Hi-C matrix resolution (50 Kb, 25 Kb, or
  5 Kb), and need to delineate TAD boundaries for visualization or evaluation.
when_to_use_negative:
- You only have raw Hi-C contact matrices and no per-bin cluster labels — run the
  TOAST data preprocessing, GAE, and clustering modules first
- You need TADs shorter than 200 Kb — the method explicitly discards contiguous regions
  at or below 200 Kb
- The task is comparing TAD callers (Armatus, Arrowhead, TopDom, etc.) rather than
  producing TOAST boundaries
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: Provides the TAD generation module that groups contiguous same-label bins
    into TADs and delineates boundaries; toast.py in /src runs identification and
    plotting
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Produces the per-bin cluster labels that contiguous-region grouping consumes
- name: Pandas
  role: Used in the TOAST environment (Pandas 1.1.5) for handling per-bin label and
    boundary tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:d3bdc9192e52432c74cb0b1bfda3d84df4ea4590a6778c57fb7a5cf6a171025d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - HDBSCAN
  - Pandas
  - TOAST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tad-boundary-delineation-from-cluster-labels

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-bin HDBSCAN cluster labels from the TOAST pipeline into TAD boundaries by grouping contiguous same-label genomic bins into regions, retaining only regions longer than 200 Kb, and writing them to tad_boundaries.txt for downstream visualization.

## When to use

Use when you have per-bin cluster labels from HDBSCAN on TOAST graph auto-encoder embeddings plus the source Hi-C matrix resolution (50 Kb, 25 Kb, or 5 Kb), and need to delineate TAD boundaries for visualization or evaluation.

## When NOT to use

- You only have raw Hi-C contact matrices and no per-bin cluster labels — run the TOAST data preprocessing, GAE, and clustering modules first
- You need TADs shorter than 200 Kb — the method explicitly discards contiguous regions at or below 200 Kb
- The task is comparing TAD callers (Armatus, Arrowhead, TopDom, etc.) rather than producing TOAST boundaries

## Inputs

- Per-bin HDBSCAN cluster labels from TOAST's clustering module
- Resolution of the source Hi-C contact matrix (50 Kb, 25 Kb, or 5 Kb)
- NxN intra-chromosomal tab-separated Hi-C contact matrix (input context for the bin indices)

## Outputs

- tad_boundaries.txt with records formatted as (start bin, start location, end bin, end location)
- TAD boundary calls used for heatmap/TAD visualization

## How to apply

Load the per-bin HDBSCAN cluster labels together with the resolution of the source Hi-C contact matrix (e.g., 50 Kb, 25 Kb, or 5 Kb). Group consecutive genomic bins sharing the same cluster label into contiguous regions, following the TOAST TAD generation module. Compute each region's genomic length as (number of bins) x (matrix resolution) and apply the article's threshold: only contiguous regions whose length exceeds 200 Kb are identified as TADs — at 50 Kb resolution this means at least four same-label consecutive bins. Write retained TADs to tad_boundaries.txt with each record holding four fields: start bin, start location, end bin, end location. In the TOAST repository this delineation and plotting is done by the toast.py script in /src, where you change the input and output folders.

## Related tools

- **TOAST** (Provides the TAD generation module that groups contiguous same-label bins into TADs and delineates boundaries; toast.py in /src runs identification and plotting) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Produces the per-bin cluster labels that contiguous-region grouping consumes)
- **Pandas** (Used in the TOAST environment (Pandas 1.1.5) for handling per-bin label and boundary tables)

## Examples

```
python toast.py  # in /src, with input and output folders edited to point at the HDBSCAN-labeled matrix and tad_boundaries.txt output
```

## Evaluation signals

- tad_boundaries.txt exists and each record contains all four required fields: start bin, start location, end bin, end location
- Every retained TAD spans more than 200 Kb (e.g., at 50 Kb resolution, at least four consecutive same-label bins)
- TAD endpoints align with transitions between HDBSCAN cluster labels across consecutive genomic bins
- Downstream visualization (plot_heatmaps.py heatmaps) shows the called TADs overlaid correctly on the Hi-C contact matrix
- On Wang et al. simulated data with ground truth, TAD number difference should be small (1–8) and FMS near 0.98

## Limitations

- Regions at or below 200 Kb are discarded, so small TADs are systematically missed
- Delineation inherits noise from HDBSCAN labels, so mis-clustered bins can split or merge true TADs
- The threshold is resolution-dependent: the minimum bin count changes with matrix resolution (4 bins at 50 Kb, 8 at 25 Kb, 40 at 5 Kb)

## Evidence
