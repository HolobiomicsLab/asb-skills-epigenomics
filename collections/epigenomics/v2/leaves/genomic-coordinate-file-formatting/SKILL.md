---
name: genomic-coordinate-file-formatting
description: Use when you have per-genomic-bin cluster labels (e.g., HDBSCAN output
  from TOAST embeddings) and must delineate contiguous same-label regions into a TAD
  boundary coordinate file at a stated matrix resolution.
when_to_use_negative:
- Input is a raw NxN intra-chromosomal contact matrix rather than per-bin cluster
  labels — run TOAST's data processing, GAE, and clustering modules first
- Contiguous same-label regions shorter than or equal to 200 Kb — these are explicitly
  not identified as TADs by the article's threshold
- The task is clustering or GAE embedding generation itself, not converting labels
  into boundary coordinates
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: Provides the TAD generation module that groups contiguous same-label bins,
    applies the 200 Kb filter, and writes the TAD boundary file; run via toast.py
    in the /src folder
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Supplies the per-bin cluster labels that the coordinate formatting step groups
    into contiguous TAD regions
- name: Pandas
  role: Tabular handling of bin labels and coordinate records when assembling the
    boundary file
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:f1f6c8289ca4b606654ee2ce776e473cf52cd6fb6afc5314d568010ac52a1e2b
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# genomic-coordinate-file-formatting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-bin HDBSCAN cluster labels from TOAST's TAD generation module into contiguous same-label regions and writes retained TADs as genomic coordinate records (start bin, start location, end bin, end location) to tad_boundaries.txt for downstream visualization.

## When to use

Use when you have per-genomic-bin cluster labels (e.g., HDBSCAN output from TOAST embeddings) and must delineate contiguous same-label regions into a TAD boundary coordinate file at a stated matrix resolution.

## When NOT to use

- Input is a raw NxN intra-chromosomal contact matrix rather than per-bin cluster labels — run TOAST's data processing, GAE, and clustering modules first
- Contiguous same-label regions shorter than or equal to 200 Kb — these are explicitly not identified as TADs by the article's threshold
- The task is clustering or GAE embedding generation itself, not converting labels into boundary coordinates

## Inputs

- per-bin HDBSCAN cluster labels produced by TOAST's clustering module
- Hi-C contact matrix resolution (50 Kb, 25 Kb, or 5 Kb) defining the genomic size of each bin

## Outputs

- tad_boundaries.txt with records formatted as (start bin, start location, end bin, end location)
- TAD coordinate intervals used for downstream heatmap plotting and evaluation

## How to apply

Load the per-bin HDBSCAN cluster labels together with the resolution of the source Hi-C contact matrix (e.g., 50 Kb, 25 Kb, or 5 Kb as used for the GM12878 cell line). Group consecutive genomic bins sharing the same cluster label into contiguous regions, as done in TOAST's TAD generation module. Compute each region's genomic length as number of bins × matrix resolution, and apply the article's threshold: only contiguous regions longer than 200 Kb are identified as TADs (e.g., at 50 Kb resolution a region must span at least four same-label bins). Write the retained TADs to tad_boundaries.txt with each record formatted as (start bin, start location, end bin, end location). This formatting step is what feeds TAD boundary information into TOAST's heatmap visualization and evaluation scripts.

## Related tools

- **TOAST** (Provides the TAD generation module that groups contiguous same-label bins, applies the 200 Kb filter, and writes the TAD boundary file; run via toast.py in the /src folder) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Supplies the per-bin cluster labels that the coordinate formatting step groups into contiguous TAD regions)
- **Pandas** (Tabular handling of bin labels and coordinate records when assembling the boundary file)

## Examples

```
cd /path/to/TOAST/src && python toast.py  # set input (NxN contact matrix) and output folder paths in the script to generate and plot TADs
```

## Evaluation signals

- tad_boundaries.txt exists and every record contains the four required fields: start bin, start location, end bin, end location
- Every retained TAD corresponds to a contiguous run of bins sharing the same HDBSCAN cluster label
- Each retained region's computed length (bin count × matrix resolution) exceeds the 200 Kb threshold
- At 50 Kb resolution, no retained region contains fewer than four same-label bins
- TAD boundaries overlay correctly on heatmap plots generated by plot_heatmaps.py

## Limitations

- Regions of same-label bins that do not exceed 200 Kb are discarded, so short TAD-like structures are absent from the output file
- The output granularity is fixed by the source contact matrix resolution; records are bin-indexed plus genomic locations, so resolution must be tracked alongside the file
- The README's output description is terse ('the output is the TAD file at the merge file'), so record formatting must be validated against the four-field convention rather than a documented schema

## Evidence
