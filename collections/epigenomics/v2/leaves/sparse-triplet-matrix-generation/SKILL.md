---
name: sparse-triplet-matrix-generation
description: Use when you have a duplicate-removed ChIA-PET bedpe file (e.g. prefixname.rmdup.bedpe)
  plus a tab-delimited chromosome sizes file and need a 2D genome-wide contact map
  rather than peak-anchored loop calls.
when_to_use_negative:
- You need statistically significant peak-anchored chromatin loops rather than a raw
  binned contact map — use bedpe2Interaction / MICC-based loop calling instead
- The interactions are already anchored to ChIP-seq-like peaks (bedpe2Interaction
  output) rather than raw genomic bins, so bin-level PET counting would discard peak
  resolution
- A dense full matrix or a non-HiC-Pro consumer format is required and the triplet
  sparse tuple format cannot be converted downstream
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: ChIA-PET2
  role: Provides the bedpe2Matrix module that assigns PET ends to fixed-size genome
    bins, aggregates bin-pair counts, and writes the triplet sparse contact matrix
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: HiCPlotter
  role: Renders the emitted triplet sparse contact matrix as a Hi-C style 2D contact
    map
- name: bedtools
  role: Defines the chromosome sizes genome file convention (tab-delimited chr/size)
    consumed as --chrsizes/--bedtoolsgenome for binning
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:ed213dd21c57e17276448835336f368438a61da347d36dc69328bca8e8ef5a55
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
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sparse-triplet-matrix-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a Hi-C style 2D contact matrix from ChIA-PET paired-end tag (PET) data by binning PET ends from a PCR-duplicate-removed bedpe file into fixed-size genome bins and aggregating counts per bin pair, stored in the triplet sparse (HiC-Pro-style) tuple format for rendering with HiCPlotter.

## When to use

Use when you have a duplicate-removed ChIA-PET bedpe file (e.g. prefixname.rmdup.bedpe) plus a tab-delimited chromosome sizes file and need a 2D genome-wide contact map rather than peak-anchored loop calls.

## When NOT to use

- You need statistically significant peak-anchored chromatin loops rather than a raw binned contact map — use bedpe2Interaction / MICC-based loop calling instead
- The interactions are already anchored to ChIP-seq-like peaks (bedpe2Interaction output) rather than raw genomic bins, so bin-level PET counting would discard peak resolution
- A dense full matrix or a non-HiC-Pro consumer format is required and the triplet sparse tuple format cannot be converted downstream

## Inputs

- PCR-duplicate-removed PET bedpe file (ChIA-PET2 intermediate prefixname.rmdup.bedpe, 10-column chr/start/end/chr/start/end/name/score/strand1/strand2 format)
- Tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes with lines like 'chr1  249250621')
- Bin size parameter (e.g. --binsize 10000 for 10 kb bins)

## Outputs

- Triplet sparse-format Hi-C style contact matrix named by the --oprefix prefix, compatible with HiC-Pro tuple format and renderable by HiCPlotter

## How to apply

Run the ChIA-PET2 bedpe2Matrix module on the PCR-duplicate-removed PET bedpe (e.g. in.rmDup.bedpe, the prefixname.rmdup.bedpe intermediate) with a fixed --binsize (e.g. 10000 bp) and the tab-delimited chromosome sizes file (--chrsizes chrom_hg19.sizes). The module assigns each PET end to a fixed-size genome bin, aggregates PET counts per bin pair into a Hi-C style contact matrix, and writes the result under the --oprefix prefix. Because the ChIA-PET contact matrix is usually very sparse, the matrix is stored in the triplet tuple format used by HiC-Pro rather than as a dense matrix, which lets HiCPlotter draw the resulting contact map. Choose the bin size to trade resolution against sparsity, and validate that emitted bin coordinates fall within the chromosome sizes supplied.

## Related tools

- **ChIA-PET2** (Provides the bedpe2Matrix module that assigns PET ends to fixed-size genome bins, aggregates bin-pair counts, and writes the triplet sparse contact matrix) — https://github.com/GuipengLi/ChIA-PET2
- **HiCPlotter** (Renders the emitted triplet sparse contact matrix as a Hi-C style 2D contact map)
- **bedtools** (Defines the chromosome sizes genome file convention (tab-delimited chr/size) consumed as --chrsizes/--bedtoolsgenome for binning) — https://github.com/arq5x/bedtools2

## Examples

```
bedpe2Matrix --binsize 10000 --chrsizes chrom_hg19.sizes --ifile in.rmDup.bedpe --oprefix PREFIX --progress
```

## Evaluation signals

- The output triplet sparse matrix parses as valid (row, column, count) tuples in the HiC-Pro triplet format
- Every bin coordinate in the matrix falls within the chromosome lengths given in the supplied chrom_hg19.sizes file
- Per-bin-pair PET counts are consistent with the number of PETs in the input in.rmDup.bedpe file falling in each bin pair
- The matrix can be loaded and visualized by HiCPlotter without format errors
- Bin size in the output matches the --binsize value supplied (e.g. 10000 bp)

## Limitations

- The ChIA-PET contact matrix is usually very sparse, so resolution is limited by PET depth at the chosen bin size
- The contact map is derived from duplicate-removed PETs (prefixname.rmdup.bedpe), so it reflects raw interaction frequencies, not MICC statistical significance
- Output depends on an accurate tab-delimited chromosome sizes file matching the genome build (e.g. hg19)

## Evidence

- [results] Because the ChIA-PET contact matrix is usually very sparse, ChIA-PET2 stores it in the same triplet tuple format as HiC-Pro: [evidence span withheld — anchor status: not_found]
