---
name: genomic-interval-binning-and-visualization
description: Reference for converting significant ChIA-PET loop calls (bedpe) into
  a 10 kb-binned contact matrix and visually comparing it against Hi-C data at loop
  anchors.
when_to_use_negative:
- Input is raw ChIA-PET FASTQ/SAM data still requiring linker trimming, alignment
  and duplicate removal (use the upstream ChIA-PET2 pipeline steps first)
- You need genome-wide loop calling statistics rather than a binned visual contact-matrix
  comparison at specific loci
edam_operation: http://edamontology.org/operation_0337
tools:
- name: ChIA-PET2
  role: Provides bedpe2Matrix to bin the deduplicated bedpe file into a Hi-C style
    contact matrix at a given binsize
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bedtools
  role: bed/bedpe file manipulation for interval handling
  repo: https://github.com/arq5x/bedtools2
- name: MICC
  role: MICC2.R calls MICC to estimate statistical significance (-log10(1-PostProbability),
    FDR) of chromatin loops before plotting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:a27544a2f9fe1331d69e3adb82f4beb47891b0a3b8782ae2f6fe7a3890454407
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
  - MICC
  - R
  - bedtools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Genomic interval binning and visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts ChIA-PET deduplicated bedpe interaction files into Hi-C style binned contact matrices (triplet sparse format) and overlays them with Hi-C maps to assess transitivity, redundancy and structure of chromatin loops.

## When to use

Reference for converting significant ChIA-PET loop calls (bedpe) into a 10 kb-binned contact matrix and visually comparing it against Hi-C data at loop anchors.

## When NOT to use

- Input is raw ChIA-PET FASTQ/SAM data still requiring linker trimming, alignment and duplicate removal (use the upstream ChIA-PET2 pipeline steps first)
- You need genome-wide loop calling statistics rather than a binned visual contact-matrix comparison at specific loci

## Inputs

- Deduplicated ChIA-PET bedpe file (e.g. in.rmDup.bedpe) from ChIA-PET2
- Chromosome sizes file (e.g. chrom_hg19.sizes) for hg19
- Hi-C contact map (e.g. GM12878) at matching 10 kb resolution
- MICC2.R significant-loop output with -log10(1-PostProbability) and FDR per loop

## Outputs

- Hi-C style binned contact matrix in triplet sparse format (HiCPlotter-compatible)
- Side-by-side overlay figure of CTCF ChIA-PET contact matrix vs Hi-C contact map
- Table of transitivity and redundancy cases among chromatin loops

## How to apply

Take the deduplicated ChIA-PET bedpe file (e.g. in.rmDup.bedpe produced by ChIA-PET2's removeDup) and run ChIA-PET2's bedpe2Matrix with --binsize 10000 and a chromosome sizes file (chrom_hg19.sizes) to produce a Hi-C style contact matrix in triplet sparse format. Import a matching Hi-C contact map (e.g. GM12878 from Rao et al.) at the same 10 kb resolution for the regions covered by the CTCF loops. Restrict to significant loops using MICC2.R output, which estimates -log10(1-PostProbability) and FDR per loop via a Bayesian mixture model. Overlay the CTCF contact matrix with the Hi-C contact map using HiCPlotter (the triplet sparse matrix is HiCPlotter-compatible) at loci where Hi-C implies a contact (transitivity) or where one Hi-C loop maps to multiple ChIA-PET loops (redundancy), then render a side-by-side comparison figure and tabulate the redundancy cases.

## Related tools

- **ChIA-PET2** (Provides bedpe2Matrix to bin the deduplicated bedpe file into a Hi-C style contact matrix at a given binsize) — https://github.com/GuipengLi/ChIA-PET2
- **bedtools** (bed/bedpe file manipulation for interval handling) — https://github.com/arq5x/bedtools2
- **MICC** (MICC2.R calls MICC to estimate statistical significance (-log10(1-PostProbability), FDR) of chromatin loops before plotting)

## Examples

```
bedpe2Matrix --binsize 10000 chrom_hg19.sizes in.rmDup.bedpe ctcf_10kb_matrix
```

## Evaluation signals

- Contact matrix is emitted in triplet sparse format compatible with HiCPlotter
- Binning resolution of ChIA-PET matrix and imported Hi-C map both equal 10 kb
- Plot only includes loops passing the MICC2.R significance cutoff (default CUTOFFPET=2)
- Transitivity check: anchors implied connected by Hi-C (e.g. L3a-L4a given L1-L3a and L1-L4a) show no ChIA-PET loop where the finding states none exists

## Limitations

- The article provides only a few examples of transitivity, redundancy and robustness of chromatin loops
- ChIA-PET2 calls loops based on ChIP-Seq-like peaks, so loop calling depends on peak-calling sensitivity
- No ChIA-PET2 software version is stated and no changelog is available in the repository

## Evidence
