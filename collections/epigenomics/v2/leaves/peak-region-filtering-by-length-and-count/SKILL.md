---
name: peak-region-filtering-by-length-and-count
description: Use when MACS2-called merged peak regions from ChIP-seq marks (e.g. ENCODE
  K562 H3K4me3 + H3K9ac BAMs) must be filtered by region length in bins and minimum
  counts/pileup prior to DGW DTW clustering.
when_to_use_negative:
- Input is already a filtered peak set with length and count constraints applied (re-filtering
  could bias the region set)
- Regions are anchored to genomic features such as transcription start sites or gene
  annotation rather than peak-finder output — DGW's unsupervised setting expects peak
  regions from a peak finder such as MACS
- Peak lengths are naturally shorter than 5 bins or the marks have very low pileup
  everywhere, since the filters would remove nearly all regions
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: MACS2
  role: Calls enriched regions relative to the input control before the union merge
    and length/count filtering
- name: DGW
  role: Consumes the filtered BED regions; its --min-pileup constraint drops regions
    with no bin with more than 10 reads
  repo: github.com/lukauskas/dgw
- name: Python
  role: Language in which the filtering and the DGW pipeline are implemented (DGW
    needs Python 2.7)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1306-0
    title: 'DGW: an exploratory data analysis tool for clustering and visualisation
      of epigenomic marks'
  source_content_hash: sha256:fe4dd8ce2327375b2a800f373179a5ae8ea20a69558a531bcf4f2468df1075cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DGW
  - MACS2
  - Python
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# peak-region-filtering-by-length-and-count

## Summary

Filter MACS2-called merged ChIP-seq peak regions by length (larger than 5 and smaller than 1000 bins) and by count (at least 10 counts), plus a --min-pileup constraint requiring at least one bin with more than 10 reads, before DTW-based alignment and clustering with DGW.

## When to use

Use when MACS2-called merged peak regions from ChIP-seq marks (e.g. ENCODE K562 H3K4me3 + H3K9ac BAMs) must be filtered by region length in bins and minimum counts/pileup prior to DGW DTW clustering.

## When NOT to use

- Input is already a filtered peak set with length and count constraints applied (re-filtering could bias the region set)
- Regions are anchored to genomic features such as transcription start sites or gene annotation rather than peak-finder output — DGW's unsupervised setting expects peak regions from a peak finder such as MACS
- Peak lengths are naturally shorter than 5 bins or the marks have very low pileup everywhere, since the filters would remove nearly all regions

## Inputs

- Merged BED file of MACS2-called peak regions (union of regions called for either epigenomic mark)
- Binned coverage at 50 bp non-overlapping windows derived from BAM files such as wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam and wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam

## Outputs

- Filtered BED file retaining only peaks longer than 5 and shorter than 1000 bins with at least 10 counts
- Set of candidate regions passed to dgw-worker for binning, DTW distance computation, and agglomerative hierarchical clustering

## How to apply

After calling peaks with MACS2 against the input control (e.g. wgEncodeBroadHistoneK562ControlStdAlnRep1.bam) and merging the two mark peak sets by union of regions called for either mark, restrict the analysed set of peaks to those with a length larger than 5 and smaller than 1000 bins (at the 50 bp non-overlapping bin size DGW uses). Filter out peaks with fewer than 10 counts. Additionally drop unexpressed regions that have no bin with more than 10 reads (the --min-pileup constraint). The rationale is that extremely short regions carry too few bins to warp meaningfully, extremely long ones distort DTW cost, and low-coverage regions carry no usable epigenomic signal for shape-based clustering.

## Related tools

- **MACS2** (Calls enriched regions relative to the input control before the union merge and length/count filtering)
- **DGW** (Consumes the filtered BED regions; its --min-pileup constraint drops regions with no bin with more than 10 reads) — github.com/lukauskas/dgw
- **Python** (Language in which the filtering and the DGW pipeline are implemented (DGW needs Python 2.7))

## Examples

```
dgw-worker filtered_peaks.bed wgEncodeBroadHistoneK562H3k4me3StdAlnRep1.bam wgEncodeBroadHistoneK562H3k9acStdAlnRep1.bam --min-pileup 10 -b 50
```

## Evaluation signals

- No retained peak has a length outside the (5, 1000) bin interval at the 50 bp bin size
- Every retained peak has at least 10 counts
- No retained region lacks a bin with more than 10 reads (min-pileup satisfied)
- Filtered BED covers the union of regions called for either H3K4me3 or H3K9ac, not only their intersection
- The downstream dgw-worker run completes the pairwise-distance stage and produces dgw_linkage.npy and per-cluster BED files

## Limitations

- The thresholds (5–1000 bins, 10 counts, 10 reads per bin) were tuned for 50 bp non-overlapping windows on ENCODE K562 H3K4me3/H3K9ac data and may need adjustment for other bin sizes, marks, or cell types
- Cutting the dendrogram downstream 'is a difficult choice', so filtering does not guarantee biologically meaningful clusters
- DGW requires Python 2.7 and the lukauskas/dtwco DTW repository is no longer maintained (read only as of 2020-10-06)

## Evidence

- [methods] Peak length filter: "restricted the analysed set of peaks to those that had a length larger than 5 and smaller than 1000 bins"
- [methods] Minimum count filter: "Also we filtered out peaks with less than 10 counts."
- [methods] Min-pileup constraint: "the unexpressed regions that have no bin with more than 10 reads in it (`-min-pileup` constraint to change) will be ignored"
- [methods] Binning and merging context: "we then merged the two sets by considering every region called for either mark"
- [readme] DGW repository: "This is the repository for the open source DGW library developped by Saulius Lukauskas."
