---
name: genomic-interval-anchoring-quantification
description: Use when you have a set of called TAD boundaries (start bin, start location,
  end bin, end location) plus genomic peak/interval files (narrowPeak BED, enhancer/promoter
  BED) and need anchoring ratios per regulatory factor.
when_to_use_negative:
- You need to call or identify TAD boundaries themselves from Hi-C contact matrices
  — use the TOAST GAE+HDBSCAN pipeline, not this anchoring quantification step
- Your boundaries come from a non-Hi-C definition or your regulatory data lacks genomic
  coordinates (e.g., only expression matrices) — interval overlap is undefined
- You need cell-type-specific enhancer/promoter calls for a cell line other than those
  available in the EPIANN data (GM12878, HUVEC, HeLa-S3, IMR90, K562, NHEK)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3169
tools:
- name: TOAST
  role: Produces the TAD boundary calls whose anchoring ratios are quantified; repository
    includes evaluate-achor-count.py for ChIP-seq anchor counting
  repo: https://github.com/ghaiyan/TOAST
- name: EPIANN
  role: Source of GM12878 enhancer/promoter interval datasets used as anchoring signals
  repo: github.com/wgmao/EPIANN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:2e4e2b3b156346f668cb1be9491fb5dd950c5a1ea312b664071296ebf2516048
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - EPIANN
  - TOAST
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-interval-anchoring-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the fraction of TAD boundaries anchored by regulatory signals (ChIP-seq peaks for CTCF, SMC3, RAD21, POLR2A, histone marks, and Enhancer/Promoter intervals) by counting overlaps within a 20 Kb window centered on each boundary and averaging per-chromosome ratios.

## When to use

Use when you have a set of called TAD boundaries (start bin, start location, end bin, end location) plus genomic peak/interval files (narrowPeak BED, enhancer/promoter BED) and need anchoring ratios per regulatory factor.

## When NOT to use

- You need to call or identify TAD boundaries themselves from Hi-C contact matrices — use the TOAST GAE+HDBSCAN pipeline, not this anchoring quantification step
- Your boundaries come from a non-Hi-C definition or your regulatory data lacks genomic coordinates (e.g., only expression matrices) — interval overlap is undefined
- You need cell-type-specific enhancer/promoter calls for a cell line other than those available in the EPIANN data (GM12878, HUVEC, HeLa-S3, IMR90, K562, NHEK)

## Inputs

- TOAST-called TAD boundary file (start bin, start location, end bin, end location) for GM12878
- ENCODE GM12878 ChIP-seq narrowPeak files for CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, H3K9me3 (accessions ENCFF74-9HDD, ENCFF002GST, ENCFF822QJA, ENCFF775OOS, ENCFF480KNX, ENCFF537KDM, ENCFF174RRQ, ENCFF218YZR)
- H3K4me1 ChIP-seq intervals
- Enhancer and Promoter intervals from the EPIANN GM12878 EPI dataset (github.com/wgmao/EPIANN/tree/master/GM12878)

## Outputs

- anchor_ratios.csv with ten factor-ratio pairs (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter)
- Per-chromosome anchoring ratios per signal
- Optional ChIP-seq track plots at TADs (plot_TAD_markers.py)

## How to apply

Load the TOAST-called TAD boundary file for the GM12878 cell line with columns (start bin, start location, end bin, end location). Load replicated ENCODE ChIP-seq narrowPeak files (CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, H3K9me3, H3K4me1) and Enhancer/Promoter intervals from the EPIANN GM12878 EPI dataset. For each of the ten regulatory signals, count peaks/anchors overlapping a 20 Kb window centered on each TAD boundary, then divide anchored boundaries by total boundaries to get the anchoring ratio. Average per-chromosome ratios and write the ten factor-ratio pairs to anchor_ratios.csv. The TOAST repository provides evaluate-achor-count.py to compute anchor counts for ChIP-seq factors and plot_TAD_markers.py to visualize ChIP-seq tracks at TADs.

## Related tools

- **TOAST** (Produces the TAD boundary calls whose anchoring ratios are quantified; repository includes evaluate-achor-count.py for ChIP-seq anchor counting) — https://github.com/ghaiyan/TOAST
- **EPIANN** (Source of GM12878 enhancer/promoter interval datasets used as anchoring signals) — github.com/wgmao/EPIANN

## Examples

```
python src/evaluate-achor-count.py  # computes ChIP-seq anchor counts at TOAST TAD boundaries; then aggregate to anchor_ratios.csv with ratios CTCF=0.66, SMC3=0.47, RAD21=0.54, ...
```

## Evaluation signals

- Computed average ratios approximate the reported values: CTCF 0.66, SMC3 0.47, RAD21 0.54, POLR2A 0.27, H3K36me3 0.24, H3K9me3 0.12, H3K4me3 0.32, H3K4me1 0.41, Enhancer 0.26, Promoter 0.13
- anchor_ratios.csv exists and contains exactly all ten factors
- Every ratio lies within [0, 1]
- Each factor's anchoring ratio equals anchored boundaries divided by total boundaries per chromosome before averaging
- CTCF shows the highest anchoring ratio and H3K9me3 the lowest, consistent with the reported ordering

## Limitations

- Results depend on the TAD-calling algorithm; ratios are reported as averages across different TAD recognition algorithms, so boundaries from a different caller will shift values
- Enhancer/Promoter intervals are specific to the EPIANN GM12878 EPI dataset (hg19 coordinates); other cell lines or assemblies require new interval sources
- The 20 Kb window centered on each boundary is a fixed choice; anchoring ratios are sensitive to this window size

## Evidence
