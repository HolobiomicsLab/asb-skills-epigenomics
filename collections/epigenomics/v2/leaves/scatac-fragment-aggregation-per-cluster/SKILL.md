---
name: scatac-fragment-aggregation-per-cluster
description: Use when you already have single-cell cluster assignments for scATAC-seq
  data (e.g., from a SnapATAC snap file with barcodes and the FM fragment session)
  and need to generate pseudo-bulk signal per cluster for peak calling.
when_to_use_negative:
- Cluster labels are not yet available or unreliable — the article warns that peaks
  in rare populations may be underrepresented when crude clusters reflect only dominant
  cell types
- You need a population-level reference independent of iterative clustering; iterative
  clustering requires multiple rounds of clustering, aggregation, and peak calling,
  hindering large-scale datasets
- Input is already a processed peak or feature matrix (cell-by-peak matrix in the
  PM session) rather than raw fragments needing aggregation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SnapATAC
  role: R package that groups reads from subsets of cells (cluster-based read grouping)
    to build pseudo-bulk profiles from the snap file's fragment session
  repo: https://github.com/r3fang/SnapATAC
- name: SnapTools
  role: Python module providing extractReads and runMACS / runMACSForAll wrappers
    around MACS2 for per-cluster fragment extraction and peak calling
  repo: github.com/r3fang/SnapTools
- name: SnapATAC2
  role: Successor package (Python/Rust) whose end-to-end pipeline also covers peak
    calling from aggregated single-cell epigenomic data
  repo: scverse/SnapATAC2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:43ea563291220571e9fc24ae4c9ea6678294ec0374664571d45e6b316a78b413
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/r3fang/SnapATAC
  tools:
  - Python 2.7
  - R 3.4.0
  - SnapATAC
  - SnapATAC2
  - SnapTools
  - extractReads
  - runMACS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scatac-fragment-aggregation-per-cluster

## Summary

Aggregates single-nucleus ATAC-seq fragments from cells assigned to each cluster into pseudo-bulk accessibility profiles, which are then used for per-cluster MACS2 peak calling to build a cell-type-specific atlas of candidate cis-regulatory elements. This is how SnapATAC-derived clusters (e.g., the 55,592-nucleus mouse secondary motor cortex atlas) are converted into discrete regulatory element sets.

## When to use

Use when you already have single-cell cluster assignments for scATAC-seq data (e.g., from a SnapATAC snap file with barcodes and the FM fragment session) and need to generate pseudo-bulk signal per cluster for peak calling.

## When NOT to use

- Cluster labels are not yet available or unreliable — the article warns that peaks in rare populations may be underrepresented when crude clusters reflect only dominant cell types
- You need a population-level reference independent of iterative clustering; iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, hindering large-scale datasets
- Input is already a processed peak or feature matrix (cell-by-peak matrix in the PM session) rather than raw fragments needing aggregation

## Inputs

- Snap file (.snap, HDF5) containing the FM fragment session and BD barcode session
- Cluster assignment table mapping cell barcodes to cluster IDs (from SnapATAC embedding/clustering)
- Reference genome (e.g., mm10) for MACS2 peak calling

## Outputs

- Per-cluster pseudo-bulk fragment/signal tracks
- Per-cluster MACS2 peak sets (BED, with --SPMR pileup)
- Merged, deduplicated unique candidate CRE set (cre_peaks.bed; 373,583 peaks for the MOs atlas)
- Companion table of per-cluster peak counts and bulk-novelty fraction (~56%)

## How to apply

Load the snap file (a hierarchically structured HDF5 file with HD, AM, PM, GM, BD, and FM sessions) together with the cluster labels derived from the SnapATAC embedding, in R (>= 3.4.0 & < 3.6.0, since R 3.6 breaks the rhdf5 package). For each major cluster, use SnapATAC read-grouping functions backed by SnapTools (extractReads / snapAddPmat, Python 2.7 recommended) to pool fragments of all member cells into a single pseudo-bulk signal per cluster. Call accessible peaks per cluster with runMACS / runMACSForAll in SnapTools, which wraps MACS2 with the settings '--nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR'. Merge per-cluster peak calls across clusters and deduplicate to obtain the unique candidate CRE set; for the mouse secondary motor cortex atlas this yields 373,583 unique candidate cis-regulatory elements across 31 cell populations, ~56% of which are undetectable in bulk ATAC-seq of the same region.

## Related tools

- **SnapATAC** (R package that groups reads from subsets of cells (cluster-based read grouping) to build pseudo-bulk profiles from the snap file's fragment session) — https://github.com/r3fang/SnapATAC
- **SnapTools** (Python module providing extractReads and runMACS / runMACSForAll wrappers around MACS2 for per-cluster fragment extraction and peak calling) — github.com/r3fang/SnapTools
- **SnapATAC2** (Successor package (Python/Rust) whose end-to-end pipeline also covers peak calling from aggregated single-cell epigenomic data) — scverse/SnapATAC2

## Examples

```
snaptools snap-add-pmat --snap-file=MOs.snap --peak-file=cre_peaks.bed --verbose=True
```

## Evaluation signals

- Merged unique peak set contains approximately 373,583 elements (paper reports ~370,000 candidate regulatory elements across 31 cell populations)
- About 56% of merged candidate CREs are not detectable in bulk ATAC-seq peaks from the same region (mouse secondary motor cortex)
- MACS2 is invoked with the expected flags: --nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR
- cre_peaks.bed is produced together with a companion per-cluster peak-count table and bulk-novelty fraction
- Every barcode in each pseudo-bulk group belongs to exactly one cluster and fragment counts are consistent with the FM session contents

## Limitations

- Iterative clustering with repeated aggregation and peak calling hinders application to large-scale datasets
- Peaks in rare cell populations may be underrepresented because crude clusters represent the most dominant cell types
- Functional validation of additional open chromatin regions identified from rare cell types is still needed (further experiments required)
- Tooling is version-sensitive: R >= 3.4.0 & < 3.6.0 is required (R 3.6 breaks rhdf5), and Python 2.7 is highly recommended for SnapTools

## Evidence

- [other] For each major cluster, aggregate fragments into a pseudo-bulk signal using the SnapATAC read-grouping functions: "How to group reads from any subset of cells?"
- [other] MACS2 settings for per-cluster peak calling: [evidence span withheld — anchor status: not_found]
- [abstract] ~370,000 candidate regulatory elements across 31 cell populations: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [readme] snap file structure storing fragments: [evidence span withheld — anchor status: external_doc]
- [readme] Version constraint on R for SnapATAC: "R (>= 3.4.0 & < 3.6.0) (SnapATAC) (3.6 does not work for rhdf5 package);"
