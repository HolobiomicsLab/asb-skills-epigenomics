---
name: genomic-bin-assignment-from-chromosome-sizes
description: Use when a PET/interaction bedpe file must be aggregated into a fixed-bin
  (e.g. 10 kb) 2D contact map whose bin grid is bounded by a chromosome sizes file
  such as chrom_hg19.sizes.
when_to_use_negative:
- The goal is significant chromatin loop calling from peaks rather than a binned contact
  matrix — use the ChIA-PET2 bedpe2Interaction module instead
- Input is already a dense or sparse contact matrix and only rendering/normalization
  is needed
- The bedpe coordinate genome build does not match the chromosome sizes file, so bin
  coordinates would be inconsistent
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Provides the bedpe2Matrix module that assigns PET ends to fixed-size genome
    bins and writes the triplet sparse contact matrix
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bedtools
  role: Defines the tab-delimited chromosome sizes (bedtools genome) file format consumed
    as the binning reference
  repo: https://github.com/arq5x/bedtools2
- name: HiC-Pro
  role: Reference for the triplet tuple sparse matrix output format of the contact
    matrix
- name: HiCPlotter
  role: Renders the resulting Hi-C style 2D contact map for visual validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:1791969fe8fb4511cf8a6e68c303db2496c3bf7ac6eb99703f8dadbf65efd656
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
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Genomic bin assignment from chromosome sizes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a PCR-duplicate-removed PET bedpe file into a Hi-C style 2D contact matrix by assigning each PET end to fixed-size genome bins defined by a tab-delimited chromosome sizes file, storing the result in the sparse triplet tuple format shared with HiC-Pro.

## When to use

Use when a PET/interaction bedpe file must be aggregated into a fixed-bin (e.g. 10 kb) 2D contact map whose bin grid is bounded by a chromosome sizes file such as chrom_hg19.sizes.

## When NOT to use

- The goal is significant chromatin loop calling from peaks rather than a binned contact matrix — use the ChIA-PET2 bedpe2Interaction module instead
- Input is already a dense or sparse contact matrix and only rendering/normalization is needed
- The bedpe coordinate genome build does not match the chromosome sizes file, so bin coordinates would be inconsistent

## Inputs

- PCR-duplicate-removed PET bedpe file (e.g. prefixname.rmdup.bedpe / in.rmDup.bedpe, 10-column bedpe: chr, start, end, chr, start, end, name, score, strand1, strand2)
- Tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes)
- Bin size parameter (e.g. --binsize 10000)

## Outputs

- Hi-C style 2D contact matrix in triplet sparse (tuple) format compatible with HiC-Pro, named by the --oprefix prefix
- Contact map renderable by HiCPlotter

## How to apply

Load the duplicate-removed PET file (e.g. prefixname.rmdup.bedpe, the ChIA-PET2 removeDup output) together with a tab-delimited chromosome sizes file (e.g. chrom_hg19.sizes with 'chr1 249250621' style rows). Run the ChIA-PET2 bedpe2Matrix module with --binsize 10000 (10 kb bins), --chrsizes, --ifile and --oprefix so each PET end is assigned to a fixed-size genome bin. Aggregate PET counts per bin pair to build the Hi-C style contact matrix; because the ChIA-PET contact matrix is usually very sparse, write it in the triplet tuple sparse format used by HiC-Pro, named by the --oprefix prefix. The chromosome sizes file must be tab delimited and consistent with the genome build of the bedpe coordinates, since it defines the bin grid extent.

## Related tools

- **ChIA-PET2** (Provides the bedpe2Matrix module that assigns PET ends to fixed-size genome bins and writes the triplet sparse contact matrix) — https://github.com/GuipengLi/ChIA-PET2
- **bedtools** (Defines the tab-delimited chromosome sizes (bedtools genome) file format consumed as the binning reference) — https://github.com/arq5x/bedtools2
- **HiC-Pro** (Reference for the triplet tuple sparse matrix output format of the contact matrix)
- **HiCPlotter** (Renders the resulting Hi-C style 2D contact map for visual validation)

## Examples

```
bedpe2Matrix --binsize 10000 --chrsizes chrom_hg19.sizes --ifile in.rmDup.bedpe --oprefix PREFIX --progress
```

## Evaluation signals

- The emitted triplet sparse matrix parses correctly as (bin_i, bin_j, count) tuples in the HiC-Pro-compatible format
- Bin coordinates in the matrix fall within the chromosome lengths given in the supplied tab-delimited sizes file (e.g. chr1 <= 249250621 for hg19)
- PET counts per bin pair are consistent with the number of PETs in the input rmdup.bedpe falling in each bin pair
- The contact map can be visualized with HiCPlotter without coordinate errors
- Re-running with the same --binsize and --chrsizes reproduces an identical matrix

## Limitations

- The ChIA-PET contact matrix is usually very sparse, so a triplet sparse representation is required rather than a dense matrix
- The genome size file must be tab delimited and match the genome build of the bedpe coordinates; the README documents the required 'chr1  249250621' structure
- Bin resolution is fixed by --binsize (e.g. 10000); no adaptive binning is provided

## Evidence

- [other] bedpe2Matrix invocation with fixed binsize and chrsizes: [evidence span withheld — anchor status: not_found]
- [other] Triplet sparse storage rationale: [evidence span withheld — anchor status: not_found]
