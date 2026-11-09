---
name: dtw-warped-profile-visualization
description: Use when epigenomic ChIP-Seq peak regions (e.g. ENCODE H3K4me3/H3K9ac
  K562) have already been DGW-aligned and clustered and you need to visually inspect
  DTW-warped heat-maps, cluster prototypes, and validate alignment against TSS/FSS
  points of interest.
when_to_use_negative:
- Raw BAM/ChIP-Seq data that has not yet been peak-called (MACS2), binned into 50
  bp windows, and clustered — run the DGW worker first to produce the .dgw artefacts
- Data lacking points of interest — regions without any entry in fss.poi should be
  dropped with --ignore-no-poi-regions, so POI entropy validation is impossible without
  annotation
- Datasets where standard feature tables or pre-computed profiles suffice and shape-based
  alignment adds no value
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
tools:
- name: DGW
  role: Runs dgw-explorer, the interactive dendrogram/heat-map viewer used to cut
    clusters and render DTW-warped profiles
  repo: https://github.com/lukauskas/dgw
- name: Python
  role: DGW is a stand-alone Python package (requires Python 2.7) in which dgw-explorer
    and its matplotlib-based visualisations run
  repo: https://github.com/lukauskas/dgw
- name: matplotlib
  role: Rendering of dendrograms, four-panel cluster heat-maps, POI overlays and histogram
    figures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:cd7804a4f05e0840c713c7e40b84528f466dee1da5ec3e34abe1ae925bfc03bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lukauskas/dgw
  tools:
  - DGW
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# dtw-warped-profile-visualization

## Summary

Reconstruct the DGW dgw-explorer aligned-cluster heatmap visualisation: launch the interactive explorer on a {prefix}_config.dgw run, cut the dendrogram into shape-based clusters, and render four-panel per-cluster views (prototype, average DTW projection, original and warped coverage heat-maps) with points-of-interest overlays and POI entropy validation.

## When to use

Use when epigenomic ChIP-Seq peak regions (e.g. ENCODE H3K4me3/H3K9ac K562) have already been DGW-aligned and clustered and you need to visually inspect DTW-warped heat-maps, cluster prototypes, and validate alignment against TSS/FSS points of interest.

## When NOT to use

- Raw BAM/ChIP-Seq data that has not yet been peak-called (MACS2), binned into 50 bp windows, and clustered — run the DGW worker first to produce the .dgw artefacts
- Data lacking points of interest — regions without any entry in fss.poi should be dropped with --ignore-no-poi-regions, so POI entropy validation is impossible without annotation
- Datasets where standard feature tables or pre-computed profiles suffice and shape-based alignment adds no value

## Inputs

- {prefix}_config.dgw DGW configuration file
- {prefix}_dataset.pd binned coverage dataset
- {prefix}_linkage.npy hierarchical clustering linkage matrix
- {prefix}_prototypes.pickle cluster prototypes (scaled prioritised shape averaging)
- {prefix}_warping_paths.pickle DTW warping paths
- Points-of-interest BED file (e.g. TSS and first splice site positions) passed via -poi

## Outputs

- dgw_explorer_cluster_heatmaps.png aligned heat-map figure (valid PNG)
- Per-cluster four-panel views: prototype, average DTW projection, original and warped coverage heat-maps
- POI position histograms pre- and post-warping per cluster
- Per-cluster BED files and saved prototype/heat-map data

## How to apply

Launch dgw-explorer with the worker-produced {prefix}_config.dgw configuration file, ensuring {prefix}_dataset.pd, {prefix}_linkage.npy, {prefix}_prototypes.pickle and {prefix}_warping_paths.pickle reside in the same directory. Render the dendrogram and peak heat-map window, then cut the dendrogram interactively at a level near the leaves (the paper reports a cut yielding 45 clusters as appropriate). For each cluster, display the four-panel cluster explorer: the cluster prototype computed via scaled prioritised shape averaging, the average DTW projection onto the prototype, and heat-maps of the original and warped per-dataset coverage. Overlay up to two sets of points of interest via the -poi parameter (e.g. first splice sites in black, transcription start sites in white). Generate POI position histograms pre- and post-warping per cluster, rescaling raw data to the maximum region length before histogramming, then save prototypes, heat-maps and per-cluster BED files and export the figure as dgw_explorer_cluster_heatmaps.png.

## Related tools

- **DGW** (Runs dgw-explorer, the interactive dendrogram/heat-map viewer used to cut clusters and render DTW-warped profiles) — https://github.com/lukauskas/dgw
- **Python** (DGW is a stand-alone Python package (requires Python 2.7) in which dgw-explorer and its matplotlib-based visualisations run) — https://github.com/lukauskas/dgw
- **matplotlib** (Rendering of dendrograms, four-panel cluster heat-maps, POI overlays and histogram figures)

## Examples

```
dgw-explorer {prefix}_config.dgw --poi fss.poi,tss.poi --ignore-no-poi-regions
```

## Evaluation signals

- dgw_explorer_cluster_heatmaps.png exists in the output directory and is a valid PNG
- Warped POI position histograms show reduced entropy relative to raw histograms (paper reports 12.91% TSS and 7.72% FSS entropy decrease in the selected cluster)
- Dendrogram cut level yields the expected cluster count (paper reports 45 clusters as appropriate)
- Each cluster's four-panel explorer shows a coherent prototype (scaled prioritised shape averaging output) with the warped heat-map more tightly aligned around POIs than the original heat-map
- All four DGW artefacts ({prefix}_dataset.pd, {prefix}_linkage.npy, {prefix}_prototypes.pickle, {prefix}_warping_paths.pickle) load correctly from the same directory as the config file

## Limitations

- dgw-explorer relies on the DTW dendrogram cut, which the paper notes is a difficult choice; results depend on the interactive cut level
- POI validation requires completely contained POI regions — only regions in poi_regions.bed where all points fall within the main region are matched
- DTW alignment was constrained by a Sakoe-Chiba band of width 12; warping beyond this bound is not representable in the warped heat-maps
- DGW requires Python 2.7 to run, and dependent packages (mlpy, fastcluster, dtwco) are archived/unmaintained

## Evidence

- [abstract] DGW uses Dynamic Time Warping to adaptively rescale and align genomic distances: "DGW uses Dynamic Time Warping to adaptively rescale and align genomic distances which allows to group regions of interest with similar shapes"
- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] entropy decrease after warping: "we observe a decrease of 12.91 % for TSS and 7.72 % for FSS location distributions in the selected cluster after warping"
- [methods] prototype via scaled prioritised shape averaging: "here we use the scaled prioritised shape averaging algorithm of [18]"
- [methods] complete-overlap POI matching: "find all the regions in `poi_regions.bed` that overlap *completely* (i.e. *all* points in the points of interest region are contained within the main region)"
- [readme] DGW explorer documentation: "This is the repository for the open source DGW library developped by Saulius Lukauskas. Please consult the documentation on https://dgw.readthedocs.org/"
