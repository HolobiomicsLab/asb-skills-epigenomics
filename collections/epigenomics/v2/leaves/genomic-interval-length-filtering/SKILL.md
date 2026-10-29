---
name: genomic-interval-length-filtering
description: Use when you have per-bin labels (e.g., HDBSCAN clusters on Hi-C genomic
  bins at 50/25/5 Kb resolution) and need contiguous same-label regions filtered by
  a genomic length threshold to define TAD boundaries.
when_to_use_negative:
- Input is already a curated TAD boundary file rather than raw per-bin cluster labels
- Region labels come from a clustering whose bins are not on a uniform genomic grid,
  so bin count cannot be converted to genomic length via a fixed resolution
- The goal is sub-TAD or boundary refinement at scales below 200 Kb, which this filter
  explicitly discards
edam_operation: http://edamontology.org/operation_3902
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: TOAST
  role: Provides the TAD generation module that groups contiguous same-label bins
    and applies the 200 Kb length filter to produce tad_boundaries.txt
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Supplies the per-bin labels that the contiguous-region grouping and length
    filter operate on
- name: Pandas
  role: Used for handling per-bin label tables and writing the filtered TAD records
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:01c2031a72eeaa8f2502562be10b9c7e309e258e275913a5c5a74d85f9340a67
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - HDBSCAN
  - Pandas
  - TOAST
  edam_operation: http://edamontology.org/operation_3902
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# genomic-interval-length-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-bin cluster labels into TAD boundary calls by grouping consecutive same-label genomic bins into contiguous regions and retaining only regions longer than 200 Kb, as implemented in the TOAST TAD generation module.

## When to use

Use when you have per-bin labels (e.g., HDBSCAN clusters on Hi-C genomic bins at 50/25/5 Kb resolution) and need contiguous same-label regions filtered by a genomic length threshold to define TAD boundaries.

## When NOT to use

- Input is already a curated TAD boundary file rather than raw per-bin cluster labels
- Region labels come from a clustering whose bins are not on a uniform genomic grid, so bin count cannot be converted to genomic length via a fixed resolution
- The goal is sub-TAD or boundary refinement at scales below 200 Kb, which this filter explicitly discards

## Inputs

- Per-bin HDBSCAN cluster labels for each genomic bin
- Resolution of the source Hi-C contact matrix (50 Kb, 25 Kb, or 5 Kb)
- NxN intra-chromosomal Hi-C contact matrix defining bin indices

## Outputs

- tad_boundaries.txt with records formatted as (start bin, start location, end bin, end location)
- Set of TADs, each a contiguous same-label region longer than 200 Kb

## How to apply

Load the per-bin HDBSCAN cluster labels together with the resolution of the source Hi-C contact matrix (e.g., 50 Kb, 25 Kb, or 5 Kb). Group consecutive genomic bins sharing the same cluster label into contiguous regions, following the TOAST TAD generation module. Compute each region's genomic length as the number of same-label bins multiplied by the matrix resolution (e.g., at 50 Kb resolution a region must span at least four bins to exceed 200 Kb). Retain only regions whose length exceeds 200 Kb as TADs, and write them to tad_boundaries.txt with each record formatted as (start bin, start location, end bin, end location). The rationale is that short contiguous runs of identical labels are more likely clustering noise than genuine topologically associated domains.

## Related tools

- **TOAST** (Provides the TAD generation module that groups contiguous same-label bins and applies the 200 Kb length filter to produce tad_boundaries.txt) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Supplies the per-bin labels that the contiguous-region grouping and length filter operate on)
- **Pandas** (Used for handling per-bin label tables and writing the filtered TAD records)

## Examples

```
python toast.py  # within /src, set input and output folders; toast.py runs TAD calling (incl. TAD generation/200 Kb filter) and writes the TAD output file
```

## Evaluation signals

- tad_boundaries.txt exists and each record contains the four required fields (start bin, start location, end bin, end location)
- Every retained region exceeds the 200 Kb length threshold (e.g., >= 4 same-label bins at 50 Kb resolution)
- Each output TAD corresponds to a strictly contiguous run of bins sharing a single HDBSCAN cluster label
- Region genomic length equals bin count multiplied by the source matrix resolution

## Limitations

- Regions of same-label bins at or below 200 Kb are discarded, so genuine small TADs are lost by design
- The filter assumes a uniform bin resolution; mixed-resolution or ragged bin grids break the bin-count-to-length conversion
- Non-contiguous bins with the same label are never merged — contiguity is required

## Evidence
