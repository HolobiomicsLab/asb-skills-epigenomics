---
name: ip-control-log2-ratio-computation
description: Use when testing whether a defined region set (e.g., HOT regions) is
  enriched for R-loops by comparing DRIP-seq IP coverage against RNaseH-treated or
  mock controls over occupancy-matched control regions.
when_to_use_negative:
- Input regions are not defined relative to TF occupancy, so HOT regions and control
  regions cannot be compared on equal footing via occupancy-binned percentiles
- No IP or control (mock or RNaseH-treated) DRIP-seq data are available for the cell
  line or organism in question, since log2(IP/control) cannot be computed
- The goal is to characterize ChIP-seq artifacts via knocked-out TF ChIP-seq rather
  than to quantify R-loop enrichment
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: ENCODE
  role: Source of DRIP-seq IP and control raw datasets for the analyzed cell lines
    (accessions listed in Supplementary Table S1)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:28d2fec51bea30dac86f5e01055af54ee8284682b06259437ce14ded25f87e16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENCODE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ip-control-log2-ratio-computation

## Summary

Computes log2(IP/control) DRIP-seq enrichment per genomic region for HOT regions versus occupancy-binned control region sets, using RNaseH-treated DRIP-seq as the control where a mock control is unavailable, to test whether HOT regions are enriched for RNA-DNA hybrids (R-loops) across cell lines and C. elegans.

## When to use

Use when testing whether a defined region set (e.g., HOT regions) is enriched for R-loops by comparing DRIP-seq IP coverage against RNaseH-treated or mock controls over occupancy-matched control regions.

## When NOT to use

- Input regions are not defined relative to TF occupancy, so HOT regions and control regions cannot be compared on equal footing via occupancy-binned percentiles
- No IP or control (mock or RNaseH-treated) DRIP-seq data are available for the cell line or organism in question, since log2(IP/control) cannot be computed
- The goal is to characterize ChIP-seq artifacts via knocked-out TF ChIP-seq rather than to quantify R-loop enrichment

## Inputs

- DRIP-seq IP raw sequencing datasets (accessions from Supplementary Table S1) for analyzed cell lines and C. elegans
- Control datasets: mock DRIP-seq controls, or RNaseH-treated DRIP-seq controls where mock controls are unavailable
- HOT region coordinate sets (BED-format genomic intervals)
- Occupancy-binned control region sets (TF occupancy percentile bins)
- Reference genomes for each analyzed species/cell line

## Outputs

- Normalized per-base coverage tracks for IP and control DRIP-seq libraries
- Per-region log2(IP/control) enrichment values for HOT region sets
- Per-region log2(IP/control) enrichment values for occupancy-binned control region sets
- Aggregated enrichment summary table per region set, per cell line, and for C. elegans

## How to apply

Retrieve the DRIP-seq IP and control (RNaseH-treated or mock) raw datasets for each analyzed cell line and for C. elegans using the accessions listed in Supplementary Table S1 (ENCODE data). Align DRIP-seq reads to the appropriate reference genome and generate normalized per-base coverage tracks for IP and control libraries. Compute log2(IP/control) enrichment per region over the HOT region coordinate sets, substituting the RNaseH-treated DRIP-seq library as the control in cell lines lacking a mock control. Compute the same log2(IP/control) enrichment over occupancy-binned control region sets (TF occupancy percentiles) so HOT and control regions are compared on equal footing. Aggregate enrichment values per region set, per cell line, and for C. elegans into a tabular summary; the R-loop enrichment on HOT regions is interpreted alongside the paper's discriminatory features (GC/CpG-rich k-mers, R-loops, G-quadruplex DNA).

## Related tools

- **ENCODE** (Source of DRIP-seq IP and control raw datasets for the analyzed cell lines (accessions listed in Supplementary Table S1))

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- R-loop enrichment (positive log2(IP/control) aggregated over HOT regions) is observed in every analyzed cell line, matching the paper's reported discriminatory feature of HOT regions
- Enrichment is also observed for the C. elegans HOT region set after computing the same log2(IP/control) metric
- HOT region enrichment is evaluated against control region sets binned by TF occupancy percentiles, confirming the comparison is on equal footing
- In cell lines lacking a mock control, the RNaseH-treated DRIP-seq track is used as the control and enrichment values remain qualitatively consistent
- Output summary table contains one enrichment entry per region set per cell line plus C. elegans with no missing region-set/cell-line combinations

## Limitations

- Cell lines lacking a mock control require substituting RNaseH-treated DRIP-seq as the control, which may not be strictly equivalent
- Enrichment of R-loops on HOT regions is interpreted as a discriminatory feature potentially associated with artificial ChIP-seq enrichment (GC/CpG-rich k-mers, R-loops, G-quadruplex DNA), so DRIP-seq enrichment alone does not prove biological TF co-binding

## Evidence

- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [readme] The manuscript "HOT or not: examining the basis of high-occupancy target regions" is published in [NAR](http://dx.doi.org/10.1093/nar/gkz460): "The manuscript "HOT or not: examining the basis of high-occupancy target regions" is published in [NAR](http://dx.doi.org/10.1093/nar/gkz460)"
