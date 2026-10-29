---
name: chip-seq-coverage-binning
description: Use when raw ChIP-seq BAM files for one or more epigenomic marks (e.g.
  ENCODE K562 H3K4me3/H3K9ac) plus a MACS/MACS2 peak list must be turned into fixed-width
  binned coverage profiles for shape-based alignment or clustering.
when_to_use_negative:
- The coverage data are already binned into fixed windows or represented as a feature
  table, so re-binning is redundant.
- The analysis requires absolute-position or gene-anchored comparison (e.g. explicitly
  anchored on transcription start sites or gene annotation) rather than unsupervised
  shape alignment.
- Peak regions are shorter than 5 or longer than 1000 bins, or contain no bin with
  more than 10 reads, so they fall outside the analysed set by design.
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: DGW
  role: Extends single-end reads to estimated fragment lengths, bins coverage into
    non-overlapping 50 bp windows per mark, and exposes the binned vectors for DTW
    alignment and clustering
  repo: github.com/lukauskas/dgw
- name: MACS2
  role: Defines the peak regions against input control that become the units of coverage
    binning
- name: pandas
  role: Used with numpy to read the processed binned coverage dataset, representing
    each peak as a sequence of multidimensional vectors
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:dbc2372e4fef4a02148458346851669ee084406329ce50b683e4cb306588c018
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - MACS2
  - Python
  - UCSC genome browser
  - mlpy
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# chip-seq-coverage-binning

## Summary

Converts aligned-read ChIP-seq BAM signal for multiple epigenomic marks into fixed-length, per-peak coverage profiles by extending single-end reads to estimated fragment lengths and binning coverage into non-overlapping 50 bp windows, so each peak becomes a sequence of multidimensional vectors suitable for DTW-based shape comparison in DGW.

## When to use

Use when raw ChIP-seq BAM files for one or more epigenomic marks (e.g. ENCODE K562 H3K4me3/H3K9ac) plus a MACS/MACS2 peak list must be turned into fixed-width binned coverage profiles for shape-based alignment or clustering.

## When NOT to use

- The coverage data are already binned into fixed windows or represented as a feature table, so re-binning is redundant.
- The analysis requires absolute-position or gene-anchored comparison (e.g. explicitly anchored on transcription start sites or gene annotation) rather than unsupervised shape alignment.
- Peak regions are shorter than 5 or longer than 1000 bins, or contain no bin with more than 10 reads, so they fall outside the analysed set by design.

## Inputs

- Aligned-read BAM files with .bai indices (e.g. wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam, wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam and their control)
- Filtered peak-region BED file (MACS/MACS2-called regions, merged by union across marks)
- Bin size parameter (50 bp, non-overlapping windows)
- Coverage counts per bin per epigenomic mark (computed by DGW from the BAM files)

## Outputs

- Binned coverage dataset: per-peak sequences of multidimensional vectors (one coverage vector per 50 bp bin per epigenomic mark)
- Filtered peak set with per-peak bin counts (peaks between 5 and 1000 bins, at least 10 counts)
- Inputs for downstream dgw-worker pairwise DTW distances, dendrogram (dgw_linkage.npy) and per-cluster BED files

## How to apply

For each epigenomic mark, extend single-end reads to the estimated fragment lengths (allowing DGW to extend from the alignment), then compute read coverage within each MACS2-called peak region using non-overlapping 50 bp bins, producing one coverage vector per bin per mark and thus a multidimensional vector sequence per peak. Restrict the analysed peaks to those with length larger than 5 and smaller than 1000 bins, and drop peaks that have no bin with more than 10 reads (--min-pileup constraint) or fewer than 10 counts. Feed the resulting binned coverage dataset (loaded with pandas/numpy) to DGW's DTW distance computation, which uses the squared Euclidean metric on these vectors, a Sakoe-Chiba band of width 12, and forward/reversed strand comparisons. The rationale is that binning at a fixed window size makes peaks comparable by shape regardless of absolute length, enabling unsupervised alignment and clustering without anchors such as TSSs or gene annotation.

## Related tools

- **DGW** (Extends single-end reads to estimated fragment lengths, bins coverage into non-overlapping 50 bp windows per mark, and exposes the binned vectors for DTW alignment and clustering) — github.com/lukauskas/dgw
- **MACS2** (Defines the peak regions against input control that become the units of coverage binning)
- **pandas** (Used with numpy to read the processed binned coverage dataset, representing each peak as a sequence of multidimensional vectors)

## Examples

```
dgw-worker --bin-size 50 filtered_peaks.bed wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam
```

## Evaluation signals

- Each peak is represented as a sequence with one coverage vector per 50 bp non-overlapping bin for every epigenomic mark in the dataset.
- Retained peaks satisfy the length filter (larger than 5 and smaller than 1000 bins) and the count filter (at least 10 counts; every kept peak has a bin with more than 10 reads).
- The binned dataset loads correctly with pandas/numpy and yields vector sequences consumable by the DTW distance stage.
- Running the downstream dgw-worker stage produces dgw_linkage.npy and per-cluster BED files, confirming the binned inputs were valid.
- Binned profiles show the expected bimodal/single-peak structures around landmarks (e.g. TSS and first splicing site), as visualised per profile in the article.

## Limitations

- Peaks with no bin exceeding 10 reads, fewer than 10 counts, or lengths outside 5-1000 bins are silently excluded from the analysis.
- Single-end reads must be extended to estimated fragment lengths; binning raw read positions without extension changes the coverage signal.
- DGW requires Python 2.7 to run, which constrains reproducibility in modern environments.

## Evidence

- [methods] Non-overlapping 50 bp binning of peak coverage: "coverage within peak regions are binned into non-overlapping windows spanning 50 bp"
- [methods] Read extension before binning: "Single-end reads are extended to the estimated fragment lengths"
- [methods] Peak length and count filters bounding the binned set: "restricted the analysed set of peaks to those that had a length larger than 5 and smaller than 1000 bins"
- [methods] Minimum-pileup filter for ignoring unexpressed regions: "the unexpressed regions that have no bin with more than 10 reads in it (`-min-pileup` constraint to change) will be ignored"
