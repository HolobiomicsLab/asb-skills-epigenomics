---
name: chia-pet-bedpe-binning
description: Use when PET pairs are already in a duplicate-removed bedpe file (e.g.
  prefixname.rmdup.bedpe from ChIA-PET2) and a genome-binned 2D contact matrix in
  triplet sparse format is needed for HiCPlotter-style visualization.
when_to_use_negative:
- The goal is significant chromatin loop calling with MICC statistics rather than
  a binned contact map — use the bedpe2Interaction / MICC2.R steps instead
- Input is raw fastq or unfiltered PET data — the bedpe must first be produced and
  PCR duplicates removed (trimLinker, buildBedpe, removeDup)
- A dense full matrix representation is required downstream, since ChIA-PET contact
  matrices are usually very sparse and are stored here only as triplet tuples
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Provides the bedpe2Matrix module that bins bedpe PET ends into fixed-size
    genome bins and writes the Hi-C-Pro-style triplet sparse contact matrix; takes
    the removeDup-produced rmdup.bedpe as input
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: HiCPlotter
  role: Renders the resulting triplet-format Hi-C style contact map as a 2D contact
    plot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:820cc32834fc994d9a3d582b3ef49b105552c2adea59038d4d63abd4fcda0836
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/GuipengLi/ChIA-PET2
  tools:
  - ChIA-PET2
  - HiCPlotter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chia-pet-bedpe-binning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a Hi-C style 2D contact map from ChIA-PET data by assigning each PET end of a duplicate-removed bedpe file to fixed-size genome bins and aggregating PET counts per bin pair, stored in the sparse triplet tuple format used by HiC-Pro so the map can be rendered by HiCPlotter.

## When to use

Use when PET pairs are already in a duplicate-removed bedpe file (e.g. prefixname.rmdup.bedpe from ChIA-PET2) and a genome-binned 2D contact matrix in triplet sparse format is needed for HiCPlotter-style visualization.

## When NOT to use

- The goal is significant chromatin loop calling with MICC statistics rather than a binned contact map — use the bedpe2Interaction / MICC2.R steps instead
- Input is raw fastq or unfiltered PET data — the bedpe must first be produced and PCR duplicates removed (trimLinker, buildBedpe, removeDup)
- A dense full matrix representation is required downstream, since ChIA-PET contact matrices are usually very sparse and are stored here only as triplet tuples

## Inputs

- PCR-duplicate-removed PET bedpe file (e.g. prefixname.rmdup.bedpe, columns chr/start/end/chr/start/end/name/score/strand1/strand2)
- Tab-delimited chromosome sizes file for bedtools (e.g. chrom_hg19.sizes: 'chr1<TAB>249250621')
- Bin size parameter (e.g. --binsize 10000 for 10 kb bins)

## Outputs

- Hi-C style 2D contact matrix in triplet sparse format (HiC-Pro-compatible tuple format), named by the --oprefix prefix
- Genome-binned contact map ready for rendering by HiCPlotter

## How to apply

Load the PCR-duplicate-removed PET bedpe file (ChIA-PET2 intermediate prefixname.rmdup.bedpe) together with a tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes with lines like 'chr1 249250621'). Run the ChIA-PET2 bedpe2Matrix module with --binsize 10000 --chrsizes chrom_hg19.sizes --ifile in.rmDup.bedpe --oprefix PREFIX --progress so each PET end is assigned to a fixed 10 kb genome bin. Aggregate PET counts over all bin pairs to form the Hi-C style contact matrix. Because ChIA-PET contact matrices are usually very sparse, write the result in the same triplet tuple sparse format as HiC-Pro, named by the --oprefix prefix, rather than as a dense matrix. Validate that the emitted triplet file parses correctly and that bin coordinates fall within the chromosome lengths supplied by the sizes file before rendering with HiCPlotter.

## Related tools

- **ChIA-PET2** (Provides the bedpe2Matrix module that bins bedpe PET ends into fixed-size genome bins and writes the Hi-C-Pro-style triplet sparse contact matrix; takes the removeDup-produced rmdup.bedpe as input) — https://github.com/GuipengLi/ChIA-PET2
- **HiCPlotter** (Renders the resulting triplet-format Hi-C style contact map as a 2D contact plot)

## Examples

```
ChIA-PET2 bedpe2Matrix --binsize 10000 --chrsizes chrom_hg19.sizes --ifile in.rmDup.bedpe --oprefix PREFIX --progress
```

## Evaluation signals

- The emitted triplet sparse matrix parses correctly as (bin1, bin2, count) tuples
- All bin coordinates fall within the chromosome lengths listed in the supplied chrom_hg19.sizes file
- Aggregated bin-pair counts are consistent with the PET counts in the input rmdup.bedpe file
- The matrix can be loaded and visualized by HiCPlotter as a 2D contact map
- Sparse (empty) bin pairs are omitted rather than materialized, matching the sparse triplet convention

## Limitations

- The ChIA-PET contact matrix is usually very sparse, so the output is only usable by tools that accept triplet sparse (HiC-Pro-style) format
- Resolution of the contact map is fixed by the chosen --binsize (e.g. 10000 bp); finer binning on sparse ChIA-PET data yields even emptier matrices
- The ChIA-PET2 README documents the 2D contact map output only in general terms ('2 dimensional contact map output'); the bedpe2Matrix option set here is grounded in the workflow notes, not a fully quoted README section

## Evidence
