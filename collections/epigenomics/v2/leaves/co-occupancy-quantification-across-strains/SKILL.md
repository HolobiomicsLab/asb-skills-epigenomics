---
name: co-occupancy-quantification-across-strains
description: Use when you have MACS2 ChIP-seq peak interval files for two DNA-binding
  proteins in two or more strain backgrounds and need per-strain counts of co-occupied,
  ORF-proximal regions.
when_to_use_negative:
- Input is raw ChIP-seq reads or BED-formatted read alignments rather than MACS2-called
  peak intervals; peak calling must happen first
- The question is expression co-regulation rather than physical co-occupancy — use
  RNA-seq co-regulation analysis (28 co-regulated promoters driving 63 genes) instead
  of peak overlap counting
- Peaks of interest cannot be linked to an annotated ORF within 500 bp, since the
  reported overlap counts are defined only for ORF-proximal peaks
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Perl
  role: Host language for the deposited peak-overlap scripts (findOverlaps.pl, getPeakAnno.pl,
    getGenePeakAnno2.pl, getGenePeakAnno2.pl, findClosest2.pl, FileUtil.pm) that load
    peaks and annotations, intersect interval sets, and annotate overlaps relative
    to ORFs
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Used to browse the uploaded sequences and visually inspect LuxR/H-NS co-occupied
    peak regions alongside the ORF annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:69b0489665fde9815b23a5c5e8da86427ab98a8f8f69f7a53ffa820a997e6192
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# co-occupancy-quantification-across-strains

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how many LuxR and H-NS MACS2 ChIP-seq peaks overlap (co-occupancy) in wild-type versus ΔluxR V. harveyi, using distance-based interval intersection restricted to peaks proximal to an ORF (≤500 bp). It turns two peak sets into a per-strain count of co-occupied loci, reproducing the paper's 68 (WT) and 88 (ΔluxR) figures.

## When to use

Use when you have MACS2 ChIP-seq peak interval files for two DNA-binding proteins in two or more strain backgrounds and need per-strain counts of co-occupied, ORF-proximal regions.

## When NOT to use

- Input is raw ChIP-seq reads or BED-formatted read alignments rather than MACS2-called peak intervals; peak calling must happen first
- The question is expression co-regulation rather than physical co-occupancy — use RNA-seq co-regulation analysis (28 co-regulated promoters driving 63 genes) instead of peak overlap counting
- Peaks of interest cannot be linked to an annotated ORF within 500 bp, since the reported overlap counts are defined only for ORF-proximal peaks

## Inputs

- MACS2 ChIP-seq peak interval files for LuxR (wild-type and ΔluxR)
- MACS2 ChIP-seq peak interval files for H-NS (wild-type and ΔluxR)
- V. harveyi ORF annotation (multi-fasta cleaned via reformatFasta.pl and gene peak annotation via getGenePeakAnno2.pl)

## Outputs

- per-strain count of overlapping LuxR/H-NS regions (e.g. 68 in wild-type, 88 in ΔluxR)
- peak_overlap_counts.csv with per-strain overlap tallies
- overlapping peak sets usable for Venn diagrams and co-occupied promoter lists

## How to apply

Download the processed LuxR and H-NS MACS2 peak interval files (NCBI GEO GSE136191) and load them together with the V. harveyi ORF annotation using the Perl helper scripts deposited at github.com/Juliacvk/LuxR_H-NS (findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl). Run a findOverlaps-style distance-based intersection between the LuxR and H-NS peak sets separately per strain background, retaining all overlapping region pairs. Filter the retained overlaps to peaks proximal to an ORF within 500 bp — in the article, 506 LuxR peaks and 302 H-NS peaks yielded 88 overlapping ORF-proximal peaks. Tally co-occupied loci per strain (68 in wild-type, 88 in ΔluxR) and write the per-strain tallies to a CSV such as peak_overlap_counts.csv. The decision point is the ORF-proximity cutoff: applying it versus not explains the difference between the 88-peak discussion figure and the broader 68-region WT overlap count.

## Related tools

- **Perl** (Host language for the deposited peak-overlap scripts (findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, getGenePeakAnno2.pl, findClosest2.pl, FileUtil.pm) that load peaks and annotations, intersect interval sets, and annotate overlaps relative to ORFs) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Used to browse the uploaded sequences and visually inspect LuxR/H-NS co-occupied peak regions alongside the ORF annotation)

## Examples

```
git clone https://github.com/Juliacvk/LuxR_H-NS && cd LuxR_H-NS && perl findOverlaps.pl
```

## Evaluation signals

- The computed wild-type overlap count reproduces the reported 68 overlapping LuxR/H-NS regions
- After applying the ORF-proximity filter (≤500 bp), the ΔluxR overlap count reproduces the reported 88 co-occupied ORF-proximal peaks (consistent with 506 LuxR peaks and 302 H-NS peaks total)
- Output CSV (peak_overlap_counts.csv) contains one tally per strain background with no missing strains
- The overlap output matches the counts shown in the Venn diagram produced by findOverlaps.pl
- Every retained overlap pair satisfies the distance-to-ORF threshold of ≤500 bp as annotated by getPeakAnno.pl/getGenePeakAnno2.pl

## Limitations

- Overlap counts are contingent on the upstream MACS2 peak-calling step; different peak-calling parameters will change the 68/88 tallies
- The reported counts depend on the ≤500 bp ORF-proximity cutoff; without this filter the overlap count differs (e.g. 68 WT regions versus 88 ORF-proximal peaks), so the criteria must be stated explicitly
- Whether LuxR also functions alongside H-NS to repress gene expression is unresolved and additional experimentation is needed, so co-occupancy counts alone do not establish a repression mechanism

## Evidence
