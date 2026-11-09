---
name: enhancer-promoter-anchor-fraction-computation
description: Use when you have a TAD boundary file and ChIP-seq narrowPeak or enhancer/promoter
  interval files and need per-factor fractions of boundaries anchored by regulatory
  signals.
when_to_use_negative:
- Input is raw Hi-C contact matrix rather than an already-called TAD boundary file
  — TOAST (toast.py) must first be run to produce boundaries
- You need enhancer–promoter interaction prediction itself rather than boundary anchoring
  statistics — use EPIANN model training, not this counting step
- Cell line is not GM12878 and no matched ENCODE/EPIANN regulatory data exist for
  that line, since the ratio values are validated only against GM12878 signals
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: Provides the TAD boundaries and includes evaluate-achor-count.py which computes
    the anchor count for ChIP-seq factors at those boundaries
  repo: https://github.com/ghaiyan/TOAST
- name: EPIANN
  role: Source of the GM12878 EPI dataset supplying enhancer and promoter interval
    coordinates used as two of the ten anchoring signals
  repo: https://github.com/wgmao/EPIANN
- name: HDBSCAN
  role: 'Upstream: assigns labels to genomic bins during TOAST TAD calling, whose
    boundaries are then assessed for anchoring'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:39a7bbfac8f082ab05316d494b32cd74ecd7ac9f1cfdd433a0fc66d5aa07de48
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# enhancer-promoter-anchor-fraction-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the proportion (anchoring ratio) of regulatory peaks — transcription factors (CTCF, SMC3, RAD21, POLR2A), histone marks (H3K36me3, H3K9me3, H3K4me3, H3K4me1), Enhancer, and Promoter intervals — that overlap a 20 Kb window centered on each TOAST-called TAD boundary, to quantify boundary regulatory enrichment.

## When to use

Use when you have a TAD boundary file and ChIP-seq narrowPeak or enhancer/promoter interval files and need per-factor fractions of boundaries anchored by regulatory signals.

## When NOT to use

- Input is raw Hi-C contact matrix rather than an already-called TAD boundary file — TOAST (toast.py) must first be run to produce boundaries
- You need enhancer–promoter interaction prediction itself rather than boundary anchoring statistics — use EPIANN model training, not this counting step
- Cell line is not GM12878 and no matched ENCODE/EPIANN regulatory data exist for that line, since the ratio values are validated only against GM12878 signals

## Inputs

- TOAST TAD boundary file for GM12878 (start bin, start location, end bin, end location)
- ENCODE replicated ChIP-seq narrowPeak files for CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, H3K9me3 (accessions ENCFF74-9HDD, ENCFF002GST, ENCFF822QJA, ENCFF775OOS, ENCFF480KNX, ENCFF537KDM, ENCFF174RRQ, ENCFF218YZR)
- Enhancer and Promoter interval files from the EPIANN GM12878 EPI dataset (hg19)
- H3K4me1 ChIP-seq peaks

## Outputs

- anchor_ratios.csv with ten factor-ratio pairs (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter)
- Per-chromosome anchoring counts/ratios per factor

## How to apply

Load the TOAST-called TAD boundary file (start bin, start location, end bin, end location) for the GM12878 cell line. Load replicated ENCODE ChIP-seq narrowPeak files for CTCF, POLR2A, RAD21, SMC3, H3K4me3, H3K36me3, and H3K9me3, plus H3K4me1 and the Enhancer/Promoter intervals from the EPIANN GM12878 EPI dataset. For each of the ten regulatory signals, count peaks overlapping a 20 Kb window centered on each TAD boundary, divide by the total number of boundaries to obtain the anchoring ratio per factor, average the per-chromosome ratios, and write the ten factor-ratio pairs to anchor_ratios.csv. The rationale is that authentic TAD boundaries should be enriched for insulator and cohesin factors (CTCF highest at 0.66), so ratio orderings serve as biological validation of TAD calls.

## Related tools

- **TOAST** (Provides the TAD boundaries and includes evaluate-achor-count.py which computes the anchor count for ChIP-seq factors at those boundaries) — https://github.com/ghaiyan/TOAST
- **EPIANN** (Source of the GM12878 EPI dataset supplying enhancer and promoter interval coordinates used as two of the ten anchoring signals) — https://github.com/wgmao/EPIANN
- **HDBSCAN** (Upstream: assigns labels to genomic bins during TOAST TAD calling, whose boundaries are then assessed for anchoring)

## Examples

```
python src/evaluate-achor-count.py --tad boundaries.tsv --peaks CTCF.bed,SMC3.bed,RAD21.bed,POLR2A.bed --epi GM12878/ --window 20000 --out anchor_ratios.csv
```

## Evaluation signals

- anchor_ratios.csv exists and contains all ten factors (CTCF, SMC3, RAD21, POLR2A, H3K36me3, H3K9me3, H3K4me3, H3K4me1, Enhancer, Promoter)
- Each computed ratio lies in [0, 1]
- Averaged ratios approximate the reported values 0.66 (CTCF), 0.47 (SMC3), 0.54 (RAD21), 0.27 (POLR2A), 0.24 (H3K36me3), 0.12 (H3K9me3), 0.32 (H3K4me3), 0.41 (H3K4me1), 0.26 (Enhancer), 0.13 (Promoter)
- CTCF anchoring ratio is the highest of all factors, consistent with insulator enrichment at TAD boundaries

## Limitations

- Anchoring is defined by a fixed 20 Kb window centered on each boundary; a different window size changes the ratios
- Ratios are averaged per chromosome then across chromosomes, so chromosomes with few boundaries contribute equally to the average
- TOAST requires contiguous regions exceeding 200 Kb to be identified as TADs, so very short domains are excluded from boundary anchoring analysis
- A TAD identified as valid must have at least two edges — fragmented or singleton bins do not yield boundaries for anchoring

## Evidence

- [methods] For each regulatory signal, count peaks/anchors overlapping a 20 Kb window centered on each TAD boundary.: [evidence span withheld — anchor status: not_found]
