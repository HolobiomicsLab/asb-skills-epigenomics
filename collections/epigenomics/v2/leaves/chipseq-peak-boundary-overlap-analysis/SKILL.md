---
name: chipseq-peak-boundary-overlap-analysis
description: Use when you have TAD boundary coordinates (e.g., from TOAST on GM12878
  Hi-C) plus ENCODE narrowPeak ChIP-seq files or enhancer/promoter interval sets,
  and need to quantify the fraction of boundaries anchored by each regulatory signal.
when_to_use_negative:
- The goal is to identify TAD boundaries from Hi-C contact matrices rather than to
  validate existing boundaries — use the TOAST GAE+HDBSCAN pipeline itself instead
- Only raw ChIP-seq reads are available without called narrowPeak peaks or enhancer/promoter
  interval annotations — peak calling and E/P annotation must precede this skill
- The TAD caller output lacks genomic start/end locations (only bin indices and no
  resolution metadata), since the 20 Kb overlap window cannot be applied reliably
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: Produces the TAD boundary coordinates whose anchoring ratios are computed;
    repository includes evaluate-achor-count.py and plot_TAD_markers.py for anchor
    counting and ChIP-seq track visualization
  repo: https://github.com/ghaiyan/TOAST
- name: EPIANN
  role: Source of the GM12878 enhancer/promoter (EPI) interval dataset used as two
    of the ten anchored signals
  repo: github.com/wgmao/EPIANN
- name: HDBSCAN
  role: Clustering step inside TOAST that assigns bin labels from GAE embeddings,
    from which the validated TAD boundaries are delineated
  repo: https://github.com/ghaiyan/TOAST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:11ce9bb7d58dbab62fd88c81e5dfd553dd7dbc51a74c0049a076a1dbe1c7abe2
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
  - HDBSCAN
  - TOAST
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

# chipseq-peak-boundary-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes anchoring ratios of ChIP-seq peaks (transcription factors, histone marks, enhancers, promoters) at TAD boundaries by counting peaks overlapping a 20 Kb window centered on each boundary. Used in the TOAST paper to validate that algorithm-called TAD boundaries coincide with known regulatory landmarks such as CTCF, SMC3, and RAD21.

## When to use

Use when you have TAD boundary coordinates (e.g., from TOAST on GM12878 Hi-C) plus ENCODE narrowPeak ChIP-seq files or enhancer/promoter interval sets, and need to quantify the fraction of boundaries anchored by each regulatory signal.

## When NOT to use

- The goal is to identify TAD boundaries from Hi-C contact matrices rather than to validate existing boundaries — use the TOAST GAE+HDBSCAN pipeline itself instead
- Only raw ChIP-seq reads are available without called narrowPeak peaks or enhancer/promoter interval annotations — peak calling and E/P annotation must precede this skill
- The TAD caller output lacks genomic start/end locations (only bin indices and no resolution metadata), since the 20 Kb overlap window cannot be applied reliably

## Inputs

- TOAST TAD boundary file (start bin, start location, end bin, end location) for GM12878
- ENCODE GM12878 ChIP-seq replicated narrowPeak files (CTCF, SMC3, RAD21, POLR2A, H3K4me3, H3K36me3, H3K9me3, H3K4me1)
- Enhancer and Promoter intervals from the EPIANN GM12878 EPI dataset (hg19)
- 20 Kb window size parameter centered on each TAD boundary

## Outputs

- anchor_ratios.csv with ten factor-ratio pairs (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter)
- Per-chromosome anchoring ratios averaged into the final values

## How to apply

Load the TAD boundary file for the cell line (columns: start bin, start location, end bin, end location) produced by TOAST (GAE + HDBSCAN pipeline on 50/25/5 Kb GM12878 Hi-C matrices). Load replicated ENCODE GM12878 ChIP-seq narrowPeak files for CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, H3K9me3, and H3K4me1, plus Enhancer and Promoter intervals from the EPIANN GM12878 EPI dataset. For each of the ten signals, count how many TAD boundaries have a peak/interval overlapping a 20 Kb window centered on the boundary, then divide by the total number of boundaries to get the anchoring ratio per factor. Average per-chromosome ratios and write the ten factor-ratio pairs to anchor_ratios.csv. The TOAST repository provides evaluate-achor-count.py for computing anchor counts and plot_TAD_markers.py for visualizing ChIP-seq tracks at TADs.

## Related tools

- **TOAST** (Produces the TAD boundary coordinates whose anchoring ratios are computed; repository includes evaluate-achor-count.py and plot_TAD_markers.py for anchor counting and ChIP-seq track visualization) — https://github.com/ghaiyan/TOAST
- **EPIANN** (Source of the GM12878 enhancer/promoter (EPI) interval dataset used as two of the ten anchored signals) — github.com/wgmao/EPIANN
- **HDBSCAN** (Clustering step inside TOAST that assigns bin labels from GAE embeddings, from which the validated TAD boundaries are delineated) — https://github.com/ghaiyan/TOAST

## Examples

```
python evaluate-achor-count.py --tads tad_boundaries.txt --chipsep peaks/ --window 20000 --out anchor_ratios.csv
```

## Evaluation signals

- anchor_ratios.csv exists and contains all ten factors (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter)
- Each computed ratio lies in [0, 1]
- Average ratios approximate the reported values: 0.66 (CTCF), 0.47 (SMC3), 0.54 (RAD21), 0.27 (POLR2A), 0.24 (H3K36me3), 0.12 (H3K9me3), 0.32 (H3K4me3), 0.41 (H3K4me1), 0.26 (Enhancer), 0.13 (Promoter)
- CTCF shows the highest anchoring ratio and H3K9me3/promoters among the lowest, consistent with the published ordering
- The 20 Kb window is centered on each boundary before overlap counting (i.e., boundary midpoint ± 10 Kb)

## Limitations

- Ratios depend on the TAD-calling algorithm; values are averaged across boundaries from different TAD recognition algorithms and may differ for a single caller
- Enhancer/promoter anchors rely on the EPIANN GM12878 EPI dataset (hg19, TargetFinder-derived), so genome build and enhancer definition mismatch can shift ratios
- Only contiguous regions longer than 200 Kb are identified as TADs by TOAST, so boundaries from shorter called regions are excluded from the anchor analysis

## Evidence

- [other] For each regulatory signal, count peaks/anchors overlapping a 20 Kb window centered on each TAD boundary.: [evidence span withheld — anchor status: not_found]
- [abstract] Average anchoring ratios abstract quote: "obtained an average ratio of anchoring CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, and Promoters of 0.66, 0.47, 0.54, 0.27, 0.24, 0.12, 0.32, 0.41, 0.26, and 0.13,"
- [readme] README anchor count script quote: "(5)evaluate-achor-count.py:compute the anchor count for Chip-seq factors."
- [methods] EPIANN dataset download quote: "the EPI dataset of the GM12878 cell line from https://github.com/wgmao/EPIANN/tree/master/GM12878 was downloaded"
- [methods] 200 Kb TAD length filter quote: "If the length of a contiguous region exceeds 200 Kb, it can be identified as a TAD"
