---
name: chromatin-contact-matrix-generation
description: Use when you hold a deduplicated ChIA-PET bedpe (e.g., in.rmDup.bedpe)
  and need a fixed-binsize Hi-C-style contact matrix (e.g., 10 kb, triplet sparse)
  to overlay with Hi-C maps or interrogate loop structure at anchors.
when_to_use_negative:
- Input is raw ChIA-PET FASTQ reads — run the full ChIA-PET2 pipeline (trimLinker,
  bwa alignment, buildBedpe, removeDup, peak calling) first to obtain the deduplicated
  bedpe
- Goal is a genome-wide Hi-C contact matrix from Hi-C sequencing reads — this skill
  converts PET counts at ChIA-PET loop anchors, not raw Hi-C contact frequencies
- Task is allele-specific interaction calling or per-loop statistical significance
  testing alone rather than contact map construction for overlay
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: ChIA-PET2
  role: Provides bedpe2Matrix to convert the deduplicated bedpe into a Hi-C-style
    contact matrix in triplet sparse format; the pipeline also produces the in.rmDup.bedpe
    input and supports 2-dimensional contact map output
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bedtools
  role: v2.24.0+ used for bed/bedpe file manipulation when preparing intervals and
    loop anchors for matrix construction
  repo: https://github.com/arq5x/bedtools2
- name: MICC
  role: Called via MICC2.R to estimate statistical significance of chromatin loops
    (-log10(1-PostProbability), FDR via Bayesian mixture model) so the contact matrix
    can be restricted to significant loops
- name: HiCPlotter
  role: Renders the overlay of the triplet-sparse ChIA-PET contact matrix with the
    matched-resolution Hi-C contact map for transitivity/redundancy inspection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:e32c0c81dbd5d3c97e94b4ce66a4402e6c891e6f5d0e99868f13b9c8c7e44b35
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
  - MICC
  - R
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chromatin-contact-matrix-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a Hi-C-style chromatin contact matrix from a deduplicated ChIA-PET bedpe using ChIA-PET2's bedpe2Matrix, so that CTCF-mediated loops can be directly compared with matched-resolution Hi-C maps (e.g., 10 kb GM12878) to test transitivity and redundancy of chromatin loops.

## When to use

Use when you hold a deduplicated ChIA-PET bedpe (e.g., in.rmDup.bedpe) and need a fixed-binsize Hi-C-style contact matrix (e.g., 10 kb, triplet sparse) to overlay with Hi-C maps or interrogate loop structure at anchors.

## When NOT to use

- Input is raw ChIA-PET FASTQ reads — run the full ChIA-PET2 pipeline (trimLinker, bwa alignment, buildBedpe, removeDup, peak calling) first to obtain the deduplicated bedpe
- Goal is a genome-wide Hi-C contact matrix from Hi-C sequencing reads — this skill converts PET counts at ChIA-PET loop anchors, not raw Hi-C contact frequencies
- Task is allele-specific interaction calling or per-loop statistical significance testing alone rather than contact map construction for overlay

## Inputs

- Deduplicated ChIA-PET bedpe file (e.g., in.rmDup.bedpe from ChIA-PET2 removeDup)
- Chromosome sizes file (e.g., chrom_hg19.sizes for hg19)
- Bin size parameter (e.g., --binsize 10000 for 10 kb resolution)
- Matching Hi-C contact map at the same resolution (10 kb, triplet sparse format) for overlay
- MICC2.R significance output (-log10(1-PostProbability) and FDR per loop) for filtering significant loops

## Outputs

- Hi-C-style contact matrix in triplet sparse format (bin pair plus contact count)
- HiCPlotter-compatible contact map restricted to regions covered by CTCF loops
- Side-by-side overlay figure of ChIA-PET CTCF contact matrix vs. Hi-C contact map
- Tabulation of transitivity and redundancy cases (e.g., L3a-L4a absent, L3b-L4a present)

## How to apply

Convert the deduplicated CTCF ChIA-PET bedpe (in.rmDup.bedpe) into a Hi-C-style contact matrix with ChIA-PET2's bedpe2Matrix using --binsize 10000 and the chromosome sizes file (chrom_hg19.sizes); ChIA-PET2 emits a 2-dimensional contact map as triplet sparse format, which is HiCPlotter-compatible. Import the matching GM12878 Hi-C contact map at the same 10 kb resolution for the regions covered by the CTCF loops, so both maps share bin coordinates derived from the same chrom_hg19.sizes. Restrict the ChIA-PET contact matrix to significant loops using the MICC2.R output, which estimates -log10(1-PostProbability) and FDR per loop via a Bayesian mixture model (default CUTOFFPET=2). Overlay the ChIA-PET contact matrix with the Hi-C contact map in HiCPlotter at loci where Hi-C implies a transitive contact (e.g., L3a-L4a given L1-L3a and L1-L4a) but the ChIA-PET CTCF loop is absent. Tabulate redundancy cases where one Hi-C loop corresponds to multiple CTCF-mediated loops and render the side-by-side comparison figure.

## Related tools

- **ChIA-PET2** (Provides bedpe2Matrix to convert the deduplicated bedpe into a Hi-C-style contact matrix in triplet sparse format; the pipeline also produces the in.rmDup.bedpe input and supports 2-dimensional contact map output) — https://github.com/GuipengLi/ChIA-PET2
- **bedtools** (v2.24.0+ used for bed/bedpe file manipulation when preparing intervals and loop anchors for matrix construction) — https://github.com/arq5x/bedtools2
- **MICC** (Called via MICC2.R to estimate statistical significance of chromatin loops (-log10(1-PostProbability), FDR via Bayesian mixture model) so the contact matrix can be restricted to significant loops)
- **HiCPlotter** (Renders the overlay of the triplet-sparse ChIA-PET contact matrix with the matched-resolution Hi-C contact map for transitivity/redundancy inspection)

## Examples

```
bedpe2Matrix -i in.rmDup.bedpe --binsize 10000 -g chrom_hg19.sizes -o ctcf_10kb.sparse
```

## Evaluation signals

- Output matrix is in triplet sparse format (compatible with HiCPlotter) with bins consistent with the chosen --binsize (10000) and the chrom_hg19.sizes coordinates
- Every significant loop pair surviving MICC2.R filtering (-log10(1-PostProbability), FDR) appears as a non-zero entry in the contact matrix
- Transitivity check: bins corresponding to L3a-L4a show no contact despite existing L1-L3a and L1-L4a contacts, while L3b-L4a shows a contact
- Bin resolution of the ChIA-PET matrix matches the imported GM12878 Hi-C map (both 10 kb) so overlay coordinates align
- Redundancy tabulation shows each Hi-C loop correctly mapped to its CTCF-mediated ChIA-PET loop(s)

## Limitations

- ChIA-PET2 calls chromatin loops based on ChIP-Seq-like peaks, so matrix signal depends on peak-calling sensitivity at anchors
- Only a few examples of the transitivity, redundancy and robustness of chromatin loops were provided in the article
- Resolution is bounded by the chosen binsize (10 kb here); finer loop structure requires smaller bins and more PETs

## Evidence
