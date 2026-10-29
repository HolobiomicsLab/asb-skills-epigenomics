---
name: hic-matrix-visualization-compatibility
description: Use when a ChIA-PET bedpe interaction file (e.g. in.rmDup.bedpe) needs
  to be binned into a genome-wide 2D contact matrix for heatmap-style visualization
  with Hi-C tools such as HiCPlotter.
when_to_use_negative:
- The input is raw FASTQ or SAM data — the bedpe2Matrix step requires a duplicate-removed
  bedpe file, which is produced by upstream ChIA-PET2 steps (trimLinker, buildBedpe,
  removeDup)
- The goal is significant chromatin loop calling rather than a binned contact map
  — use ChIA-PET2's bedpe2Interaction/MICC2.R pathway instead
- The visualization target requires a dense/cool-format Hi-C matrix rather than sparse
  triplet format
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Provides the bedpe2Matrix module that bins PET bedpe coordinates and aggregates
    PET counts per bin pair into the triplet-format contact matrix
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: HiCPlotter
  role: Renders the emitted triplet sparse contact matrix as a Hi-C style 2D contact
    map
- name: HiC-Pro
  role: Defines the triplet tuple sparse matrix format that ChIA-PET2 reuses for its
    contact matrix output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:66e119c4865761063688fc3d8a1baeb84e6b61024a4875ee7890d62cc6da7241
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
  - HiC-Pro
  - HiCPlotter
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hic-matrix-visualization-compatibility

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the ChIA-PET2 bedpe2Matrix step that converts a PCR-duplicate-removed PET bedpe file into a Hi-C style 2D contact matrix, stored in HiC-Pro-compatible triplet sparse format so the contact map can be rendered by HiCPlotter.

## When to use

Use when a ChIA-PET bedpe interaction file (e.g. in.rmDup.bedpe) needs to be binned into a genome-wide 2D contact matrix for heatmap-style visualization with Hi-C tools such as HiCPlotter.

## When NOT to use

- The input is raw FASTQ or SAM data — the bedpe2Matrix step requires a duplicate-removed bedpe file, which is produced by upstream ChIA-PET2 steps (trimLinker, buildBedpe, removeDup)
- The goal is significant chromatin loop calling rather than a binned contact map — use ChIA-PET2's bedpe2Interaction/MICC2.R pathway instead
- The visualization target requires a dense/cool-format Hi-C matrix rather than sparse triplet format

## Inputs

- PCR-duplicate-removed PET bedpe file (ChIA-PET2 intermediate, e.g. in.rmDup.bedpe / prefixname.rmdup.bedpe)
- Tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes with lines like 'chr1  249250621')
- Bin size parameter (e.g. 10000 bp)

## Outputs

- Hi-C style 2D contact matrix in triplet sparse format (HiC-Pro-compatible), named by the --oprefix prefix
- HiCPlotter-renderable 2D contact map image

## How to apply

Start from the PCR-duplicate-removed PET bedpe file (ChIA-PET2 intermediate prefixname.rmdup.bedpe, e.g. in.rmDup.bedpe) and a tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes formatted as 'chr1<TAB>249250621' per line). Run the ChIA-PET2 bedpe2Matrix module with --binsize 10000, --chrsizes pointing at the sizes file, --ifile pointing at the bedpe, and an --oprefix output prefix, optionally with --progress to monitor the run. bedpe2Matrix assigns each PET end to fixed-size genome bins and aggregates PET counts per bin pair to build the Hi-C style contact matrix. Because the ChIA-PET contact matrix is usually very sparse, the matrix is written in the same triplet tuple sparse format as HiC-Pro, named by the --oprefix prefix, which makes it directly renderable by HiCPlotter.

## Related tools

- **ChIA-PET2** (Provides the bedpe2Matrix module that bins PET bedpe coordinates and aggregates PET counts per bin pair into the triplet-format contact matrix) — https://github.com/GuipengLi/ChIA-PET2
- **HiCPlotter** (Renders the emitted triplet sparse contact matrix as a Hi-C style 2D contact map)
- **HiC-Pro** (Defines the triplet tuple sparse matrix format that ChIA-PET2 reuses for its contact matrix output)

## Examples

```
bedpe2Matrix --binsize 10000 --chrsizes chrom_hg19.sizes --ifile in.rmDup.bedpe --oprefix PREFIX --progress
```

## Evaluation signals

- The emitted triplet sparse matrix parses correctly as (bin1, bin2, count) tuples with the expected --oprefix naming
- Bin coordinates in the triplet matrix fall within the chromosome lengths given in the supplied chrom_hg19.sizes file
- Per-bin-pair counts are consistent with the number of PETs in the input in.rmDup.bedpe at the chosen 10000 bp binsize
- The matrix loads and renders in HiCPlotter without format errors, confirming HiC-Pro triplet compatibility
- Sparse occupancy of the matrix matches the expected sparsity of ChIA-PET contact maps

## Limitations

- The contact matrix is derived from ChIA-PET PET counts and is usually very sparse, so bin-level heatmaps may be mostly empty at fine binsizes
- ChIA-PET2 calls loops based on ChIP-Seq-like peaks, so interaction anchor quality depends on peak-calling sensitivity; the binned matrix inherits this anchor bias
- No software version of ChIA-PET2 or its dependencies (bwa, samtools, bedtools, MACS2, g++) is stated in the paper

## Evidence

- [readme] 2 modes of read alignments (short or long read alignment) and 2 dimensional contact map output.: [evidence span withheld — anchor status: not_found]
- [other] Because the ChIA-PET contact matrix is usually very sparse, ChIA-PET2 stores it in the same triplet tuple format as HiC-Pro, and the resulting Hi-C style contact map can be drawn by HiCPlotter.: [evidence span withheld — anchor status: not_found]
