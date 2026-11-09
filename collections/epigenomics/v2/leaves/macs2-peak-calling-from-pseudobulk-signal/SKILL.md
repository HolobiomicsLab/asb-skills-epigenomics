---
name: macs2-peak-calling-from-pseudobulk-signal
description: Use when you have cluster (or cell-type) assignments from single-cell
  ATAC-seq fragments and need a genome-wide set of candidate accessible peaks that
  captures cell-type-specific accessibility beyond what bulk peak calls detect.
when_to_use_negative:
- You only have a population-level (bulk) ATAC-seq sample with no cell clusters —
  iterative clustering plus per-cluster aggregation is unnecessary and rare-population
  peaks cannot be targeted.
- You have no per-cell fragment (FM) data — e.g. input is already a precomputed cell-by-peak
  matrix without underlying fragments, so pseudobulk signal cannot be built.
- Your goal is peak calling for a single, uniform sample rather than resolving cell-type-specific
  accessibility across heterogeneous tissue.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: SnapTools
  role: Python module that stores/queries the snap file and wraps MACS2 for per-cluster
    peak calling (runMACS / runMACSForAll) and read grouping (extractReads)
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC
  role: R package providing the clustering that defines the read groups (clusters)
    whose fragments are aggregated for peak calling
  repo: https://github.com/r3fang/SnapATAC
- name: MACS2
  role: Peak caller invoked via runMACS with --nomodel --shift 100 --ext 200 --qval
    1e-2 -B --SPMR on per-cluster pseudobulk signal
- name: extractReads
  role: SnapTools function to group reads from subsets of cells, producing the pseudobulk
    fragment signal for each cluster
  repo: https://github.com/r3fang/SnapTools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:c6e779681b8e2a5ac189e991ba0784f70817c40a10f6c42d03d89e020e4ccb6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapTools
  tools:
  - MACS2
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapTools
  - extractReads
  - runMACS
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# MACS2 peak calling from pseudobulk signal

## Summary

Call accessible peaks by aggregating ATAC-seq fragments from each cell cluster into pseudobulk signal, then running MACS2 per cluster and merging/deduplicating the per-cluster peak sets into a single candidate cis-regulatory element (CRE) catalog.

## When to use

Use when you have cluster (or cell-type) assignments from single-cell ATAC-seq fragments and need a genome-wide set of candidate accessible peaks that captures cell-type-specific accessibility beyond what bulk peak calls detect.

## When NOT to use

- You only have a population-level (bulk) ATAC-seq sample with no cell clusters — iterative clustering plus per-cluster aggregation is unnecessary and rare-population peaks cannot be targeted.
- You have no per-cell fragment (FM) data — e.g. input is already a precomputed cell-by-peak matrix without underlying fragments, so pseudobulk signal cannot be built.
- Your goal is peak calling for a single, uniform sample rather than resolving cell-type-specific accessibility across heterogeneous tissue.

## Inputs

- Snap (.hdf5) file containing per-barcode fragments (FM session) and the cell-by-peak matrix, e.g. the MOs atlas snap file
- Cluster assignments for each nucleus derived from the SnapATAC embedding
- Bulk ATAC-seq peak calls from the same tissue/region (for the novelty intersection)

## Outputs

- cre_peaks.bed: merged, deduplicated set of unique candidate cis-regulatory elements
- Companion table of per-cluster peak counts and the bulk-novelty fraction
- Per-cluster MACS2 peak calls (with -B/SPMR signal tracks)

## How to apply

For each major cluster in the SnapATAC atlas (e.g. the 55,592-nucleus mouse secondary motor cortex atlas), aggregate fragments into a pseudobulk signal using SnapTools read-grouping (extractReads / runSnapAddPmat). Call peaks per cluster by wrapping MACS2 with runMACS / runMACSForAll using the flags '--nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR'. Merge the per-cluster MACS2 peak calls and deduplicate to obtain the unique candidate CRE set, then intersect the merged peaks against bulk ATAC-seq peaks from the same region to compute the fraction of cluster-derived elements not detectable in bulk. In the MOs atlas this yielded 373,583 unique candidate CREs across 31 cell populations, ~56% of which were not detectable in bulk ATAC-seq. Rationale: aggregating reads per cluster compensates for the sparsity of single-cell profiles (only 5–15% of peaks detected per cell), and rare-population-specific elements are recovered because pooling is done within clusters rather than across the whole sample.

## Related tools

- **SnapTools** (Python module that stores/queries the snap file and wraps MACS2 for per-cluster peak calling (runMACS / runMACSForAll) and read grouping (extractReads)) — https://github.com/r3fang/SnapTools
- **SnapATAC** (R package providing the clustering that defines the read groups (clusters) whose fragments are aggregated for peak calling) — https://github.com/r3fang/SnapATAC
- **MACS2** (Peak caller invoked via runMACS with --nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR on per-cluster pseudobulk signal)
- **extractReads** (SnapTools function to group reads from subsets of cells, producing the pseudobulk fragment signal for each cluster) — https://github.com/r3fang/SnapTools

## Examples

```
runMACSForAll --snap-file mos_atlas.snap --cluster cluster_assignment.txt --outdir ./peaks --nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR
```

## Evaluation signals

- The merged, deduplicated unique peak set should contain approximately 373,583 elements (paper reports ~370,000 candidate regulatory elements across 31 cell populations).
- Roughly 56% of merged CREs should not overlap bulk ATAC-seq peaks from the same brain region — this bulk-novelty fraction is the validation metric.
- Each per-cluster MACS2 run should report significant peaks at q-value < 1e-2 (i.e. no peaks below the qval cutoff should be retained).
- The combined per-cluster peak counts should exceed the merged unique count, confirming that deduplication removed overlapping calls across clusters.
- The snap file QC report (usable fragments per barcode) should show sufficient coverage per cluster so pseudobulk signal supports reliable MACS2 calling.

## Limitations

- Iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, which hinders application to large-scale datasets.
- In iterative clustering the crude clusters represent the most dominant cell types, so peaks in rare populations may still be underrepresented.
- Peak-based methods hinder multi-sample integrative analysis where each sample has its own unique peak reference.
- Further experiments are needed to thoroughly validate the function of additional open chromatin regions identified from rare cell types.
- The older SnapATAC/SnapTools stack requires Python 2.7 (<3.0) and R >= 3.4.0 & < 3.6.0 (R 3.6 breaks the rhdf5 package).

## Evidence

- [methods] MACS2 settings on per-cluster pseudobulk: [evidence span withheld — anchor status: not_found]
- [intro] Sparse single-cell signal motivates aggregation: "The growing volume of scATAC-seq datasets coupled with the sparsity of signals in each individual profile due to low detection efficiency (5–15% of peaks detected per cell)7 present a unique"
- [abstract] ~370,000 candidate regulatory elements across 31 cell populations: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [discussion] Iterative clustering scale limitation: "the strategy of iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, thus hindering its application to large-scale datasets."
- [readme] SnapTools is the python module for snap files (pseudobulk source data): "SnapTools - a python module for pre-processing and working with [snap](https://github.com/r3fang/SnapATAC/wiki/FAQs) file."
- [readme] Snap file stores per-cell fragments needed for read grouping: [evidence span withheld — anchor status: external_doc]
