---
name: crem-cluster-statistics-reproduction
description: Use when you have chromosome start/end coordinates of REMs (e.g. the
  2,404,861 REMs from Zenodo record 3758494) and need to merge them into CREMs and
  recompute the paper's clustering statistics.
when_to_use_negative:
- The input is not interval data with chr/start/end REM coordinates (e.g. it is already
  an aggregated feature or statistics table).
- The goal is motif enrichment or target-gene identification for a set of REMs (use
  the PASTAA/Gene Query workflow instead of CREM clustering).
- You need individual REM-gene associations rather than merged clusters of overlapping/adjacent
  REMs.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EpiRegio
  role: Source of the REM dataset being clustered; EpiRegio groups overlapping/adjacent
    REMs into CREMs with unique IDs, and the Zenodo record 3758494 provides all REMs
    from the released database version
  repo: github.com/TeamRegio/EpiRegioDB
- name: STITCHIT
  role: Algorithm underlying EpiRegio that produced the REMs; its linear model interprets
    chromatin accessibility data with respect to variation in gene expression
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:1535da8afaa73ac33620182444823eb5b94ee70eebd8b3bbdef037f8dc017eac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# crem-cluster-statistics-reproduction

## Summary

Reproduces EpiRegio's definition and quantitative characterization of Clusters of Regulatory EleMents (CREMs), i.e. groups of overlapping or directly adjacent REMs, and validates the aggregate statistics (total CREM count, average REMs per CREM, mean span, fully-overlapping clusters) against the published REM/CREM characteristics.

## When to use

Use when you have chromosome start/end coordinates of REMs (e.g. the 2,404,861 REMs from Zenodo record 3758494) and need to merge them into CREMs and recompute the paper's clustering statistics.

## When NOT to use

- The input is not interval data with chr/start/end REM coordinates (e.g. it is already an aggregated feature or statistics table).
- The goal is motif enrichment or target-gene identification for a set of REMs (use the PASTAA/Gene Query workflow instead of CREM clustering).
- You need individual REM-gene associations rather than merged clusters of overlapping/adjacent REMs.

## Inputs

- REM dataset from Zenodo record 3758494 with 2,404,861 REMs (REM IDs plus chr, start, end coordinates and REM-gene assignments)

## Outputs

- CREM table with unique CREM IDs and member REMs (minimum of two REMs per CREM)
- crem_statistics.csv with total CREM count, average REMs per CREM, mean span ± standard deviation (bp), and count of complete-overlap CREMs

## How to apply

Load the REM dataset (REM IDs with chr, start, end and REM-gene assignments) from the deposited Zenodo record 3758494 and sort REMs by chromosome and start coordinate, keeping gene assignments since one interval can carry multiple REM IDs. Merge intervals on each chromosome using the CREM rule: REMs overlapping by at least 1 bp, or directly adjacent with a 0 bp gap, fall into the same cluster, and only clusters composed of a minimum of two REMs are retained as CREMs with unique IDs. For each CREM, compute the number of member REMs and the CREM span (max end minus min start) in bp, and count CREMs in which every REM fully overlaps all other REMs in the cluster. Aggregate the summary statistics (total CREMs, average REMs per CREM, mean span ± standard deviation, complete-overlap count), compare them with the reported values, and write the table to crem_statistics.csv. The rationale is that EpiRegio groups overlapping/adjacent REMs into CREMs with unique IDs so that regulatory elements linked to the same gene can be retrieved and interpreted as consolidated units.

## Related tools

- **EpiRegio** (Source of the REM dataset being clustered; EpiRegio groups overlapping/adjacent REMs into CREMs with unique IDs, and the Zenodo record 3758494 provides all REMs from the released database version) — github.com/TeamRegio/EpiRegioDB
- **STITCHIT** (Algorithm underlying EpiRegio that produced the REMs; its linear model interprets chromatin accessibility data with respect to variation in gene expression)

## Evaluation signals

- Computed total CREM count must equal 365,286 distinct CREMs
- Average number of REMs per CREM must equal 3.5
- Mean CREM span must be 534 bp with a standard deviation of 460 bp
- Count of CREMs in which all contained REMs overlap completely must equal 9,786
- Every retained CREM must contain at least two REMs, and CREM span must equal max end minus min start of its member REMs

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of the actual regulation of genes by their REMs.
- The same genomic interval can carry multiple REM IDs (multiple REM-gene assignments), so REM counting must preserve assignments rather than deduplicate intervals blindly.
- No tool versions are stated for the software mentioned, so exact reproducibility may depend on the database release retrieved from Zenodo.

## Evidence

- [intro] Distinctive approach: EpiRegio starts from a gene and looks for REMs, rather than the reverse, and groups overlapping/adjacent REMs into CREMs with unique IDs.: "In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa."
- [methods] CREM statistics are computed from the full REM set deposited on Zenodo record 3758494.: "All dataset versions are available at our `Zenodo repository <https://zenodo.org/record/3758494#.Xp6JVi2w2Rt>`_"
- [readme] EpiRegio REMs underlie the CREM clustering and are linked to protein-coding and non-protein-coding genes.: "EpiRegio is a resource of Regulatory EleMents (REMs) associated to protein coding, and non-protein coding genes."
- [intro] EpiRegio's REMs derive from the StitchIt algorithm linking chromatin accessibility variation to gene expression variation.: "EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression"
