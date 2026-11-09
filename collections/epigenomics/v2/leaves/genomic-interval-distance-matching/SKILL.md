---
name: genomic-interval-distance-matching
description: Use when processed ChIP-seq peak interval files (e.g. MACS2 LuxR and
  H-NS peaks from GEO GSE136191/GSE136050) must be intersected or distance-matched,
  such as counting co-occupied LuxR/H-NS loci per strain or flagging H-NS doublets
  within 1 kb of a LuxR peak.
when_to_use_negative:
- Input is raw ChIP-seq reads rather than processed MACS2 peak interval files — peak
  calling must be done first (peaks come from GEO GSE136191/GSE136050)
- The goal is expression-level analysis (RNA-seq differential expression) rather than
  genomic interval overlap or peak-proximity assignment
- Peak coordinates cannot be tied to an ORF annotation or a defined reference genome,
  since the ≤500 bp ORF-proximity filter depends on annotated gene positions
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Perl (findOverlaps.pl)
  role: Distance-based intersection of LuxR and H-NS peak sets per strain background
    to identify overlapping peaks for Venn diagrams
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: Perl (findClosest2.pl)
  role: Produces the table relating luxR peaks to H-NS double peaks (1 kb proximity
    matching)
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: Perl (getPeakAnno.pl / getGenePeakAnno2.pl)
  role: Adds annotation information to MACS2 peaks and links peaks to genes for the
    ORF-proximity (≤500 bp) filter
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Genome browser session for inspecting the uploaded sequences and co-occupied
    regions
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:d1ae115b12eace859f7fea63304dbcee480e5a5938bcf8540375abf9aed3c65c
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
  - Perl (findClosest2.pl)
  - Perl (findOverlaps.pl)
  - Perl (getPeakAnno.pl / getGenePeakAnno2.pl)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# genomic-interval-distance-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Perl-based skill for distance-based intersection and proximity matching of MACS2 ChIP-seq peak intervals, used to quantify LuxR/H-NS co-occupied loci and to pair H-NS doublet peaks with nearby LuxR binding sites in the V. harveyi genome.

## When to use

Use when processed ChIP-seq peak interval files (e.g. MACS2 LuxR and H-NS peaks from GEO GSE136191/GSE136050) must be intersected or distance-matched, such as counting co-occupied LuxR/H-NS loci per strain or flagging H-NS doublets within 1 kb of a LuxR peak.

## When NOT to use

- Input is raw ChIP-seq reads rather than processed MACS2 peak interval files — peak calling must be done first (peaks come from GEO GSE136191/GSE136050)
- The goal is expression-level analysis (RNA-seq differential expression) rather than genomic interval overlap or peak-proximity assignment
- Peak coordinates cannot be tied to an ORF annotation or a defined reference genome, since the ≤500 bp ORF-proximity filter depends on annotated gene positions

## Inputs

- Processed MACS2 ChIP-seq peak interval files for LuxR and H-NS in wild-type and Δluxr V. harveyi (NCBI GEO GSE136191, GSE136050)
- V. harveyi ORF/genome annotation file
- Perl helper scripts from github.com/Juliacvk/LuxR_H-NS (FileUtil.pm, findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl)

## Outputs

- peak_overlap_counts.csv: per-strain tallies of overlapping LuxR/H-NS regions (68 in wild-type; 88 co-occupied loci in Δluxr)
- hns_doublets.tsv: all H-NS doublets (52 genome-wide) flagged for LuxR peak within 1 kb (32 flagged)
- Table relating luxR peaks to H-NS double peaks (findClosest2.pl output)

## How to apply

Load the processed LuxR and H-NS MACS2 peak interval files and the V. harveyi ORF annotation with the deposited Perl helpers (FileUtil.pm, getPeakAnno.pl, getGenePeakAnno2.pl). Run findOverlaps.pl as a distance-based intersection of the LuxR and H-NS peak sets separately for each strain background (wild-type and ΔluxR), retaining all overlapping region pairs, then filter to peaks proximal to an ORF (≤500 bp) and tally co-occupied loci per strain. For doublet analysis, scan the H-NS peak list for doublets defined as two H-NS peaks within 2 kb whose intervening trough drops to ≤80% of the height of the lower peak, then use findClosest2.pl to relate each luxR peak to H-NS double peaks within 1 kb. The rationale is that peak-level interval overlap is the operational definition of co-occupancy used by the paper, where LuxR and H-NS share 28 promoters/63 genes and 88 overlapping peaks lie proximal to an ORF.

## Related tools

- **Perl (findOverlaps.pl)** (Distance-based intersection of LuxR and H-NS peak sets per strain background to identify overlapping peaks for Venn diagrams) — https://github.com/Juliacvk/LuxR_H-NS
- **Perl (findClosest2.pl)** (Produces the table relating luxR peaks to H-NS double peaks (1 kb proximity matching)) — https://github.com/Juliacvk/LuxR_H-NS
- **Perl (getPeakAnno.pl / getGenePeakAnno2.pl)** (Adds annotation information to MACS2 peaks and links peaks to genes for the ORF-proximity (≤500 bp) filter) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Genome browser session for inspecting the uploaded sequences and co-occupied regions)

## Evaluation signals

- The wild-type overlap tally reproduces the reported 68 overlapping LuxR/H-NS regions, and the Δluxr background yields 88 co-occupied loci
- Overlap output matches the published figure of 88 overlapping peaks proximal to an ORF (≤500 bp) out of 506 LuxR and 302 H-NS peaks
- Doublet scan reports exactly 52 H-NS doublets genome-wide, with 32 having a LuxR peak within 1 kb
- Every retained interval pair satisfies the stated distance thresholds (doublet members within 2 kb with trough ≤80% of the lower peak; LuxR within 1 kb of the doublet; ORF proximity ≤500 bp)
- Output files (peak_overlap_counts.csv, hns_doublets.tsv) contain per-strain/per-doublet rows with coordinates, so counts can be re-derived by an independent interval-overlap tool (e.g. bedtools intersect)

## Limitations

- Counts are threshold-dependent and specific to the V. harveyi genome annotation and the LuxR/H-NS ChIP-seq datasets in GSE136191/GSE136050; changing the ≤500 bp ORF-proximity or 1 kb/2 kb distance cutoffs will change the tallies
- The README lists the scripts (FileUtil.pm, reformatFasta.pl, findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl) but provides no usage documentation, so command-line arguments must be inferred from the scripts
- The co-occupancy count is a peak-overlap proxy; the paper separately reports 28 co-regulated/co-occupied promoters driving 63 genes, which is not the same metric as the 88 ORF-proximal overlapping peaks

## Evidence
