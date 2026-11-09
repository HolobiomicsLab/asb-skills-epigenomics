---
name: cross-species-hot-region-comparison
description: Use when DRIP-seq IP/control raw data (with RNaseH-treated or mock controls)
  are available for cell lines and C. elegans, and the goal is to test whether HOT
  regions are enriched for R-loops relative to occupancy-matched control regions.
when_to_use_negative:
- The goal is defining HOT regions from ChIP-seq TF occupancy itself (peak calling/occupancy
  thresholding) rather than testing R-loop enrichment on pre-existing HOT region coordinates.
- No DRIP-seq IP data and no RNaseH-treated or mock control exist for the species/cell
  line in question, making log2(IP/control) enrichment uncomputable.
- The analysis targets ChIP-seq artifact properties (e.g., false positive signals
  in knocked-out TF ChIP-seq) rather than R-loop/RNA-DNA hybrid enrichment.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: ENCODE
  role: Source of DRIP-seq IP and control datasets for analyzed cell lines
- name: HOT-or-not analysis scripts
  role: Repository scripts (Scripts/HOT-region-main.R) used for the HOT region analyses
    underlying this skill
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:af7df42ff1920486fdefc360a91cd422343f80bf32e14bdde5c4552a485dd6ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - ENCODE
  - HOT-or-not analysis scripts
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-species-hot-region-comparison

## Summary

Quantifies R-loop (RNA-DNA hybrid) enrichment on high-occupancy target (HOT) regions via DRIP-seq log2(IP/control) signal across multiple cell lines and C. elegans, comparing HOT regions against occupancy-binned control region sets on equal footing.

## When to use

Use when DRIP-seq IP/control raw data (with RNaseH-treated or mock controls) are available for cell lines and C. elegans, and the goal is to test whether HOT regions are enriched for R-loops relative to occupancy-matched control regions.

## When NOT to use

- The goal is defining HOT regions from ChIP-seq TF occupancy itself (peak calling/occupancy thresholding) rather than testing R-loop enrichment on pre-existing HOT region coordinates.
- No DRIP-seq IP data and no RNaseH-treated or mock control exist for the species/cell line in question, making log2(IP/control) enrichment uncomputable.
- The analysis targets ChIP-seq artifact properties (e.g., false positive signals in knocked-out TF ChIP-seq) rather than R-loop/RNA-DNA hybrid enrichment.

## Inputs

- DRIP-seq IP and control (RNaseH-treated or mock) raw sequencing datasets per cell line and for C. elegans, with accessions from Supplementary Table S1
- HOT region coordinate sets (BED-format genomic intervals)
- Transcription-factor occupancy-binned control region sets (percentile-binned BED intervals)
- Reference genomes for each analyzed cell line/species

## Outputs

- Normalized per-base DRIP-seq coverage tracks for IP and control libraries
- Per-region log2(IP/control) enrichment values over HOT regions and occupancy-binned control regions
- Tabular summary of aggregate enrichment per region set, per cell line, and for C. elegans

## How to apply

Retrieve DRIP-seq IP and control (RNaseH-treated or mock) raw datasets for each analyzed cell line and for C. elegans using the accessions listed in Supplementary Table S1 (data available via ENCODE). Align reads to the species-appropriate reference genome and generate normalized per-base coverage tracks for IP and control libraries. Compute log2(IP/control) enrichment per genomic region over the HOT region coordinate sets, substituting RNaseH-treated DRIP-seq as the control in cell lines lacking a mock control. Compute the same log2(IP/control) enrichment over the occupancy-binned control region sets so HOT and control regions are compared on equal footing, then aggregate enrichment values per region set, per cell line, and for C. elegans into a tabular summary. The rationale is that the paper identifies enrichment of RNA-DNA hybrids (R-loops) as a discriminatory feature of HOT regions, so reproducible enrichment across all analyzed samples validates the finding.

## Related tools

- **ENCODE** (Source of DRIP-seq IP and control datasets for analyzed cell lines)
- **HOT-or-not analysis scripts** (Repository scripts (Scripts/HOT-region-main.R) used for the HOT region analyses underlying this skill) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
Rscript Scripts/HOT-region-main.R
```

## Evaluation signals

- Aggregate log2(IP/control) DRIP-seq enrichment over HOT regions is positive in every analyzed cell line compared to occupancy-percentile-binned control region sets.
- DRIP-seq enrichment on HOT regions is also observed for C. elegans, confirming cross-species enrichment.
- RNaseH-treated DRIP-seq used as control reduces the apparent HOT-region signal relative to IP-over-input comparisons in cell lines lacking mock controls.
- HOT and control region sets are compared using the same log2(IP/control) metric over equivalently defined genomic intervals (occupancy percentile bins).
- A tabular output exists with one aggregate enrichment value per region set, per cell line, plus C. elegans.

## Limitations

- In cell lines lacking a mock control, RNaseH-treated DRIP-seq must be substituted as the control, which may reduce comparability across samples.
- A lack of signal can arise from technical conditions rather than biology (e.g., low number of reads, low library complexity, or unsuccessful IP).
- R-loop enrichment is only a discriminatory feature of HOT regions and does not establish a causal mechanism for the artificial ChIP-seq enrichment on HOT regions.

## Evidence

- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [intro] To date, there has been no cross-species comparison of HOT regions in terms of sequence features.: "To date, there has been no cross-species comparison of HOT regions in terms of sequence features."
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
