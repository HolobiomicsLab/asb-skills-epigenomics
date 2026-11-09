---
name: dtw-alignment-of-epigenomic-peak-shapes
description: Use when ChIP-seq enrichment measurements within peak regions (e.g. MACS2-called
  peaks) must be aligned and clustered by shape similarity in a completely unsupervised
  way, without transcription start sites or gene annotation.
when_to_use_negative:
- When gene annotation, TSS anchors, or supervised labels are available and preferred
  for alignment
- When peaks are shorter than 5 bins, longer than 1000 bins, or have fewer than 10
  counts — these are filtered out before DGW
- When standard hierarchical clustering on raw (unaligned) profiles suffices — it
  only succeeds for small noise/modifications (v<0.15, p<0.15)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0605
tools:
- name: DGW
  role: Core tool performing DTW-based simultaneous alignment, distance computation,
    and hierarchical clustering of epigenomic peak shapes
  repo: github.com/lukauskas/dgw
- name: MACS2
  role: Peak caller used upstream to identify enriched regions relative to an input
    control sample
- name: Python
  role: Runtime environment; DGW is a Python package requiring Python 2.7
  repo: github.com/lukauskas/dgw
- name: fastcluster
  role: Agglomerative hierarchical clustering backend (complete linkage) used by DGW
  repo: github.com/sauliusl/fastcluster
- name: mlpy
  role: Provides the constrained DTW implementation (Sakoe-Chiba band) used for pairwise
    distance computation
  repo: github.com/sauliusl/mlpy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:f078af98172dac2e5cf9c87da108b78206c342af0c68091b02cbd551350bdafc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - MACS2
  - Python
  - fastcluster
  - mlpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# dtw-alignment-of-epigenomic-peak-shapes

## Summary

Unsupervised simultaneous alignment and clustering of epigenomic ChIP-seq peak shapes using Dynamic Genome Warping (DGW), which applies Dynamic Time Warping to adaptively rescale and align genomic distances across multiple marks, grouping regions of interest with similar shapes without TSS or gene annotation anchors.

## When to use

Use when ChIP-seq enrichment measurements within peak regions (e.g. MACS2-called peaks) must be aligned and clustered by shape similarity in a completely unsupervised way, without transcription start sites or gene annotation.

## When NOT to use

- When gene annotation, TSS anchors, or supervised labels are available and preferred for alignment
- When peaks are shorter than 5 bins, longer than 1000 bins, or have fewer than 10 counts — these are filtered out before DGW
- When standard hierarchical clustering on raw (unaligned) profiles suffices — it only succeeds for small noise/modifications (v<0.15, p<0.15)

## Inputs

- Aligned-read BAM files (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam, wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam) with .bam.bai index files
- Input control BAM file (e.g. wgEncodeBroadHistoneK562ControlStdAlnRep1.bam)
- Filtered BED file of merged peak regions
- Optional points-of-interest BED (poi_regions.bed) for POI-based alignment

## Outputs

- dgw_linkage.npy (hierarchical clustering linkage matrix)
- Per-cluster BED files of aligned, shape-similar peak regions
- Pairwise DTW distance matrices (strand-aware, forward and reversed)
- Cluster prototypes via scaled prioritised shape averaging
- Dendrogram for cluster cutting

## How to apply

Peak-call each ChIP-seq mark (e.g. ENCODE K562 H3K4me3 and H3K9ac BAM files) against an input control with MACS2, then merge peak sets by union of regions called for either mark. Filter merged peaks to lengths larger than 5 and smaller than 1000 bins (50 bp non-overlapping windows), with at least 10 counts and at least one bin with more than 10 reads (-min-pileup). Run dgw-worker on the filtered BED file with the BAM datasets using bin size 50, the squared Euclidean metric, and a Sakoe-Chiba band of width 12; DGW extends single-end reads to estimated fragment lengths, bins coverage strand-aware, computes two DTW distances per pairwise peak comparison (one with sequences unchanged, one with one reversed), and performs agglomerative hierarchical clustering with complete linkage to avoid chaining. Cut the dendrogram at an appropriate level and compute prototypes via scaled prioritised shape averaging.

## Related tools

- **DGW** (Core tool performing DTW-based simultaneous alignment, distance computation, and hierarchical clustering of epigenomic peak shapes) — github.com/lukauskas/dgw
- **MACS2** (Peak caller used upstream to identify enriched regions relative to an input control sample)
- **Python** (Runtime environment; DGW is a Python package requiring Python 2.7) — github.com/lukauskas/dgw
- **fastcluster** (Agglomerative hierarchical clustering backend (complete linkage) used by DGW) — github.com/sauliusl/fastcluster
- **mlpy** (Provides the constrained DTW implementation (Sakoe-Chiba band) used for pairwise distance computation) — github.com/sauliusl/mlpy

## Examples

```
dgw-worker filtered_peaks.bed --output dgw_results --data wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam --nbins 50 --band-width 12 --metric sqeuclidean --linkage complete
```

## Evaluation signals

- dgw_linkage.npy and the per-cluster BED files exist after the run
- The pairwise-distance stage completes with runtime of the same order as the reported ~420 CPU-minutes across six cores (just over one hour wall-clock)
- DGW clustering recovers expected cluster structure where standard hierarchical clustering fails (simulation recovers 5 seed patterns)
- TSS/FSS location-distribution entropy decreases after warping (reported 12.91% for TSS, 7.72% for FSS in the selected cluster)
- Dendrogram can be cut to yield clusters whose members share similar peak shapes across the multiple marks

## Limitations

- Requires Python 2.7 to run
- Cutting the dendrogram at an appropriate level is a difficult choice
- Standard hierarchical clustering without DGW alignment only succeeds when noise and modifications are small (v<0.15, p<0.15)
- mlpy project is dead; the constrained DTW code survives in the archived dtwco package, which is no longer maintained and recommends tslearn as alternative
- Reported runtime is substantial: ~420 CPU-minutes across six cores for the ENCODE K562 two-mark run

## Evidence

- [abstract] DGW uses Dynamic Time Warping to adaptively rescale and align genomic distances which allows to group regions of interest with similar shapes: "DGW uses Dynamic Time Warping to adaptively rescale and align genomic distances which allows to group regions of interest with similar shapes"
- [methods] constrained the DTW with a Sakoe-Chiba Band of width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed: "we compute two distances for every pairwise peak comparison: one with the two sequences unchanged, and one with one of them reversed"
- [methods] perform agglomerative hierarchical clustering, using complete linkage to avoid chaining: "perform agglomerative hierarchical clustering, using complete linkage to avoid chaining"
- [methods] Single-end reads are extended to the estimated fragment lengths: "Single-end reads are extended to the estimated fragment lengths"
- [readme] This is the repository for the open source DGW library developped by Saulius Lukauskas.: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
