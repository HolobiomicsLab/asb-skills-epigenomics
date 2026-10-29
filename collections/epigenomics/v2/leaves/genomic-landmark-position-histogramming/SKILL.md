---
name: genomic-landmark-position-histogramming
description: Use when evaluating whether a DTW-based clustering/alignment of epigenomic
  peak regions has sharpened the positions of annotated genomic landmarks (TSS, FSS)
  within each cluster.
when_to_use_negative:
- Regions where the landmark positions are not fully contained — only regions where
  all POI points overlap completely with the main region are valid for matching
- Peaks excluded by the DGW filters (length < 5 or > 1000 bins, fewer than 10 counts,
  or no bin with more than 10 reads) — the POI statistics would not reflect the clustered
  data
- When the goal is landmark-free unsupervised alignment quality assessment rather
  than validation against annotated genomic features
edam_operation: http://edamontology.org/operation_0417
tools:
- name: DGW
  role: Provides dgw-explorer for interactive dendrogram cutting, POI overlay on heat-maps
    (-poi parameter), and pre/post-warping POI position histograms
  repo: https://github.com/lukauskas/dgw
- name: Python
  role: Language in which DGW and its explorer are implemented (requires Python 2.7)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:6f1dae23a4ad9683573f849b1d5cf6f205cdde326f5d463ea35b94735b66c914
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_0417
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Genomic landmark position histogramming

## Summary

Overlay points of interest (e.g. transcription start sites, first splice sites) on DGW-explorer cluster heat-maps and build per-cluster position histograms of these landmarks before and after DTW warping to quantify alignment quality.

## When to use

Use when evaluating whether a DTW-based clustering/alignment of epigenomic peak regions has sharpened the positions of annotated genomic landmarks (TSS, FSS) within each cluster.

## When NOT to use

- Regions where the landmark positions are not fully contained — only regions where all POI points overlap completely with the main region are valid for matching
- Peaks excluded by the DGW filters (length < 5 or > 1000 bins, fewer than 10 counts, or no bin with more than 10 reads) — the POI statistics would not reflect the clustered data
- When the goal is landmark-free unsupervised alignment quality assessment rather than validation against annotated genomic features

## Inputs

- {prefix}_config.dgw dgw-explorer configuration file
- {prefix}_dataset.pd, {prefix}_linkage.npy, {prefix}_prototypes.pickle, {prefix}_warping_paths.pickle supporting files
- POI regions file (poi_regions.bed) containing landmark positions such as TSSs and first splice sites

## Outputs

- Cluster heat-map figure (dgw_explorer_cluster_heatmaps.png) with up to two POI sets overlaid
- Per-cluster POI position histograms computed pre-warping (raw data rescaled to maximum region length) and post-warping
- Per-cluster BED files and prototype/heat-map data saved to the output directory

## How to apply

Launch dgw-explorer with the {prefix}_config.dgw configuration (with {prefix}_dataset.pd, {prefix}_linkage.npy, {prefix}_prototypes.pickle and {prefix}_warping_paths.pickle in the same directory), then cut the dendrogram near the leaves (the paper reports a cut yielding 45 clusters). For each cluster, overlay up to two sets of points of interest via the -poi parameter (e.g. first splice sites in black, transcription start sites in white) on the original and warped coverage heat-maps. Generate POI position histograms pre- and post-warping, rescaling the raw data to the maximum region length before histogramming so pre/post histograms are comparable. Rationale: if DTW warping genuinely captures landmark structure, the warped position distributions should concentrate — the paper reports entropy decreases of 12.91% (TSS) and 7.72% (FSS) in the selected cluster. Landmarks must fall completely inside the analysed peak region to be counted; regions with no POI entries can be dropped with --ignore-no-poi-regions.

## Related tools

- **DGW** (Provides dgw-explorer for interactive dendrogram cutting, POI overlay on heat-maps (-poi parameter), and pre/post-warping POI position histograms) — https://github.com/lukauskas/dgw
- **Python** (Language in which DGW and its explorer are implemented (requires Python 2.7))

## Examples

```
dgw-explorer {prefix}_config.dgw -poi fss.poi,tss.poi --ignore-no-poi-regions
```

## Evaluation signals

- The exported figure dgw_explorer_cluster_heatmaps.png exists and is a valid PNG
- Warped POI position histograms show reduced entropy relative to the raw histograms (paper reports 12.91% TSS and 7.72% FSS entropy decrease in the selected cluster)
- Raw POI positions are rescaled to the maximum region length before histogramming, so pre- and post-warping histograms share a common scale
- Only regions whose POI entries are completely contained within the main region contribute to the histograms

## Limitations

- Dendrogram cut level is a difficult choice; the paper's entropy improvements (12.91% TSS, 7.72% FSS) are reported for one selected cluster, not globally
- Regions lacking any entry in fss.poi are ignored (or must be dropped with --ignore-no-poi-regions), biasing landmark statistics to fully contained regions
- DGW requires Python 2.7, and the dtwco dependency repository is no longer maintained

## Evidence

- [methods] Entropy of TSS positions decreased 12.91% and FSS 7.72% after warping in the selected cluster: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [methods] Complete-overlap containment requirement for POI matching: "find all the regions in `poi_regions.bed` that overlap *completely* (i.e. *all* points in the points of interest region are contained"
- [abstract] Landmark recognition from histone marks: "DGW automatically recognises and aligns important genomic features such as transcription start sites and splicing sites from histone marks."
