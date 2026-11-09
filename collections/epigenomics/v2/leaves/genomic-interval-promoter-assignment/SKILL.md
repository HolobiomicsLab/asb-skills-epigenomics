---
name: genomic-interval-promoter-assignment
description: Use when you have a set of ChIP-seq peak intervals (e.g., MACs2 output
  for two regulators) plus RNA-seq differential expression calls, and you need to
  assign peaks to promoters/ORFs and count co-regulated, co-occupied loci.
when_to_use_negative:
- The question is about the regulatory mechanism at a promoter (e.g., H-NS remodeling/displacement)
  rather than assigning peaks to promoters and counting co-regulated loci
- ChIP-seq peaks have not been called or overlap sets between regulators have not
  been established (the pipeline starts from MACs2 peaks and overlapping intervals)
- RNA-seq co-regulation calls across the relevant conditions (e.g., GSE136050 population-density
  conditions) are unavailable, so promoter co-occupancy cannot be intersected with
  co-regulation
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3169
tools:
- name: Perl
  role: Runs the deposited scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl,
    findOverlaps.pl) that annotate MACs2 peaks, assign peaks to genes/ORFs, and intersect
    co-occupied promoters with co-regulated genes
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Visual inspection of the resulting loci to confirm peak-to-promoter assignments
    are consistent with the genome annotation
- name: UCSC Genome Browser
  role: Alternative genome browser for verifying peak-to-promoter assignments against
    the annotation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:879409b86111f644efe8ec3d800f91b592c80f0908926646cad0b7d8f65cc0d8
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Genomic interval promoter assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates overlapping ChIP-seq peak intervals to the nearest open reading frame (peaks within 500 bp upstream of an ORF) and intersects the resulting promoter set with transcriptomically co-regulated genes to reconstruct the set of co-regulated, co-occupied promoters — here, the 28 LuxR/H-NS promoters driving 63 genes in V. harveyi.

## When to use

Use when you have a set of ChIP-seq peak intervals (e.g., MACs2 output for two regulators) plus RNA-seq differential expression calls, and you need to assign peaks to promoters/ORFs and count co-regulated, co-occupied loci.

## When NOT to use

- The question is about the regulatory mechanism at a promoter (e.g., H-NS remodeling/displacement) rather than assigning peaks to promoters and counting co-regulated loci
- ChIP-seq peaks have not been called or overlap sets between regulators have not been established (the pipeline starts from MACs2 peaks and overlapping intervals)
- RNA-seq co-regulation calls across the relevant conditions (e.g., GSE136050 population-density conditions) are unavailable, so promoter co-occupancy cannot be intersected with co-regulation

## Inputs

- MACs2 ChIP-seq peak lists for two regulators (e.g., 506 LuxR peaks and 302 H-NS peaks)
- Overlapping/co-occupied peak intervals (88 overlapping peaks proximal to an ORF ≤500 bp)
- RNA-seq differential expression results (GSE136050) identifying co-regulated genes
- V. harveyi genome annotation (ORF coordinates) for nearest-ORF assignment

## Outputs

- Table of promoters and their associated downstream genes that are both co-occupied and co-regulated (expected: 28 promoters driving 63 genes)
- Annotated peak-to-gene/promoter assignment table from getPeakAnno.pl / getGenePeakAnno2.pl
- Venn-style overlap sets from findOverlaps.pl and LuxR-vs-H-NS peak relations from findClosest2.pl

## How to apply

Start from the set of overlapping peaks for the two regulators (in this study, 88 peaks overlapped between 506 LuxR peaks and 302 H-NS peaks and were proximal to an ORF within 500 bp). Annotate each filtered co-occupied region to its nearest ORF using the deposited Perl utilities: getPeakAnno.pl adds annotation information to MACs2 peaks, getGenePeakAnno2.pl adds peaks to gene information, and findClosest2.pl relates LuxR peaks to H-NS double peaks, retaining peaks within 500 bp upstream of an ORF. Load RNA-seq differential expression results (GSE136050) identifying genes co-regulated across population-density conditions, then intersect the co-occupied promoter set with the co-regulated gene set, matching each promoter to its downstream ORF. Compile the intersected loci into a table of promoters and associated genes, and inspect peak-to-promoter assignments in JBrowse or the UCSC Genome Browser to confirm consistency with the genome annotation.

## Related tools

- **Perl** (Runs the deposited scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl, findOverlaps.pl) that annotate MACs2 peaks, assign peaks to genes/ORFs, and intersect co-occupied promoters with co-regulated genes) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Visual inspection of the resulting loci to confirm peak-to-promoter assignments are consistent with the genome annotation)
- **UCSC Genome Browser** (Alternative genome browser for verifying peak-to-promoter assignments against the annotation)

## Examples

```
perl getPeakAnno.pl && perl getGenePeakAnno2.pl && perl findClosest2.pl
```

## Evaluation signals

- The reconstruction reproduces the reported result of 28 promoters controlling 63 genes co-regulated by both LuxR and H-NS
- Every retained peak is proximal to an ORF within the 500 bp upstream threshold before promoter-gene matching
- The intersection starts from the 88 overlapping LuxR/H-NS peaks (out of 506 LuxR and 302 H-NS peaks) filtered with transcriptomic data and gene organization (65 filtered regions as the intermediate set)
- Peak-to-promoter assignments visually verified in JBrowse/UCSC Genome Browser agree with the genome annotation
- Each promoter in the output table is matched to its downstream ORF from the RNA-seq co-regulated gene set

## Limitations

- Only peaks proximal to an ORF (≤500 bp upstream) are considered; distal or intergenic-overlapping peaks are excluded from promoter assignment
- The Perl scripts assume MACs2-formatted peak input and are written for the LuxR/H-NS two-regulator comparison; adapting them to other regulators or peak callers requires code changes
- The 28-promoter/63-gene result depends on the RNA-seq co-regulation definition from GSE136050; different expression cutoffs would change the intersected set
- Transcriptional control of each promoter is likely promoter dependent, so the intersection identifies co-regulated loci but not the underlying repression/activation mechanism

## Evidence
