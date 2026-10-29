---
name: rnaseh-control-selection
description: Use when computing per-region DRIP-seq log2(IP/control) R-loop enrichment
  over HOT regions or occupancy-binned control regions and a given cell line has no
  mock control, so the RNaseH-treated DRIP-seq library must serve as control.
when_to_use_negative:
- When a proper mock (input) DRIP-seq control exists for the cell line — the mock
  control should be used instead of substituting the RNaseH-treated library
- When the goal is ChIP-seq peak calling or filtering rather than R-loop (RNA-DNA
  hybrid) enrichment quantification from DRIP-seq data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: ENCODE
  role: Source of the DRIP-seq IP and control raw datasets used for R-loop enrichment
    analysis on HOT regions
- name: HOT-region-main.R
  role: Analysis script from the paper's repository containing the HOT-region analysis
    workflow
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:496488cefea41c7ec0024676f70db80c02c212db3fa245fb47187da1d96f6a36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - ENCODE
  - HOT-region-main.R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rnaseh-control-selection

## Summary

A DRIP-seq control-selection step for R-loop enrichment analysis: when a cell line lacks a mock (input) DRIP-seq control, the RNaseH-treated DRIP-seq library is substituted as the denominator when computing log2(IP/control) enrichment over HOT and occupancy-binned control regions.

## When to use

Use when computing per-region DRIP-seq log2(IP/control) R-loop enrichment over HOT regions or occupancy-binned control regions and a given cell line has no mock control, so the RNaseH-treated DRIP-seq library must serve as control.

## When NOT to use

- When a proper mock (input) DRIP-seq control exists for the cell line — the mock control should be used instead of substituting the RNaseH-treated library
- When the goal is ChIP-seq peak calling or filtering rather than R-loop (RNA-DNA hybrid) enrichment quantification from DRIP-seq data

## Inputs

- DRIP-seq IP raw sequencing datasets for each analyzed cell line and for C. elegans (accessions from Supplementary Table S1)
- RNaseH-treated DRIP-seq control libraries for cell lines lacking a mock control
- Mock (input) DRIP-seq control libraries where available
- HOT region coordinate sets (BED-style genomic intervals)
- Control region sets binned by TF occupancy percentiles
- Reference genome sequences for each analyzed species

## Outputs

- Normalized per-base DRIP-seq coverage tracks for IP and control libraries
- Per-region log2(IP/control) enrichment values over HOT regions and occupancy-binned control region sets
- Aggregated enrichment table per region set, per cell line, and for C. elegans

## How to apply

Retrieve the DRIP-seq IP and control (RNaseH-treated or mock) raw datasets for each analyzed cell line (and C. elegans) using the accessions listed in Supplementary Table S1, sourced from ENCODE. Align reads to the appropriate reference genome and generate normalized per-base coverage tracks for IP and control libraries. For each genomic region set — HOT region coordinates and control region sets binned by TF occupancy percentiles — compute log2(IP/control) enrichment per region; the decision point is that in cell lines lacking a mock control, the RNaseH-treated DRIP-seq library is substituted as the control so HOT and control regions are compared on equal footing. Aggregate enrichment values per region set, per cell line, and for C. elegans into a tabular output.

## Related tools

- **ENCODE** (Source of the DRIP-seq IP and control raw datasets used for R-loop enrichment analysis on HOT regions)
- **HOT-region-main.R** (Analysis script from the paper's repository containing the HOT-region analysis workflow) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- R-loop enrichment on HOT regions (positive log2(IP/control) over HOT regions relative to occupancy-binned controls) is observed in every analyzed cell line
- DRIP-seq enrichment on HOT regions is also observed in C. elegans
- HOT and control region sets are compared on equal footing — both use the same log2(IP/control) computation with RNaseH-treated substitution only where mock controls are absent
- Aggregated enrichment values are summarized in a tabular output per region set, per cell line, and for C. elegans

## Limitations

- Substituting RNaseH-treated DRIP-seq as control is only applied when a cell line lacks a mock control, so control quality varies across samples
- The paper's acceptance criterion is the qualitative presence of R-loop enrichment on HOT regions across all analyzed samples; no quantitative enrichment threshold is specified
- R-loop enrichment is interpreted as a discriminatory feature associated with artificial ChIP-seq enrichment on HOT regions, not as a measure of functional TF binding

## Evidence

- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [abstract] The artificial ChIP-seq enrichment on HOT regions could be associated to these discriminatory features.: "The artificial ChIP-seq enrichment on HOT regions could be associated to these discriminatory features."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
