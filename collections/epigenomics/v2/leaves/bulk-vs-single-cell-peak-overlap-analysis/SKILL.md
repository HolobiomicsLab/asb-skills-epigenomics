---
name: bulk-vs-single-cell-peak-overlap-analysis
description: Use when a scATAC-seq atlas has per-cluster pseudo-bulk peak sets and
  a bulk ATAC-seq reference from the same tissue exists, and you must report the fraction
  of cell-type-specific CREs missed by bulk profiling.
when_to_use_negative:
- Input is a peak-based method output where each sample has its own unique peak reference
  — the article notes peak-based methods hinder multi-sample integrative analysis
  and bias against rare populations
- The single-cell dataset has not yet been clustered, since per-cluster aggregation
  is the prerequisite for pseudo-bulk peak calling
- No bulk ATAC-seq reference from the same tissue/region is available, making the
  novelty-fraction comparison impossible
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3306
tools:
- name: SnapATAC
  role: R package used to load the MOs atlas snap file, handle cluster assignments,
    and group reads from cell subsets (extractReads) before per-cluster peak calling
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python 2.7 module providing extractReads, runSnapAddPmat, runMACS and runMACSForAll
    for pseudo-bulk matrix building and MACS2-wrapped peak calling on the snap file
  repo: https://github.com/r3fang/SnapTools
- name: MACS2 (via runMACS)
  role: Peak caller invoked by runMACS with --nomodel --shift 100 --ext 200 --qval
    1e-2 -B --SPMR to call accessible peaks per aggregated cluster
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:d7904d53d3bb6d85840acca4838cdb0392f62dc7d54be3658354030f8c5cbfcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - MACS2 (via runMACS)
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapTools
  - extractReads
  - runMACS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Bulk vs single-cell peak overlap analysis

## Summary

Quantifies how many candidate cis-regulatory elements discovered by pooling single-cell scATAC-seq fragments per cell-type cluster are invisible to conventional bulk ATAC-seq peak calling. In the MOs atlas this showed ~370,000 candidate regulatory elements across 31 cell populations, with 56% undetectable in bulk ATAC-seq of the same region.

## When to use

Use when a scATAC-seq atlas has per-cluster pseudo-bulk peak sets and a bulk ATAC-seq reference from the same tissue exists, and you must report the fraction of cell-type-specific CREs missed by bulk profiling.

## When NOT to use

- Input is a peak-based method output where each sample has its own unique peak reference — the article notes peak-based methods hinder multi-sample integrative analysis and bias against rare populations
- The single-cell dataset has not yet been clustered, since per-cluster aggregation is the prerequisite for pseudo-bulk peak calling
- No bulk ATAC-seq reference from the same tissue/region is available, making the novelty-fraction comparison impossible

## Inputs

- Snap-format (.snap) single-nucleus ATAC-seq file for the mouse secondary motor cortex atlas
- Cluster assignments derived from the SnapATAC embedding (31 major cell populations)
- Per-barcode fragment (FM) session usable for read grouping
- Bulk ATAC-seq peak set (e.g. narrowPeak/BED) from the same brain region

## Outputs

- cre_peaks.bed — merged, deduplicated unique candidate cis-regulatory element set (~373,583 elements)
- Companion table of per-cluster peak counts
- Bulk-novelty fraction (percentage of CREs not detectable in bulk ATAC-seq; ~56%)

## How to apply

Load the MOs atlas snap file (55,592 mouse secondary motor cortex nuclei) with cluster assignments from the SnapATAC embedding in R (SnapATAC requires R >= 3.4.0 & < 3.6.0, since 3.6 breaks rhdf5). For each major cluster, aggregate per-barcode fragments into pseudo-bulk signal with SnapATAC read-grouping (extractReads) and snaptools snap-add-pmat via SnapTools (Python 2.7 recommended). Call accessible peaks per cluster with runMACS/runMACSForAll, which wraps MACS2 using '--nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR'. Merge the per-cluster peak sets and deduplicate to count the unique candidate CREs, then intersect the merged set against bulk ATAC-seq peaks from the same region to compute the percentage of elements not detectable in bulk data. Rationale: bulk-only references bias toward abundant cell types, so per-cluster aggregation surfaces regulatory elements of rare populations before overlap testing.

## Related tools

- **SnapATAC** (R package used to load the MOs atlas snap file, handle cluster assignments, and group reads from cell subsets (extractReads) before per-cluster peak calling) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python 2.7 module providing extractReads, runSnapAddPmat, runMACS and runMACSForAll for pseudo-bulk matrix building and MACS2-wrapped peak calling on the snap file) — https://github.com/r3fang/SnapTools
- **MACS2 (via runMACS)** (Peak caller invoked by runMACS with --nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR to call accessible peaks per aggregated cluster)

## Examples

```
runMACS --nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR --input pseudo_bulk_cluster1.fragments.bed
```

## Evaluation signals

- Merged, deduplicated unique peak set should contain approximately 373,583 candidate CREs (paper reports ~370,000 candidate regulatory elements across 31 cell populations)
- Bulk-novelty fraction should land near 56% of elements not detectable in bulk ATAC-seq of the same region
- Each of the 31 major clusters must have contributed a per-cluster peak call before merging; check per-cluster peak counts are all non-zero
- All called peaks pass the runMACS/MACS2 significance cutoff --qval 1e-2 (q-value <= 0.01)

## Limitations

- Functional validation of additional open chromatin regions identified from rare cell types is still needed; the authors state further experiments are required
- Iterative clustering strategies require multiple rounds of clustering, aggregation, and peak calling, hindering application to large-scale datasets
- In iterative clustering, crude clusters represent the most dominant cell types, so peaks in rare populations may still be underrepresented
- Peak-based methods hinder multi-sample integrative analysis where each sample has its own unique peak reference

## Evidence

- [abstract] The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [intro] we demonstrate the utility of SnapATAC by building a high-resolution single-cell atlas of the mouse secondary motor cortex: "we demonstrate the utility of SnapATAC by building a high-resolution single-cell atlas of the mouse secondary motor cortex."
- [readme] SnapTools module for working with snap files: [evidence span withheld — anchor status: external_doc]
- [readme] R version constraint for SnapATAC: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
- [readme] SnapATAC2 end-to-end pipeline includes peak calling and differential analysis: [evidence span withheld — anchor status: external_doc]
- [discussion] Iterative clustering limitation on rare populations: "the crude clusters represent the most dominant cell types in the tissues; therefore, peaks in the rare populations may still be underrepresented."
