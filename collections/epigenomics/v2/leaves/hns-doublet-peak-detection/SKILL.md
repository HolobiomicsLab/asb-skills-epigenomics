---
name: hns-doublet-peak-detection
description: Use when ChIP-seq peak coordinates need to be scanned for paired peaks
  (doublets) with an intervening trough, or when relating doublet peaks to a second
  transcription factor's ChIP-seq binding sites within a fixed genomic distance.
when_to_use_negative:
- The goal is general peak overlap or Venn-style intersection without the doublet
  (paired-peak + trough) structure — use findOverlaps.pl-style overlap analysis instead
- Input is raw ChIP-seq reads or FASTQ rather than called Macs2 peak coordinates —
  peak calling must be performed first
- Peak files come from an organism or peak-caller with different coordinate formats
  that FileUtil.pm cannot load
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Perl
  role: Scripts (findClosest2.pl, getPeakAnno.pl, getGenePeakAnno2.pl, FileUtil.pm)
    for peak loading, annotation, doublet detection, and relating LuxR peaks to H-NS
    doublet peaks
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:2de9bafc022facd4acb12f038e7c35c2a851880e0b038c931070e7ae9dd3e454
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
  - Perl
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hns-doublet-peak-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects 'H-NS doublet' peak pairs in V. harveyi H-NS ChIP-seq data (Macs2 peaks from GSE136050) using the authors' Perl scripts, then links each doublet to nearby LuxR ChIP-seq peaks (GSE136191) to quantify LuxR association with H-NS bridging sites.

## When to use

Use when ChIP-seq peak coordinates need to be scanned for paired peaks (doublets) with an intervening trough, or when relating doublet peaks to a second transcription factor's ChIP-seq binding sites within a fixed genomic distance.

## When NOT to use

- The goal is general peak overlap or Venn-style intersection without the doublet (paired-peak + trough) structure — use findOverlaps.pl-style overlap analysis instead
- Input is raw ChIP-seq reads or FASTQ rather than called Macs2 peak coordinates — peak calling must be performed first
- Peak files come from an organism or peak-caller with different coordinate formats that FileUtil.pm cannot load

## Inputs

- H-NS ChIP-seq Macs2 peak coordinate file (V. harveyi genome, GEO GSE136050)
- LuxR ChIP-seq Macs2 peak coordinate file (GEO GSE136191)
- Perl scripts from github.com/Juliacvk/LuxR_H-NS (FileUtil.pm, findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl)

## Outputs

- hns_doublets.tsv tabulating all detected H-NS doublets with flags for LuxR peaks within 1 kb
- Doublet counts: 52 H-NS doublets genome-wide, 32 with a LuxR binding site within 1 kb

## How to apply

Download H-NS and LuxR ChIP-seq peak coordinate files from GEO accessions GSE136050 and GSE136191 and retrieve the Perl scripts from github.com/Juliacvk/LuxR_H-NS. Scan the H-NS peak list to detect doublets, defined as two H-NS peaks within 2 kb of each other whose intervening trough drops to ≤80% of the height of the lower peak. Use findClosest2.pl to produce a table relating LuxR peaks to H-NS double peaks, searching for the nearest LuxR peak within 1 kb of each doublet via genomic interval distance calculations. Tabulate all doublets and flag which have a LuxR peak within 1 kb, writing the results to hns_doublets.tsv. Validate by confirming 52 doublets genome-wide and 32 LuxR-proximal doublets, matching the reported values.

## Related tools

- **Perl** (Scripts (findClosest2.pl, getPeakAnno.pl, getGenePeakAnno2.pl, FileUtil.pm) for peak loading, annotation, doublet detection, and relating LuxR peaks to H-NS doublet peaks) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
perl findClosest2.pl
```

## Evaluation signals

- Total doublet count equals 52, matching the paper's reported value
- LuxR-proximal doublet subset equals 32 (LuxR peak within 1 kb of the doublet)
- Each detected doublet satisfies both criteria: peaks within 2 kb AND intervening trough ≤80% of the height of the lower peak
- Output file hns_doublets.tsv is produced with per-doublet rows and LuxR-proximity flags
- Every LuxR-distant doublet can be verified by recomputing genomic interval distances between doublet peaks and the LuxR peak list

## Limitations

- Doublet detection is threshold-dependent (2 kb separation, trough ≤80% of the lower peak); changing cutoffs changes the 52/32 counts
- LuxR proximity is scored only within 1 kb, so associations beyond that distance are missed
- Scripts operate on Macs2-called peak coordinates from V. harveyi ChIP-seq; the trough criterion assumes peak height/trough information is available in the input peak files

## Evidence

- [other] doublets, defined as two H-NS peaks within 2 kb of each other whose intervening trough drops to ≤80% of the height of the lower peak: [evidence span withheld — anchor status: not_found]
- [other] 52 H-NS doublets genome-wide, 32 of which are located within 1 kb of a LuxR binding site: [evidence span withheld — anchor status: not_found]
