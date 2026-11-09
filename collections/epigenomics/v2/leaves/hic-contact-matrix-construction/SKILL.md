---
name: hic-contact-matrix-construction
description: Use when you have a PCR-duplicate-removed ChIA-PET bedpe file (e.g. prefixname.rmdup.bedpe)
  and need a genome-binned 2D contact map for visualization instead of, or in addition
  to, loop calls.
when_to_use_negative:
- You need significant chromatin loop calls with PET counts, peak depths, MICC post-probabilities
  and FDR (use bedpe2Interaction + MICC2.R instead)
- Input is still raw fastq or unfiltered SAM files — the full ChIA-PET2 pipeline (trimLinker,
  mapping, buildBedpe, removeDup) must run first
- Input interaction data is not in bedpe PET format and cannot be assigned to genomic
  bin pairs by PET ends
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Provides the bedpe2Matrix module that bins bedpe PET ends, aggregates counts
    per bin pair, and writes the triplet sparse contact matrix
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: HiCPlotter
  role: Renders the resulting Hi-C style 2D contact map from the triplet sparse matrix
- name: bedtools
  role: Upstream dependency used by ChIA-PET2 for bed/bedpe file manipulation (e.g.
    pairToBed in the interaction step) and whose genome file format defines the chromosome
    sizes input
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:a359467034984a1eec6b202eef40c6abcadf5e366d3142a9201f05404ce3ec9d
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# hic-contact-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a Hi-C style 2D contact matrix from ChIA-PET data using the ChIA-PET2 bedpe2Matrix module, which bins PET ends at a fixed binsize (e.g. 10000 bp) and aggregates counts per bin pair into a triplet sparse matrix compatible with HiC-Pro/HiCPlotter.

## When to use

Use when you have a PCR-duplicate-removed ChIA-PET bedpe file (e.g. prefixname.rmdup.bedpe) and need a genome-binned 2D contact map for visualization instead of, or in addition to, loop calls.

## When NOT to use

- You need significant chromatin loop calls with PET counts, peak depths, MICC post-probabilities and FDR (use bedpe2Interaction + MICC2.R instead)
- Input is still raw fastq or unfiltered SAM files — the full ChIA-PET2 pipeline (trimLinker, mapping, buildBedpe, removeDup) must run first
- Input interaction data is not in bedpe PET format and cannot be assigned to genomic bin pairs by PET ends

## Inputs

- PCR-duplicate-removed PET bedpe file (ChIA-PET2 intermediate prefixname.rmdup.bedpe, 10-column bedpe: chr, start, end, chr, start, end, name, score, strand1, strand2)
- Tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes: 'chr1\t249250621')
- Bin size parameter (e.g. --binsize 10000 for 10 kb genome bins)

## Outputs

- Hi-C style 2D contact matrix in triplet tuple sparse format (HiC-Pro-compatible), named by the --oprefix prefix
- Contact map renderable by HiCPlotter

## How to apply

Take the ChIA-PET2 intermediate bedpe file with PCR duplicates removed (prefixname.rmdup.bedpe) and a tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes with lines like 'chr1 249250621'). Run the bedpe2Matrix module with --binsize 10000 (10 kb bins), --chrsizes, --ifile, and an --oprefix; the module assigns each PET end to fixed-size genome bins and aggregates PET counts per bin pair. Because ChIA-PET contact matrices are usually very sparse, the resulting Hi-C style matrix is written in the same triplet tuple sparse format used by HiC-Pro, named by the --oprefix prefix. Validate by parsing the triplet matrix and checking bin coordinates and counts against the chromosome sizes file, then render the map with HiCPlotter.

## Related tools

- **ChIA-PET2** (Provides the bedpe2Matrix module that bins bedpe PET ends, aggregates counts per bin pair, and writes the triplet sparse contact matrix) — https://github.com/GuipengLi/ChIA-PET2
- **HiCPlotter** (Renders the resulting Hi-C style 2D contact map from the triplet sparse matrix)
- **bedtools** (Upstream dependency used by ChIA-PET2 for bed/bedpe file manipulation (e.g. pairToBed in the interaction step) and whose genome file format defines the chromosome sizes input) — https://github.com/arq5x/bedtools2

## Examples

```
bedpe2Matrix --binsize 10000 --chrsizes chrom_hg19.sizes --ifile in.rmDup.bedpe --oprefix PREFIX --progress
```

## Evaluation signals

- The emitted triplet sparse matrix parses correctly (tuple rows: bin1, bin2, count) and follows the HiC-Pro triplet tuple format
- All bin coordinates fall within chromosome lengths defined in the supplied chrom_hg19.sizes file
- PET counts aggregated per bin pair are consistent with the input rmdup.bedpe entries at the chosen 10 kb binsize
- The matrix file is named with the --oprefix prefix and can be loaded and visualized by HiCPlotter
- The resulting contact map is sparse (most bin pairs zero/nonexistent), as expected for ChIA-PET-derived contact matrices

## Limitations

- ChIA-PET contact matrices are usually very sparse, so counts per bin pair are low compared with true Hi-C libraries
- Bin resolution is limited by the --binsize choice (e.g. 10000 bp); finer binning increases sparsity
- Only a 2D contact map is produced; statistical significance of loops (MICC post-probability and FDR) requires the separate bedpe2Interaction/MICC2.R workflow

## Evidence

- [other] Because the ChIA-PET contact matrix is usually very sparse, ChIA-PET2 stores it in the same triplet tuple format as HiC-Pro, and the resulting Hi-C style contact map can be drawn by HiCPlotter.: [evidence span withheld — anchor status: not_found]
