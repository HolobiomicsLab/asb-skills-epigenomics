---
name: genomic-region-set-enrichment-analysis
description: Use when testing whether a genomic feature (R-loops, G-quadruplexes,
  GC/CpG content) is enriched on HOT region coordinates relative to control regions
  binned by TF occupancy percentiles.
when_to_use_negative:
- The question is about TF binding specificity or motif presence rather than enrichment
  of a physical genomic feature over region sets
- No matched control library (mock or RNaseH-treated) exists for the signal dataset,
  making log2(IP/control) enrichment ill-defined
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: ENCODE
  role: Source of DRIP-seq and ChIP-seq datasets and region coordinate sets used for
    the enrichment comparison
- name: HOT-region-main.R
  role: Repository script used for the HOT region analysis, including region-set enrichment
    computations
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:4c30c5487f1ed06c938feed615c44735fd15075ad2503e45a05d842e96f70ec5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - ENCODE
  - HOT-region-main.R
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genomic-region-set-enrichment-analysis

## Summary

Computes log2(IP/control) enrichment of genomic signals (e.g. DRIP-seq R-loop data) over defined region sets (HOT regions vs. occupancy-binned controls) to test whether a discriminatory feature is enriched on HOT regions across cell lines and species.

## When to use

Use when testing whether a genomic feature (R-loops, G-quadruplexes, GC/CpG content) is enriched on HOT region coordinates relative to control regions binned by TF occupancy percentiles.

## When NOT to use

- The question is about TF binding specificity or motif presence rather than enrichment of a physical genomic feature over region sets
- No matched control library (mock or RNaseH-treated) exists for the signal dataset, making log2(IP/control) enrichment ill-defined

## Inputs

- DRIP-seq IP raw reads (FASTQ) per cell line and C. elegans, accessions from Supplementary Table S1
- Control DRIP-seq libraries (mock or RNaseH-treated) per sample
- HOT region coordinate sets (BED)
- Occupancy-binned control region sets matched by TF occupancy percentiles (BED)

## Outputs

- Normalized per-base coverage tracks for IP and control libraries (e.g. bigWig)
- Per-region log2(IP/control) enrichment values for HOT and control region sets
- Tabular summary of aggregate enrichment per region set, per cell line, and for C. elegans

## How to apply

Retrieve the signal datasets (e.g. DRIP-seq IP and control libraries from Supplementary Table S1 accessions) for each analyzed cell line and for C. elegans. Align reads to the appropriate reference genome and generate normalized per-base coverage tracks for IP and control (mock or RNaseH-treated DRIP-seq) libraries. Compute log2(IP/control) enrichment per genomic region over the HOT region coordinate sets, substituting RNaseH-treated DRIP-seq as the control when a mock control is unavailable. Compute the same log2(IP/control) metric over occupancy-binned control region sets so HOT and control regions are compared on equal footing. Aggregate enrichment values per region set, per cell line, and per species into a tabular summary. The rationale, per the article, is that R-loop enrichment is one of the discriminatory features of HOT regions, which are likely ChIP-seq artifacts.

## Related tools

- **ENCODE** (Source of DRIP-seq and ChIP-seq datasets and region coordinate sets used for the enrichment comparison)
- **HOT-region-main.R** (Repository script used for the HOT region analysis, including region-set enrichment computations) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- R-loop enrichment (positive log2(IP/control)) over HOT regions is observed in every analyzed cell line
- R-loop enrichment over HOT regions is also observed in C. elegans
- HOT regions show higher aggregate enrichment than occupancy-binned control region sets compared on equal footing
- RNaseH-treated DRIP-seq used as control collapses the enrichment signal in cell lines lacking mock controls, validating specificity to RNA-DNA hybrids

## Limitations

- HOT regions are likely ChIP-seq artifacts ('hyper-ChIPable' regions), so enrichment may reflect assay bias rather than true biological binding
- Lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP
- Enrichment comparisons depend on control regions being binned by TF occupancy percentiles; unmatched region sets bias the comparison

## Evidence

- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [abstract] Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions.: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
