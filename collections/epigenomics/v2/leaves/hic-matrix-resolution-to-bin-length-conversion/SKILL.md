---
name: hic-matrix-resolution-to-bin-length-conversion
description: Use when you have per-bin cluster labels from an N×N intra-chromosomal
  Hi-C contact matrix and need to delineate TAD boundaries as genomic intervals for
  downstream visualization or evaluation.
when_to_use_negative:
- Input is still a raw NxN contact matrix — clustering via the GAE + HDBSCAN modules
  must run first before boundary delineation
- Regions of interest are shorter than 200 Kb, e.g. sub-TAD or loop-scale features
  — the TOAST length filter will discard them
- You need statistically parameterized TAD calls (e.g. Armatus, TopDom, TADbit dynamic-programming
  or statistical methods) rather than label-based contiguous-region delineation
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: TOAST
  role: Provides the TAD generation module that converts cluster labels to TAD boundaries
    and writes tad_boundaries.txt
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Supplies the per-bin cluster labels whose contiguous runs define TADs
  repo: https://github.com/ghaiyan/TOAST
- name: Pandas
  role: Tabular manipulation of per-bin labels and boundary records (listed as a required
    package of the TOAST environment)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:094f3c53e1d84760c227e8c78d7409e76fdceb39b2618627988b3caf9792f44e
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hic-matrix-resolution-to-bin-length-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-bin HDBSCAN cluster labels from the TOAST pipeline into genomic TAD boundaries by grouping contiguous same-label bins, converting bin counts to genomic length via the Hi-C matrix resolution (e.g., 50/25/5 Kb), and retaining only regions exceeding 200 Kb.

## When to use

Use when you have per-bin cluster labels from an N×N intra-chromosomal Hi-C contact matrix and need to delineate TAD boundaries as genomic intervals for downstream visualization or evaluation.

## When NOT to use

- Input is still a raw NxN contact matrix — clustering via the GAE + HDBSCAN modules must run first before boundary delineation
- Regions of interest are shorter than 200 Kb, e.g. sub-TAD or loop-scale features — the TOAST length filter will discard them
- You need statistically parameterized TAD calls (e.g. Armatus, TopDom, TADbit dynamic-programming or statistical methods) rather than label-based contiguous-region delineation

## Inputs

- Per-bin HDBSCAN cluster labels assigned to each genomic bin by the TOAST clustering module
- Hi-C contact matrix resolution (e.g., 50 Kb, 25 Kb, or 5 Kb) of the NxN intra-chromosomal contact matrix

## Outputs

- tad_boundaries.txt with records formatted as (start bin, start location, end bin, end location)
- TAD regions used for heatmap plotting (plot_heatmaps.py) and TAD evaluation

## How to apply

Load the per-bin HDBSCAN cluster labels together with the resolution of the source Hi-C contact matrix (50 Kb, 25 Kb, or 5 Kb). Group consecutive genomic bins sharing the same cluster label into contiguous regions, as done in the TOAST TAD generation module. Compute each region's genomic length as (number of bins) × (matrix resolution); at 50 Kb resolution this means a region must contain at least four same-label bins to pass the length filter. Retain only contiguous regions whose length exceeds 200 Kb — the article states 'If the length of a contiguous region exceeds 200 Kb, it can be identified as a TAD'. Write retained TADs to tad_boundaries.txt with each record formatted as (start bin, start location, end bin, end location), so boundaries can feed into heatmap plotting and downstream validation.

## Related tools

- **TOAST** (Provides the TAD generation module that converts cluster labels to TAD boundaries and writes tad_boundaries.txt) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Supplies the per-bin cluster labels whose contiguous runs define TADs) — https://github.com/ghaiyan/TOAST
- **Pandas** (Tabular manipulation of per-bin labels and boundary records (listed as a required package of the TOAST environment))

## Examples

```
python toast.py  # after editing the input (NxN contact matrix) and output (tad_boundaries.txt) folder paths in the script
```

## Evaluation signals

- tad_boundaries.txt exists and every record contains the four required fields: start bin, start location, end bin, end location
- Every retained TAD's genomic length exceeds 200 Kb (e.g., at 50 Kb resolution each retained region spans at least four same-label bins)
- Each output TAD corresponds to a run of consecutive bins with an identical HDBSCAN cluster label in the label file
- Boundary start/end genomic coordinates are consistent with the input matrix resolution (bin index × resolution)

## Limitations

- The 200 Kb length threshold removes all shorter contiguous same-label regions, so sub-200 Kb domains are never reported as TADs
- Quality of boundaries depends on the clustering step; the article reports optimal GAE settings of 2 layers, embedding dimension 16, hidden layer dimension 64, and deviations from these may degrade labels
- Resolution is fixed by the input matrix — the conversion from bins to genomic length is only valid for the resolution of the matrix the labels were computed on (50 Kb, 25 Kb, or 5 Kb)

## Evidence
