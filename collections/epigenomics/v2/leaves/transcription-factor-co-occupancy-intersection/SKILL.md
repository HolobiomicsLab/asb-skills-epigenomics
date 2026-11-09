---
name: transcription-factor-co-occupancy-intersection
description: Use when two transcription factors have ChIP-seq peak sets and transcriptomic
  differential-expression data, and the goal is promoters and downstream genes co-occupied
  and co-regulated by both factors.
when_to_use_negative:
- Input is a single transcription factor's ChIP-seq data with no second factor to
  intersect — co-occupancy logic does not apply.
- The question is about ChIP-seq peak calling quality or normalization rather than
  promoter-level overlap with transcriptomic co-regulation.
- Peaks are not anchored to annotated ORFs/promoters (no gene annotation available),
  so the 500-bp-upstream promoter assignment cannot be made.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: Perl
  role: Runs the deposited scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findOverlaps.pl,
    findClosest2.pl) that annotate peaks to ORFs and intersect co-occupied promoters
    with co-regulated genes
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Visual inspection of co-occupied promoter loci to verify peak-to-promoter
    assignments against the genome annotation
- name: UCSC Genome Browser
  role: Alternative genome-browser check of the reconstructed co-occupied loci
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:773a2188bf885e1fde948c89d4d146db8707d823beb624587353b13881d28e83
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# transcription-factor-co-occupancy-intersection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for identifying promoters co-occupied and co-regulated by two transcription factors (here V. harveyi LuxR and H-NS) by intersecting ChIP-seq peak overlaps with RNA-seq co-regulation data using deposited Perl annotation scripts.

## When to use

Use when two transcription factors have ChIP-seq peak sets and transcriptomic differential-expression data, and the goal is promoters and downstream genes co-occupied and co-regulated by both factors.

## When NOT to use

- Input is a single transcription factor's ChIP-seq data with no second factor to intersect — co-occupancy logic does not apply.
- The question is about ChIP-seq peak calling quality or normalization rather than promoter-level overlap with transcriptomic co-regulation.
- Peaks are not anchored to annotated ORFs/promoters (no gene annotation available), so the 500-bp-upstream promoter assignment cannot be made.

## Inputs

- Macs2 ChIP-seq peak lists for both transcription factors (e.g., 506 LuxR peaks and 302 H-NS peaks)
- RNA-seq differential expression results for co-regulation (NCBI GEO GSE136050)
- Genome annotation with ORF coordinates (V. harveyi)
- ChIP-seq dataset GSE136191
- Multi-fasta genome files for annotation cleanup (via reformatFasta.pl)

## Outputs

- Table of co-occupied, co-regulated promoters and their downstream genes (28 promoters / 63 genes in the source study)
- Venn-diagram overlap sets of intersecting peaks (findOverlaps.pl)
- Peak-to-gene annotation table (getGenePeakAnno2.pl)
- Jbrowse/UCSC session of annotated co-occupied loci

## How to apply

Start from ChIP-seq peak counts (e.g., 506 LuxR peaks and 302 H-NS peaks from Macs2) and retain only peaks that overlap between the two factors and lie within 500 bp upstream of an open reading frame (88 regions in the V. harveyi study). Load RNA-seq differential-expression results (GSE136050) identifying genes co-regulated by LuxR and H-NS across population-density conditions. Annotate peaks to nearest ORFs using the repository's Perl scripts — getPeakAnno.pl and getGenePeakAnno2.pl add annotation and peak-to-gene information, findOverlaps.pl builds the Venn-diagram overlap set, and findClosest2.pl relates LuxR peaks to H-NS double peaks. Intersect the co-occupied promoter set with the co-regulated gene set by matching each promoter to its downstream ORF; in this dataset the intersection yields 28 promoters controlling 63 genes. Finally, inspect the loci in JBrowse/UCSC Genome Browser to confirm peak-to-promoter assignments against the genome annotation, and validate ChIP-seq by qPCR.

## Related tools

- **Perl** (Runs the deposited scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findOverlaps.pl, findClosest2.pl) that annotate peaks to ORFs and intersect co-occupied promoters with co-regulated genes) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Visual inspection of co-occupied promoter loci to verify peak-to-promoter assignments against the genome annotation)
- **UCSC Genome Browser** (Alternative genome-browser check of the reconstructed co-occupied loci)

## Examples

```
perl getGenePeakAnno2.pl
```

## Evaluation signals

- Overlap step reproduces 88 LuxR/H-NS overlapping peaks proximal to an ORF (≤500 bp) from the 506 and 302 input peaks.
- Final intersection reproduces 28 promoters driving 63 co-regulated genes, matching the published result.
- Each retained promoter maps to a downstream ORF within the 500 bp upstream window using getGenePeakAnno2.pl output.
- ChIP-seq peak assignments are consistent with the genome annotation when inspected in JBrowse/UCSC.
- ChIP-seq calls are validated by qPCR at representative co-occupied promoters.

## Limitations

- Findings are specific to this V. harveyi LuxR/H-NS dataset; peak counts and thresholds (500 bp upstream window) must be re-derived for other systems.
- Peak-to-promoter assignment by nearest ORF can misassign in densely packed genomes; browser inspection in JBrowse/UCSC is required to confirm.
- The scripts assume Macs2-format peak files and a specific genome annotation; reformatFasta.pl must clean multi-fasta inputs before use.
- Intersecting with transcriptomic co-regulation depends on the chosen differential-expression conditions (population-density series in GSE136050).

## Evidence
