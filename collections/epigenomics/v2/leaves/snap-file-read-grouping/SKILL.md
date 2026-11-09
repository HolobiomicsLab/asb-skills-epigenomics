---
name: snap-file-read-grouping
description: Use when scATAC-seq fragments are stored in a snap (.snap) file with
  cluster assignments already computed, and you need to pool fragments from a subset
  of cells (e.g. a major cluster) into a pseudo-bulk signal for downstream peak calling.
when_to_use_negative:
- Input is still raw FASTQ/BAM — a snap file must first be created with 'snaptools
  snap-pre' (alignment, MAPQ >= 30, fragment length <= 1000 bp filtering) before read
  grouping is possible
- Only one bulk sample with no cell-level clustering exists — there are no cell subsets
  to group and standard bulk peak calling applies directly
- Working in the SnapATAC2 (scverse) Python API — its preprocessing and peak-calling
  API differs and does not use the SnapTools extractReads/runSnapAddPmat functions
  described here
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: SnapTools
  role: Python module whose indexed FM fragment session and grouping utilities (extractReads
    / runSnapAddPmat, runMACS wrappers) perform the per-cluster fragment aggregation
  repo: https://github.com/r3fang/SnapTools
- name: SnapATAC
  role: R package that loads the snap file, supplies cluster assignments per barcode,
    and drives the read-grouping workflow prior to MACS2 peak calling
  repo: https://github.com/r3fang/SnapATAC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21583-9
    title: Comprehensive analysis of single cell ATAC-seq data with SnapATAC
  source_content_hash: sha256:59bf4018f35fd9dfd477894ec75d142cd292ec26e1fba87c0e012485f6ca728d
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
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# snap-file-read-grouping

## Summary

Aggregates single-nucleus ATAC-seq fragments from a snap file into cluster-level pseudo-bulk read groups so that per-cell-type chromatin accessibility can be peak-called with MACS2. This is the SnapTools/SnapATAC read-grouping step that converts sparse per-cell fragment data into bulk-like signal per cluster.

## When to use

Use when scATAC-seq fragments are stored in a snap (.snap) file with cluster assignments already computed, and you need to pool fragments from a subset of cells (e.g. a major cluster) into a pseudo-bulk signal for downstream peak calling.

## When NOT to use

- Input is still raw FASTQ/BAM — a snap file must first be created with 'snaptools snap-pre' (alignment, MAPQ >= 30, fragment length <= 1000 bp filtering) before read grouping is possible
- Only one bulk sample with no cell-level clustering exists — there are no cell subsets to group and standard bulk peak calling applies directly
- Working in the SnapATAC2 (scverse) Python API — its preprocessing and peak-calling API differs and does not use the SnapTools extractReads/runSnapAddPmat functions described here

## Inputs

- Snap-format (hdf5) .snap file with FM session of usable, indexed fragments per barcode
- Barcode-to-cluster assignment table from SnapATAC dimensionality reduction and Leiden clustering
- Per-cluster barcode lists defining the cell subsets to group

## Outputs

- Pseudo-bulk fragment/read signal per cell-type cluster (per-cluster aggregated coverage)
- Per-cluster peak calls (candidate CREs, merged and deduplicated, e.g. cre_peaks.bed with per-cluster peak counts)

## How to apply

Load the snap file whose FM session stores all usable, indexed fragments for each cell, together with the barcode-level cluster assignments from the SnapATAC embedding (e.g. 55,592 mouse secondary motor cortex nuclei). For each major cluster, use the read-grouping functions to select barcodes belonging to that cluster and aggregate their fragments into a single pseudo-bulk signal — SnapTools provides this via functions such as extractReads / runSnapAddPmat, which read the indexed FM session rather than re-parsing BAM files. Run the grouping under the version constraints the package requires (Python >= 2.7 & < 3.0 for SnapTools; R >= 3.4.0 & < 3.6.0 for SnapATAC, since R 3.6 breaks the rhdf5 package). Repeat per cluster (the paper aggregates 31 distinct cell populations), then feed each pseudo-bulk signal to MACS2 peak calling with '--nomodel --shift 100 --ext 200 --qval 1e-2 -B --SPMR'. The rationale is that individual nuclei detect only 5-15% of peaks, so per-cluster pooling is required before MACS2 can call peaks; merging per-cluster calls then yields the unique candidate CRE set.

## Related tools

- **SnapTools** (Python module whose indexed FM fragment session and grouping utilities (extractReads / runSnapAddPmat, runMACS wrappers) perform the per-cluster fragment aggregation) — https://github.com/r3fang/SnapTools
- **SnapATAC** (R package that loads the snap file, supplies cluster assignments per barcode, and drives the read-grouping workflow prior to MACS2 peak calling) — https://github.com/r3fang/SnapATAC

## Evaluation signals

- The snap file must contain a populated FM session ('FM session contains all usable fragments for each cell. Fragments are indexed for fast search.') — grouping fails or is pathologically slow if fragments are absent or unindexed
- Each grouped pseudo-bulk cluster should carry enough aggregated fragments for MACS2 to call peaks; per-cluster usable-fragment counts (UQ metrics from snap QC) should be checked before grouping
- Merging and deduplicating per-cluster peak calls on the MOs atlas should yield approximately 373,583 unique candidate CREs across 31 cell populations (~370,000 reported)
- Roughly 56% of the merged candidate elements should be undetectable in bulk ATAC-seq of the same region — a large drop in this fraction suggests grouping or merging errors
- Environment check: SnapTools runs under Python >= 2.7 & < 3.0 and SnapATAC under R >= 3.4.0 & < 3.6.0 (R 3.6 breaks rhdf5) — version mismatches are a common silent failure

## Limitations

- Iterative clustering plus aggregation plus peak calling requires multiple rounds, which hinders application to large-scale datasets
- In iterative schemes the crude clusters represent the most dominant cell types, so peaks in rare populations may still be underrepresented even after per-cluster grouping
- The legacy pipeline is version-pinned (Python 2.7 for SnapTools, R < 3.6 for SnapATAC/rhdf5); the maintained path is SnapATAC2 (scverse), which is not API-compatible

## Evidence

- [readme] FM session contains all usable fragments for each cell. Fragments are indexed for fast search.: [evidence span withheld — anchor status: external_doc]
- [readme] How to group reads from any subset of cells?: "How to group reads from any subset of cells?"
- [abstract] SnapATAC dissects cellular heterogeneity ... ~370,000 candidate regulatory elements in 31 distinct cell populations: "The analysis reveals ~370,000 candidate regulatory elements in 31 distinct cell populations in this brain region"
- [readme] Python (>= 2.7 & < 3.0) (SnapTools) (highly recommanded for 2.7);: "Python (>= 2.7 & < 3.0) (SnapTools) (highly recommanded for 2.7);"
- [discussion] Iterative clustering requires multiple rounds, hindering large-scale application: "the strategy of iterative clustering requires multiple rounds of clustering, aggregation, and peak calling, thus hindering its application to large-scale datasets."
