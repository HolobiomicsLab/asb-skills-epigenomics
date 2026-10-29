---
name: rloop-drip-seq-enrichment-quantification
description: Use when DRIP-seq IP and control datasets (mock or RNaseH-treated) plus
  a coordinate set of HOT or occupancy-binned regions are available and the goal is
  to test RNA-DNA hybrid enrichment per region set.
when_to_use_negative:
- Input is ChIP-seq data for a knocked-out transcription factor — use the false-positive-signal-on-HOT-regions
  analysis instead of DRIP-seq R-loop quantification.
- The question is sequence characteristics such as GC/CpG-rich k-mers or G-quadruplex
  DNA rather than DRIP-seq-measured RNA-DNA hybrids — a sequence-feature analysis
  skill is the right choice.
- No matched DRIP-seq control (neither mock nor RNaseH-treated) exists for the sample,
  since log2(IP/control) enrichment cannot be computed on equal footing with control
  regions.
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: ENCODE
  role: Source of DRIP-seq IP and control raw datasets and region annotations used
    to quantify R-loop enrichment on HOT regions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:150418398f4067d0358b58fc43283ea4788fd40cd783acfcd67b0c2bb2315e26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENCODE
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# rloop-drip-seq-enrichment-quantification

## Summary

Quantifies R-loop (RNA-DNA hybrid) enrichment over high-occupancy target (HOT) regions by computing log2(IP/control) DRIP-seq signal against occupancy-binned control region sets, using RNaseH-treated DRIP-seq as control where a mock control is absent. This tests whether HOT regions' discriminatory enrichment of R-loops holds across cell lines and species.

## When to use

Use when DRIP-seq IP and control datasets (mock or RNaseH-treated) plus a coordinate set of HOT or occupancy-binned regions are available and the goal is to test RNA-DNA hybrid enrichment per region set.

## When NOT to use

- Input is ChIP-seq data for a knocked-out transcription factor — use the false-positive-signal-on-HOT-regions analysis instead of DRIP-seq R-loop quantification.
- The question is sequence characteristics such as GC/CpG-rich k-mers or G-quadruplex DNA rather than DRIP-seq-measured RNA-DNA hybrids — a sequence-feature analysis skill is the right choice.
- No matched DRIP-seq control (neither mock nor RNaseH-treated) exists for the sample, since log2(IP/control) enrichment cannot be computed on equal footing with control regions.

## Inputs

- DRIP-seq IP raw sequencing data (FASTQ) for each analyzed cell line and for C. elegans, with accessions from Supplementary Table S1
- DRIP-seq control datasets (mock control, or RNaseH-treated DRIP-seq where mock is unavailable)
- HOT region coordinate sets (BED-style interval files)
- Occupancy-binned control region sets (regions binned by TF occupancy percentiles)
- Appropriate reference genomes for each species/cell line

## Outputs

- Normalized per-base DRIP-seq IP and control coverage tracks per library
- Per-region log2(IP/control) enrichment values for HOT region sets
- Per-region log2(IP/control) enrichment values for occupancy-binned control region sets
- Aggregated enrichment summary table stratified by region set, cell line, and C. elegans

## How to apply

Retrieve DRIP-seq IP and control (RNaseH-treated or mock) raw datasets for each analyzed cell line and for C. elegans using accessions listed in Supplementary Table S1. Align reads to the appropriate reference genome and generate normalized per-base coverage tracks for IP and control libraries. Compute log2(IP/control) enrichment per genomic region over the HOT region coordinate sets, substituting RNaseH-treated DRIP-seq as the control in cell lines lacking a mock control, so the HOT and occupancy-binned control region sets are compared on equal footing. Aggregate enrichment values per region set, per cell line, and for C. elegans into a tabular summary. Rationale: the paper identifies enrichment of RNA-DNA hybrids (R-loops) as a discriminatory feature of HOT regions, so validation is the qualitative presence of enrichment on HOT regions in every analyzed sample rather than a fixed numeric cutoff.

## Related tools

- **ENCODE** (Source of DRIP-seq IP and control raw datasets and region annotations used to quantify R-loop enrichment on HOT regions)

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- log2(IP/control) enrichment on HOT regions is positive in every analyzed cell line, not just a subset.
- DRIP-seq enrichment on HOT regions is also observed in C. elegans.
- HOT regions and occupancy-binned control region sets were evaluated with the same log2(IP/control) computation so the comparison is on equal footing.
- Cell lines lacking a mock control used RNaseH-treated DRIP-seq as the control track instead.
- Aggregated results are tabulated per region set, per cell line, and for C. elegans, matching the workflow's summary step.

## Limitations

- Using RNaseH-treated DRIP-seq as a substitute control in cell lines lacking a mock control changes the nature of the background and is a stated substitution rather than a like-for-like mock comparison.
- R-loop enrichment is accepted as a qualitative feature (presence of enrichment across all analyzed samples), not a strict quantitative threshold.
- The paper frames HOT regions as likely ChIP-seq artifacts, so R-loop enrichment on HOT regions is interpreted as a discriminatory feature associated with artificial ChIP-seq enrichment, not evidence of functional TF co-binding.

## Evidence

- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [results] DRIP-seq enrichment on HOT regions was also observed in C. elegans: [evidence span withheld — anchor status: not_found]
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
