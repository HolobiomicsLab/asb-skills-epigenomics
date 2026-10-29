---
name: tad-boundary-regulatory-annotation
description: Use when you have TAD boundaries (e.g. TOAST-called boundary files for
  GM12878) plus ENCODE ChIP-seq narrowPeak files or enhancer/promoter intervals and
  need to compute per-factor boundary anchoring ratios.
when_to_use_negative:
- You need to identify TADs de novo from a Hi-C contact matrix — use the TOAST GAE+HDBSCAN
  pipeline instead; this skill only annotates already-called boundaries
- Your regulatory data are not genome-interval-based (e.g. gene expression matrices)
  — the 20 Kb overlap counting requires narrowPeak-style genomic peaks or enhancer/promoter
  coordinates
- Working in a cell line without matching ENCODE ChIP-seq replicates and EPIANN enhancer/promoter
  data (the workflow is GM12878-specific)
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3678
tools:
- name: TOAST
  role: Provides the TAD boundaries to annotate and ships evaluate-achor-count.py
    to compute ChIP-seq anchor counts at boundaries
  repo: https://github.com/ghaiyan/TOAST
- name: EPIANN
  role: Source of the GM12878 enhancer/promoter interval dataset used for the Enhancer
    and Promoter anchoring ratios
  repo: https://github.com/wgmao/EPIANN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:f3d520f606d8ff707daebc76c21288d304fd131d6aaf18b4675c44f4adb6dc50
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tad-boundary-regulatory-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the proportion of TAD boundaries 'anchored' by regulatory signals by counting ChIP-seq peak and enhancer/promoter overlaps within a 20 Kb window centered on each boundary, then averaging per-chromosome anchoring ratios across ten factors (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter).

## When to use

Use when you have TAD boundaries (e.g. TOAST-called boundary files for GM12878) plus ENCODE ChIP-seq narrowPeak files or enhancer/promoter intervals and need to compute per-factor boundary anchoring ratios.

## When NOT to use

- You need to identify TADs de novo from a Hi-C contact matrix — use the TOAST GAE+HDBSCAN pipeline instead; this skill only annotates already-called boundaries
- Your regulatory data are not genome-interval-based (e.g. gene expression matrices) — the 20 Kb overlap counting requires narrowPeak-style genomic peaks or enhancer/promoter coordinates
- Working in a cell line without matching ENCODE ChIP-seq replicates and EPIANN enhancer/promoter data (the workflow is GM12878-specific)

## Inputs

- TOAST TAD boundary file (start bin, start location, end bin, end location) for GM12878, e.g. from the 50 Kb, 25 Kb, or 5 Kb Hi-C contact matrices
- Replicated ENCODE ChIP-seq narrowPeak files for CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, H3K9me3 (accessions ENCFF74-9HDD, ENCFF002GST, ENCFF822QJA, ENCFF775OOS, ENCFF480KNX, ENCFF537KDM, ENCFF174RRQ, ENCFF218YZR)
- Enhancer and promoter intervals from the EPIANN GM12878 EPI dataset (hg19 coordinates)
- H3K4me1 ChIP-seq intervals

## Outputs

- anchor_ratios.csv with ten factor-ratio pairs (average anchoring ratios per regulatory signal)
- Per-chromosome anchoring ratios for each of the ten regulatory signals
- Optional ChIP-seq track plots at TADs via plot_TAD_markers.py

## How to apply

Load the TOAST-called TAD boundary file (start bin, start location, end bin, end location) for each chromosome of the GM12878 cell line. Load replicated ENCODE ChIP-seq narrowPeak files for CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, and H3K9me3, plus H3K4me1 and Enhancer/Promoter intervals from the EPIANN GM12878 EPI dataset. For each of the ten regulatory signals, count peaks/anchors overlapping a 20 Kb window centered on each TAD boundary, divide anchored boundaries by total boundaries per chromosome, then average the per-chromosome ratios. Write the ten factor-ratio pairs to anchor_ratios.csv; ratios must lie in [0, 1] and should approximate the published values (CTCF 0.66, SMC3 0.47, RAD21 0.54, POLR2A 0.27, H3K36me3 0.24, H3K9me3 0.12, H3K4me3 0.32, H3K4me1 0.41, Enhancer 0.26, Promoter 0.13). The TOAST repository provides evaluate-achor-count.py for the anchor count and plot_TAD_markers.py for visualizing ChIP-seq tracks at TADs.

## Related tools

- **TOAST** (Provides the TAD boundaries to annotate and ships evaluate-achor-count.py to compute ChIP-seq anchor counts at boundaries) — https://github.com/ghaiyan/TOAST
- **EPIANN** (Source of the GM12878 enhancer/promoter interval dataset used for the Enhancer and Promoter anchoring ratios) — https://github.com/wgmao/EPIANN

## Examples

```
cd TOAST/src && python evaluate-achor-count.py
```

## Evaluation signals

- anchor_ratios.csv exists and contains all ten factors (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter)
- Each computed average anchoring ratio lies in [0, 1]
- Computed ratios approximate the reported values 0.66 (CTCF), 0.47 (SMC3), 0.54 (RAD21), 0.27 (POLR2A), 0.24 (H3K36me3), 0.12 (H3K9me3), 0.32 (H3K4me3), 0.41 (H3K4me1), 0.26 (Enhancer), 0.13 (Promoter)
- CTCF shows the highest anchoring ratio and H3K9me3 the lowest, consistent with CTCF enrichment and repressive-mark depletion at TAD boundaries

## Limitations

- Ratios are anchored to a 20 Kb window centered on boundaries; window size changes the result and is not a free parameter to tune toward published values
- Validation values were produced for GM12878 with specific ENCODE replicate and EPIANN datasets; other cell lines or data versions will not reproduce them exactly
- TAD boundaries themselves depend on the calling algorithm and resolution (TOAST requires contiguous regions exceeding 200 Kb to be identified as TADs), so upstream TAD-calling differences propagate into the ratios

## Evidence
