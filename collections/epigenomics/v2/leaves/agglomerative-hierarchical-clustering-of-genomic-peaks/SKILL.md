---
name: agglomerative-hierarchical-clustering-of-genomic-peaks
description: Use when you need to group peak regions from one or more ChIP-seq marks
  by signal-shape similarity without anchors such as TSSs or gene annotation, using
  DTW distances between binned coverage profiles.
when_to_use_negative:
- You have gene/TSS annotations available and prefer annotation-anchored alignment
  rather than unsupervised shape-based clustering
- Peaks are shorter than 5 bins or longer than 1000 bins (50 bp bins) or lack any
  bin with more than 10 reads — these are excluded by the DGW filters
- 'Standard Euclidean-profile hierarchical clustering suffices: the article shows
  standard hierarchical clustering only succeeds when added noise and modifications
  are small (v<0.15, p<0.15), so evaluate whether DTW is needed'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3168
tools:
- name: DGW
  role: Runs fragment extension, 50 bp binning, strand-aware constrained DTW pairwise
    distances, and complete-linkage agglomerative clustering via the dgw-worker command
  repo: github.com/lukauskas/dgw
- name: MACS2
  role: Peak-calls each ChIP-seq mark against the input control to produce the enriched
    regions that DGW clusters
- name: fastcluster
  role: Provides the fast agglomerative hierarchical clustering backend DGW requires,
    installed from the author's fork
  repo: http://github.com/sauliusl/fastcluster
- name: mlpy
  role: Supplies the underlying DTW implementation used by DGW for constrained pairwise
    distances
  repo: git://github.com/sauliusl/mlpy.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:6c7f195014c451a2c79e56285a3fa6efe5693f7170c4fb1a9762caf7d5118af0
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
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# agglomerative-hierarchical-clustering-of-genomic-peaks

## Summary

Clusters epigenomic ChIP-seq peak regions by shape using agglomerative hierarchical clustering with complete linkage over strand-aware Dynamic Time Warping distances, as implemented in the DGW package for ENCODE K562 histone mark data (H3K4me3, H3K9ac).

## When to use

Use when you need to group peak regions from one or more ChIP-seq marks by signal-shape similarity without anchors such as TSSs or gene annotation, using DTW distances between binned coverage profiles.

## When NOT to use

- You have gene/TSS annotations available and prefer annotation-anchored alignment rather than unsupervised shape-based clustering
- Peaks are shorter than 5 bins or longer than 1000 bins (50 bp bins) or lack any bin with more than 10 reads — these are excluded by the DGW filters
- Standard Euclidean-profile hierarchical clustering suffices: the article shows standard hierarchical clustering only succeeds when added noise and modifications are small (v<0.15, p<0.15), so evaluate whether DTW is needed

## Inputs

- MACS2-called peak BED file, merged across marks and filtered to 5–1000 bins (50 bp bins) and ≥10 counts
- Aligned-read BAM files for each epigenomic mark (e.g. ENCODE K562 H3K4me3 and H3K9ac Std AlnRep1) with .bam.bai index files
- Input control BAM (e.g. wgEncodeBroadHistoneK562ControlStdAlnRep1.bam) for peak calling

## Outputs

- dgw_linkage.npy hierarchical clustering linkage matrix
- Per-cluster BED files of peak regions
- Dendrogram and per-cluster prototypes computed by scaled prioritised shape averaging

## How to apply

Peak-call each mark against an input control with MACS2, then merge the two peak sets by union of regions called for either mark; filter merged peaks to lengths larger than 5 and smaller than 1000 bins (50 bp bins) and with at least 10 counts, dropping regions with no bin exceeding 10 reads. Run dgw-worker on the filtered BED file with the mark BAM files (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam and wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam) using 50 bp non-overlapping bins, the squared Euclidean metric, and a Sakoe-Chiba band of width 12; DGW extends single-end reads to estimated fragment lengths and computes two DTW distances per peak pair (forward and reversed) before agglomerative hierarchical clustering. Use complete linkage to avoid chaining, then cut the dendrogram to define clusters and compute prototypes with scaled prioritised shape averaging. DGW requires Python 2.7 plus the author-maintained mlpy and fastcluster forks.

## Related tools

- **DGW** (Runs fragment extension, 50 bp binning, strand-aware constrained DTW pairwise distances, and complete-linkage agglomerative clustering via the dgw-worker command) — github.com/lukauskas/dgw
- **MACS2** (Peak-calls each ChIP-seq mark against the input control to produce the enriched regions that DGW clusters)
- **fastcluster** (Provides the fast agglomerative hierarchical clustering backend DGW requires, installed from the author's fork) — http://github.com/sauliusl/fastcluster
- **mlpy** (Supplies the underlying DTW implementation used by DGW for constrained pairwise distances) — git://github.com/sauliusl/mlpy.git

## Evaluation signals

- dgw_linkage.npy and the per-cluster BED files exist after the run
- The strand-aware DTW pairwise-distance stage completes without error
- Runtime is of the same order as the reported ~420 CPU-minutes across six cores (just over one hour wall-clock)
- In the simulation benchmark, clustering recovers 5 seed patterns with ~100 members each, whereas standard hierarchical clustering only succeeds for small noise/modifications (v<0.15, p<0.15)
- Warped clusters show reduced positional entropy of genomic landmarks (12.91% decrease for TSS, 7.72% for FSS in the selected cluster)

## Limitations

- Cutting the dendrogram at an appropriate level is a difficult choice with no automatic rule
- DGW requires Python 2.7 and author-maintained forks of mlpy and fastcluster; the repositories are archived/read-only, and dtwco recommends tslearn as a successor
- Complete linkage must be used explicitly to avoid chaining; pairwise DTW is strand-aware, requiring two distance computations per peak pair, which drives the high CPU cost

## Evidence

- [methods] Complete-linkage clustering avoids chaining and is what DGW uses on DTW distances: "perform agglomerative hierarchical clustering, using complete linkage to avoid chaining"
- [methods] Peak-length filter defining the clusterable region set: "restricted the analysed set of peaks to those that had a length larger than 5 and smaller than 1000 bins"
- [methods] DTW-constrained distance with Sakoe-Chiba band width 12: "constrained the DTW with a Sakoe-Chiba Band of width 12"
- [methods] Standard clustering fails unless noise/modifications are small, motivating the DTW-based approach: "Standard Hierarchical clustering is able to correctly group the simulated peaks according to the pattern they are originally derived from only if the added noise and modifications are small"
- [readme] DGW is the package providing this clustering workflow and its documentation: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
