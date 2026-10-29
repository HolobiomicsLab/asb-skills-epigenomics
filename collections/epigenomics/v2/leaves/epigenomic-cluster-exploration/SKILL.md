---
name: epigenomic-cluster-exploration
description: Use when DGW clustering outputs ({prefix}_config.dgw, {prefix}_dataset.pd,
  {prefix}_linkage.npy, {prefix}_prototypes.pickle, {prefix}_warping_paths.pickle)
  exist and you need to visually inspect, validate and export shape-based clusters
  of epigenomic peaks.
when_to_use_negative:
- No DGW clustering artefacts (linkage, prototypes, warping paths) exist yet — run
  the DGW clustering pipeline first
- The goal is de novo peak calling or alignment without visual/interactive cluster
  inspection (use MACS2 + dgw clustering command instead)
- Regions lack any points of interest and POI-based entropy validation is the objective
  (regions without POI entries are ignored)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: DGW
  role: dgw-explorer renders dendrograms, cluster prototypes, original and warped
    heat-maps, POI overlays and exports cluster data and figures
  repo: https://github.com/lukauskas/dgw
- name: Python
  role: Runtime for DGW; the package requires Python 2.7
- name: matplotlib
  role: Rendering of dendrogram, heat-map windows and exported cluster heat-map figure
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:996dcff4c4405f3163491d4224a547cb35cb8a9845d5e983b79ef73454d00f52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# epigenomic-cluster-exploration

## Summary

Interactive exploration of DGW-produced epigenomic mark clusters using dgw-explorer: cut the dendrogram, inspect per-cluster prototypes and warped heat-maps, overlay points of interest (e.g. TSS, first splice sites), and export aligned cluster heat-map figures.

## When to use

Use when DGW clustering outputs ({prefix}_config.dgw, {prefix}_dataset.pd, {prefix}_linkage.npy, {prefix}_prototypes.pickle, {prefix}_warping_paths.pickle) exist and you need to visually inspect, validate and export shape-based clusters of epigenomic peaks.

## When NOT to use

- No DGW clustering artefacts (linkage, prototypes, warping paths) exist yet — run the DGW clustering pipeline first
- The goal is de novo peak calling or alignment without visual/interactive cluster inspection (use MACS2 + dgw clustering command instead)
- Regions lack any points of interest and POI-based entropy validation is the objective (regions without POI entries are ignored)

## Inputs

- {prefix}_config.dgw configuration file
- {prefix}_dataset.pd dataset file
- {prefix}_linkage.npy hierarchical clustering linkage matrix
- {prefix}_prototypes.pickle cluster prototypes
- {prefix}_warping_paths.pickle DTW warping paths
- POI BED file(s) (e.g. transcription start sites, first splice sites)

## Outputs

- dgw_explorer_cluster_heatmaps.png aligned heat-map figure
- Cluster prototypes, heat-maps and per-cluster BED files in the output directory
- POI position histograms before and after warping per cluster

## How to apply

Launch dgw-explorer with the {prefix}_config.dgw file, ensuring {prefix}_dataset.pd, {prefix}_linkage.npy, {prefix}_prototypes.pickle and {prefix}_warping_paths.pickle reside in the same directory. Render the dendrogram and peak heat-map window, then cut the dendrogram interactively at a level near the leaves (the paper's example yields 45 clusters). For each cluster open the four-panel cluster explorer showing the cluster prototype (from scaled prioritised shape averaging), the average DTW projection onto the prototype, and heat-maps of original and warped per-dataset coverage. Overlay up to two POI sets via the -poi parameter (e.g. first splice sites in black, transcription start sites in white) and generate POI position histograms pre- and post-warping, rescaling raw data to the maximum region length before histogramming. Save prototypes, heat-maps and per-cluster BED files, and export the aligned heat-map figure as dgw_explorer_cluster_heatmaps.png.

## Related tools

- **DGW** (dgw-explorer renders dendrograms, cluster prototypes, original and warped heat-maps, POI overlays and exports cluster data and figures) — https://github.com/lukauskas/dgw
- **Python** (Runtime for DGW; the package requires Python 2.7)
- **matplotlib** (Rendering of dendrogram, heat-map windows and exported cluster heat-map figure)

## Examples

```
dgw-explorer --poi fss.poi -poi tss.poi --ignore-no-poi-regions {prefix}_config.dgw
```

## Evaluation signals

- dgw_explorer_cluster_heatmaps.png exists and is a valid PNG
- Warped POI position histograms show reduced entropy relative to raw histograms (paper reports 12.91% TSS and 7.72% FSS entropy decrease in the selected cluster)
- Per-cluster BED files, prototypes and heat-maps are present in the output directory
- Dendrogram cut yields the expected number of clusters (paper reports 45 clusters as appropriate for its data)

## Limitations

- Cutting the dendrogram at an appropriate level is a difficult choice and is done interactively
- Regions that do not completely contain the points of interest are dropped from POI-based analysis
- DGW requires Python 2.7; related repositories (dtwco) are archived and no longer maintained

## Evidence

- [abstract] DGW uses Dynamic Time Warping to adaptively rescale and align genomic distances which allows to group regions of interest with similar shapes: "DGW uses Dynamic Time Warping to adaptively rescale and align genomic distances which allows to group regions of interest with similar shapes"
- [methods] we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [methods] here we use the scaled prioritised shape averaging algorithm of [18]: "here we use the scaled prioritised shape averaging algorithm of [18]"
- [readme] This is the repository for the open source DGW library developped by Saulius Lukauskas.: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
- [methods] find all the regions in `poi_regions.bed` that overlap *completely* (i.e. *all* points in the points of interest region are contained within the main region): "find all the regions in `poi_regions.bed` that overlap *completely* (i.e. *all* points in the points of interest region are contained within the main region)"
