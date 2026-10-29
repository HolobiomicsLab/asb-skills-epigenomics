---
name: rem-clustering-into-crems
description: Use when you have interval-formatted REM/genomic region coordinates (chr,
  start, end) and need to merge overlapping or adjacent regions into clusters and
  report aggregate span/size statistics.
when_to_use_negative:
- You only need individual REMs and their associated genes (e.g. a Gene Query), not
  the merged CREM-level grouping of overlapping/adjacent REMs
- Your intervals require a different merging criterion, e.g. the EpiRegio Region Query
  50% REM-overlap threshold for ChIP-seq peaks rather than >= 1 bp overlap or 0 bp
  gap adjacency
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: EpiRegio
  role: Source of the REM dataset and the CREM definition (overlapping/adjacent REMs
    grouped into CREMs with unique IDs)
  repo: github.com/TeamRegio/EpiRegioDB
- name: STITCHIT
  role: Upstream algorithm that identified the REMs (via chromatin accessibility vs.
    gene expression variation) which are the input intervals for CREM clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:f90420b1b8e42e3396418bb2f1a109aa0e46d84b74f4a482d4ebd956bdc86acb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rem-clustering-into-crems

## Summary

Reproduces EpiRegio's definition of Clusters of Regulatory EleMents (CREMs): per chromosome, REMs overlapping by at least 1 bp or directly adjacent (0 bp gap) are merged, and only clusters containing a minimum of two REMs are retained as CREMs, enabling quantitative characterization of the 2,404,861 REM set.

## When to use

Use when you have interval-formatted REM/genomic region coordinates (chr, start, end) and need to merge overlapping or adjacent regions into clusters and report aggregate span/size statistics.

## When NOT to use

- You only need individual REMs and their associated genes (e.g. a Gene Query), not the merged CREM-level grouping of overlapping/adjacent REMs
- Your intervals require a different merging criterion, e.g. the EpiRegio Region Query 50% REM-overlap threshold for ChIP-seq peaks rather than >= 1 bp overlap or 0 bp gap adjacency

## Inputs

- REM dataset from Zenodo record 3758494 with REM IDs and chr, start, end coordinates (2,404,861 REMs)
- REM-to-gene assignments associated with each interval

## Outputs

- CREM list with unique CREM IDs and member REM assignments (clusters of >= 2 REMs)
- Per-CREM statistics: number of member REMs and span in base pairs
- Aggregate summary table (e.g. crem_statistics.csv): total CREMs, average REMs per CREM, mean span ± SD, count of complete-overlap CREMs

## How to apply

Load the REM dataset (REM IDs with chr, start, end coordinates) from Zenodo record 3758494, which contains all 2,404,861 REMs from the released EpiRegio database version. Sort REMs by chromosome and start coordinate, keeping REM-gene assignments since the same interval can carry multiple REM IDs. On each chromosome, merge intervals using the CREM rule — overlap of at least 1 bp or direct adjacency with a 0 bp gap belongs to one cluster — and retain only clusters with a minimum of two REMs, assigning each CREM a unique ID. For each CREM compute the number of member REMs and the span (max end minus min start) in bp, and additionally count CREMs with complete overlap, i.e. clusters in which every REM fully overlaps all other REMs. Aggregate the statistics (total CREM count, average REMs per CREM, mean span ± standard deviation, count of complete-overlap CREMs) into a table and compare against the reported values of 365,286 CREMs, 3.5 REMs per CREM, 534 ± 460 bp average span, and 9,786 fully overlapping CREMs.

## Related tools

- **EpiRegio** (Source of the REM dataset and the CREM definition (overlapping/adjacent REMs grouped into CREMs with unique IDs)) — github.com/TeamRegio/EpiRegioDB
- **STITCHIT** (Upstream algorithm that identified the REMs (via chromatin accessibility vs. gene expression variation) which are the input intervals for CREM clustering)

## Examples

```
python merge_rems.py --rem REMs.tsv --sort chr,start --merge-overlap 1 --min-members 2 --out crem_statistics.csv
```

## Evaluation signals

- The computed total CREM count must equal exactly 365,286
- Average REMs per CREM must match 3.5 REMs per CREM
- Mean CREM span must match 534 bp with standard deviation ±460 bp
- The count of CREMs in which all contained REMs overlap completely must equal 9,786
- No retained cluster contains fewer than two REMs, and clusters are formed only from >= 1 bp overlaps or 0 bp gaps within a single chromosome

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs
- The same REM interval can carry multiple REM IDs / REM-gene assignments, so interval bookkeeping must preserve these assignments during merging

## Evidence

- [intro] In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa.: "In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa."
- [methods] All dataset versions are available at our Zenodo repository.: "All dataset versions are available at our `Zenodo repository <https://zenodo.org/record/3758494#.Xp6JVi2w2Rt>`_"
- [discussion] the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
