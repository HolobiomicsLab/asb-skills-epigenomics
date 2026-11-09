---
name: candidate-cis-regulatory-element-merging
description: Use when you have scATAC-seq fragment data with cluster/barcode assignments
  and need a cell-type-resolved set of candidate regulatory elements that a single
  bulk peak call would miss.
when_to_use_negative:
- Your input is already a merged population-level peak reference — peak-based aggregation
  per cluster is unnecessary and the article warns peak-based approaches bias toward
  abundant cell types and underrepresent rare populations
- You need functional validation of novel open chromatin regions — the article states
  further experiments are still required; this skill only nominates candidates computationally
- Dataset scale exceeds what iterative clustering can handle — the article notes the
  strategy of repeated clustering, aggregation, and peak calling hinders application
  to large-scale datasets (consider SnapATAC2 instead)
tools:
- name: SnapTools
  role: Python module providing runMACS/runMACSForAll (MACS2 wrapper) and snap file
    handling (extractReads, runSnapAddPmat) used for per-cluster peak calling on grouped
    fragments
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC
  role: R package supplying the clustering context and read-grouping functions used
    to build per-cluster pseudo-bulk signals before peak calling
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:a6510759450a046232e791cb149168676a6949b4ed912ace476202655adf83ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapTools
  tools:
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapTools
  - extractReads
  - runMACS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# candidate-cis-regulatory-element-merging

## Summary

Aggregates per-cell-cluster fragments from a single-cell ATAC-seq atlas (e.g., the 55,592-nucleus mouse secondary motor cortex snap file) into pseudo-bulk signals, calls cluster-specific accessible peaks with MACS2, and merges/deduplicates them into a unified candidate cis-regulatory element set (~373,583 elements across 31 cell populations).

## When to use

Use when you have scATAC-seq fragment data with cluster/barcode assignments and need a cell-type-resolved set of candidate regulatory elements that a single bulk peak call would miss.

## When NOT to use

- Your input is already a merged population-level peak reference — peak-based aggregation per cluster is unnecessary and the article warns peak-based approaches bias toward abundant cell types and underrepresent rare populations
- You need functional validation of novel open chromatin regions — the article states further experiments are still required; this skill only nominates candidates computationally
- Dataset scale exceeds what iterative clustering can handle — the article notes the strategy of repeated clustering, aggregation, and peak calling hinders application to large-scale datasets (consider SnapATAC2 instead)

## Inputs

- Snap-format (hdf5) single-cell ATAC-seq file containing the FM fragment session (e.g., MOs atlas snap for 55,592 nuclei)
- Cluster assignments for each cell, derived from the SnapATAC embedding/Leiden clustering
- Bulk ATAC-seq peak BED file from the same tissue/region (for novelty intersection)

## Outputs

- cre_peaks.bed — merged, deduplicated candidate cis-regulatory element set (~373,583 peaks)
- Per-cluster peak-count table (peak counts for each of the 31 cell populations)
- Bulk-novelty fraction (percentage of merged elements not detected in bulk ATAC-seq; ~56%)

## How to apply

Load the MOs atlas snap file and cluster assignments in R (SnapATAC, R >= 3.4.0 & < 3.6.0). For each major cluster, aggregate fragments into a pseudo-bulk signal using SnapATAC read-grouping (extractReads / runSnapAddPmat via SnapTools, Python 2.7), exploiting the indexed FM (fragment) session of the snap file. Call accessible peaks per cluster with runMACS / runMACSForAll in SnapTools, which wraps MACS2 using '--nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR'. Merge the per-cluster peak calls and deduplicate to obtain the unique candidate cis-regulatory element set, then intersect it against bulk ATAC-seq peaks from the same region to quantify the fraction of elements undetectable in bulk data (paper: ~56%). Write the merged set to cre_peaks.bed with a companion table of per-cluster peak counts and the bulk-novelty fraction.

## Related tools

- **SnapTools** (Python module providing runMACS/runMACSForAll (MACS2 wrapper) and snap file handling (extractReads, runSnapAddPmat) used for per-cluster peak calling on grouped fragments) — https://github.com/r3fang/SnapTools
- **SnapATAC** (R package supplying the clustering context and read-grouping functions used to build per-cluster pseudo-bulk signals before peak calling) — https://github.com/r3fang/SnapATAC

## Examples

```
macs2 callpeak --nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR -t cluster1.pseudo.bulk.bed -n cluster1_peaks
```

## Evaluation signals

- The merged unique peak set contains approximately 373,583 elements (paper reports ~370,000 candidate regulatory elements across 31 cell populations)
- About 56% of merged elements are not detectable in bulk ATAC-seq peaks of the same brain region when intersected
- Per-cluster peak counts sum to more than the merged unique count, confirming deduplication of overlapping peaks across clusters
- Every merged peak traces back to at least one cluster's MACS2 call at q-value threshold 1e-2
- Output cre_peaks.bed is a valid BED file with genomic coordinates, and the companion table has one row per major cluster

## Limitations

- Peaks in rare cell populations may be underrepresented because crude clusters correspond to the most dominant cell types in iterative clustering strategies
- Functional validation of additional open chromatin regions identified from rare cell types is still needed; the authors state further experiments are required
- Iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, hindering application to large-scale datasets

## Evidence

- [abstract] The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [other] Call accessible peaks per cluster with runMACS / runMACSForAll in SnapTools wrapping MACS2: [evidence span withheld — anchor status: not_found]
- [readme] FM session contains all usable fragments for each cell: [evidence span withheld — anchor status: external_doc]
- [discussion] Iterative clustering requires multiple rounds of aggregation and peak calling: "the strategy of iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, thus hindering its application to large-scale datasets"
