---
name: genomic-interval-overlap-merging
description: Use when you have sorted, coordinate-anchored genomic intervals (e.g.
  REM BED records with chr, start, end) and need to collapse overlaps/adjacencies
  into clusters and quantify their span and multiplicity.
when_to_use_negative:
- The goal is de novo peak calling from raw ChIP-seq/accessibility signal — StitchIt
  REMs are already derived regions, not raw peaks
- Only a single interval list with no need for cluster-level statistics is required
  (e.g. simple overlap percentage filtering during a Region Query)
- REM-gene associations must be preserved per original REM ID but the merging step
  would collapse intervals that carry multiple distinct REM IDs
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: EpiRegio
  role: Source of the REM dataset and the CREM definition; REMs are genomic regions
    whose chromatin accessibility variations are associated with target gene expression
    changes
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: STITCHIT
  role: Algorithm underlying REM definition — interprets chromatin accessibility data
    with respect to variation in gene expression before interval merging
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:95ee45684a4d3ec8b6f4d111ba92773a8ead019629857720c4312da130bb00f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - STITCHIT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genomic-interval-overlap-merging

## Summary

Merges overlapping or adjacent genomic intervals (EpiRegio REMs from the StitchIt-based database) into Clusters of Regulatory EleMents (CREMs) and computes aggregate statistics (count, average REMs per cluster, span, complete-overlap clusters).

## When to use

Use when you have sorted, coordinate-anchored genomic intervals (e.g. REM BED records with chr, start, end) and need to collapse overlaps/adjacencies into clusters and quantify their span and multiplicity.

## When NOT to use

- The goal is de novo peak calling from raw ChIP-seq/accessibility signal — StitchIt REMs are already derived regions, not raw peaks
- Only a single interval list with no need for cluster-level statistics is required (e.g. simple overlap percentage filtering during a Region Query)
- REM-gene associations must be preserved per original REM ID but the merging step would collapse intervals that carry multiple distinct REM IDs

## Inputs

- BED-style REM dataset with REM IDs and chr, start, end coordinates (2,404,861 REMs from Zenodo record 3758494)
- REM-gene assignment records retained through the sort step

## Outputs

- CREM table with unique cluster IDs, member REMs, and cluster span in bp
- crem_statistics.csv with total CREMs, average REMs per CREM, mean span ± standard deviation, and count of complete-overlap CREMs

## How to apply

Load the REM dataset (REM IDs with chr, start, end) from Zenodo record 3758494, which contains all 2,404,861 REMs, keeping REM-gene assignments since the same interval can carry multiple REM IDs. Sort REMs by chromosome and start coordinate, then merge on each chromosome using the CREM rule: intervals overlapping by at least 1 bp or directly adjacent with a 0 bp gap belong to one cluster, and only clusters composed of a minimum of two REMs are retained as CREMs with unique IDs. For each CREM compute the number of member REMs and the span (max end minus min start) in bp, and count CREMs in which every REM fully overlaps all other REMs. Validate against the paper's reference values (365,286 CREMs; 3.5 REMs per CREM; 534 ± 460 bp mean span; 9,786 complete-overlap CREMs) and write the summary table to crem_statistics.csv.

## Related tools

- **EpiRegio** (Source of the REM dataset and the CREM definition; REMs are genomic regions whose chromatin accessibility variations are associated with target gene expression changes) — https://github.com/TeamRegio/EpiRegioDB
- **STITCHIT** (Algorithm underlying REM definition — interprets chromatin accessibility data with respect to variation in gene expression before interval merging)

## Examples

```
sort -k1,1 -k2,2n rem.bed | bedtools merge -d 0 -c 4 -o count > crems.bed
```

## Evaluation signals

- Computed total CREM count must equal 365,286
- Average REMs per CREM must equal 3.5
- Mean CREM span must be 534 bp with standard deviation 460 bp
- Count of complete-overlap CREMs (every REM fully overlaps all others in the cluster) must equal 9,786
- Only clusters with a minimum of two REMs are retained as CREMs, and merged clusters never span chromosome boundaries

## Limitations

- The CREM rule treats a 0 bp gap as adjacency, so directly abutting (non-overlapping) intervals are merged — this may over-cluster in dense regions
- Clusters composed of a single REM are discarded, so singleton REM statistics are not represented in CREM aggregates
- The same interval can carry multiple REM IDs (different gene assignments), which must be handled during sorting to avoid incorrect cluster membership counts

## Evidence

- [methods] Merge REMs on each chromosome into clusters using the CREM rule: intervals overlapping by at least 1 bp or directly adjacent with a 0 bp gap belong to one cluster: [evidence span withheld — anchor status: not_found]
- [results] Applying the CREM definition to the REM set yields 365,286 distinct CREMs containing 3.5 REMs on average spanning 534 bp (±460 bp): [evidence span withheld — anchor status: not_found]
- [methods] All dataset versions are available at our `Zenodo repository <https://zenodo.org/record/3758494#.Xp6JVi2w2Rt>`_: "All dataset versions are available at our `Zenodo repository <https://zenodo.org/record/3758494#.Xp6JVi2w2Rt>`_"
- [intro] EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression: "EpiRegio uses the StitchIt algorithm which interprets chromatin accessibility data with respect to variation in gene expression"
