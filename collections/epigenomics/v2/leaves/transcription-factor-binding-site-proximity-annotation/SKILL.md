---
name: transcription-factor-binding-site-proximity-annotation
description: Use when ChIP-seq peak coordinate files (e.g., MACS2 peak lists from
  GSE136050 H-NS and GSE136191 LuxR datasets) must be related by interval distance
  — e.g., counting how many of a structural peak class (H-NS doublets) lie within
  1 kb of a TF peak.
when_to_use_negative:
- When the goal is simple peak-overlap Venn counting of two peak sets (≤500 bp ORF-proximal
  overlap of 88 peaks) rather than doublet-structure-plus-proximity annotation — use
  findOverlaps.pl-style set overlap instead.
- When peaks lack height/trough information, since doublet detection requires the
  trough (≤80% of the lower peak height) criterion and cannot be computed from coordinates
  alone.
- When annotating peaks to gene ORFs rather than to another TF's binding sites — getPeakAnno.pl/getGenePeakAnno2.pl
  gene annotation is the wrong proxy for TF-to-TF proximity.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Perl
  role: Implements the doublet-detection scan and nearest-LuxR-peak interval distance
    search via the deposited scripts (findClosest2.pl, FileUtil.pm, getPeakAnno.pl,
    getGenePeakAnno2.pl)
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Browser session for visual inspection of H-NS doublet and LuxR peak genomic
    locations
- name: UCSC Genome Browser
  role: Supplementary genome-browser alternative for visualizing peak/doublet coordinates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:b039b5fa902334bf2d84fece268b182eda0d0c10c8381969c3448c13dbad5950
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  repo_url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - JBrowse
  - Perl
  - UCSC Genome Browser
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# transcription-factor-binding-site-proximity-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for annotating transcription factor (LuxR) ChIP-seq peaks by their genomic proximity to architecturally distinctive features of a second ChIP-seq dataset (H-NS doublet peaks), computed with interval-distance logic in Perl over MACS2-derived peak coordinate tables.

## When to use

Use when ChIP-seq peak coordinate files (e.g., MACS2 peak lists from GSE136050 H-NS and GSE136191 LuxR datasets) must be related by interval distance — e.g., counting how many of a structural peak class (H-NS doublets) lie within 1 kb of a TF peak.

## When NOT to use

- When the goal is simple peak-overlap Venn counting of two peak sets (≤500 bp ORF-proximal overlap of 88 peaks) rather than doublet-structure-plus-proximity annotation — use findOverlaps.pl-style set overlap instead.
- When peaks lack height/trough information, since doublet detection requires the trough (≤80% of the lower peak height) criterion and cannot be computed from coordinates alone.
- When annotating peaks to gene ORFs rather than to another TF's binding sites — getPeakAnno.pl/getGenePeakAnno2.pl gene annotation is the wrong proxy for TF-to-TF proximity.

## Inputs

- H-NS ChIP-seq MACS2 peak coordinate file (GEO accession GSE136050)
- LuxR ChIP-seq MACS2 peak coordinate file (GEO accession GSE136191)
- V. harveyi genomic coordinates for interval distance calculations

## Outputs

- hns_doublets.tsv: table of all detected H-NS doublets, each flagged for presence/absence of a LuxR peak within 1 kb
- Tabulated counts: 52 H-NS doublets genome-wide, 32 LuxR-proximal doublets

## How to apply

Retrieve the authors' Perl peak-processing scripts from the github.com/Juliacvk/LuxR_H-NS repository (notably findClosest2.pl, which produces a table relating luxR peaks to H-NS double peaks, plus FileUtil.pm, findOverlaps.pl, getPeakAnno.pl, and getGenePeakAnno2.pl). First detect the structural feature class in the H-NS peak list: doublets are two similarly sized H-NS peaks within 2 kb of each other whose intervening trough drops to ≤80% of the height of the lower peak. Then, for each detected H-NS doublet, use genomic interval distance calculations to find the nearest LuxR peak within 1 kb. Tabulate all doublets, flagging which have a LuxR peak within 1 kb, and write the results (e.g., hns_doublets.tsv). Validate by confirming counts against reported values: 52 H-NS doublets genome-wide, 32 LuxR-proximal, from 506 LuxR and 302 H-NS total peaks.

## Related tools

- **Perl** (Implements the doublet-detection scan and nearest-LuxR-peak interval distance search via the deposited scripts (findClosest2.pl, FileUtil.pm, getPeakAnno.pl, getGenePeakAnno2.pl)) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Browser session for visual inspection of H-NS doublet and LuxR peak genomic locations)
- **UCSC Genome Browser** (Supplementary genome-browser alternative for visualizing peak/doublet coordinates)

## Examples

```
perl findClosest2.pl
```

## Evaluation signals

- Total doublet count from the H-NS peak scan equals 52, matching the reported genome-wide value.
- Count of doublets with a LuxR peak within 1 kb equals 32, matching the reported LuxR-proximal subset.
- Every reported doublet pair satisfies both criteria: peaks within 2 kb and intervening trough ≤80% of the lower peak height.
- Output hns_doublets.tsv contains one row per detected doublet with an explicit LuxR-proximal flag column.
- Peak input totals reconcile with the paper: 506 LuxR peaks and 302 H-NS peaks.

## Limitations

- The trough-based doublet definition depends on peak height and intervening-depth values from the ChIP-seq peak files; noisy low-signal peaks can distort the 80% trough criterion.
- The 1 kb LuxR-proximity window is a chosen cutoff; changing it changes the proximal-doublet subset count.
- Whether LuxR also functions alongside H-NS to repress gene expression is unresolved and additional experimentation is needed, so proximity annotation alone does not establish co-occupancy mechanism.
- Peak co-occupancy at the promoter level (28 promoters / 63 genes) uses different overlap criteria (≤500 bp proximal to an ORF) than the doublet-proximity analysis; the two counts are not interchangeable.

## Evidence
