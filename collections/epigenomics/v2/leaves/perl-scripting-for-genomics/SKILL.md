---
name: perl-scripting-for-genomics
description: Use when reproducing the LuxR/H-NS ChIP-seq analyses in V. harveyi —
  counting overlapping peaks, detecting H-NS doublet peaks, or assigning peaks to
  ORFs — using the Perl scripts from the LuxR_H-NS GitHub repository.
when_to_use_negative:
- Input is already a gene expression matrix or promoter table rather than MACS2 peak
  interval coordinates — use differential-expression tooling instead
- Peak data are not from V. harveyi / not in the MACS2 interval format expected by
  FileUtil.pm and getPeakAnno.pl
- You need general-purpose genome interval arithmetic (e.g. large human datasets or
  bedtools-scale throughput); these scripts were written for this specific small bacterial
  genome analysis
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Perl
  role: 'Scripting language for all peak-processing steps: overlap detection, ORF
    annotation, doublet detection, and LuxR/H-NS doublet linkage'
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Visual inspection of peak-to-promoter assignments against the genome annotation
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:9208e99a1733a7e6c3029beb327f67bdf405ec3a2bf3ff666a8cd80eb4cf988f
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# perl-scripting-for-genomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for processing bacterial ChIP-seq peak intervals with the Perl scripts deposited in the Juliacvk/LuxR_H-NS repository: intersecting LuxR and H-NS MACS2 peaks, annotating peaks to V. harveyi ORFs, and relating LuxR peaks to H-NS doublet peaks to quantify co-occupied and co-regulated quorum-sensing loci.

## When to use

Use when reproducing the LuxR/H-NS ChIP-seq analyses in V. harveyi — counting overlapping peaks, detecting H-NS doublet peaks, or assigning peaks to ORFs — using the Perl scripts from the LuxR_H-NS GitHub repository.

## When NOT to use

- Input is already a gene expression matrix or promoter table rather than MACS2 peak interval coordinates — use differential-expression tooling instead
- Peak data are not from V. harveyi / not in the MACS2 interval format expected by FileUtil.pm and getPeakAnno.pl
- You need general-purpose genome interval arithmetic (e.g. large human datasets or bedtools-scale throughput); these scripts were written for this specific small bacterial genome analysis

## Inputs

- MACS2 ChIP-seq peak interval files for LuxR and H-NS in wild-type and ΔluxR V. harveyi (GEO GSE136191)
- RNA-seq differential expression results for LuxR/H-NS co-regulated genes (GEO GSE136050)
- V. harveyi ORF annotation / genome sequence (multi-FASTA, via reformatFasta.pl)

## Outputs

- Per-strain LuxR/H-NS overlap tallies (peak_overlap_counts.csv; expected 68 WT regions, 88 ORF-proximal co-occupied loci in the reported analysis)
- H-NS doublet table with LuxR proximity flags (hns_doublets.tsv; 52 doublets, 32 LuxR-proximal)
- Annotated peak-to-gene tables from getPeakAnno.pl / getGenePeakAnno2.pl and the 28-promoter/63-gene co-regulated promoter table

## How to apply

Download the MACS2 peak interval files for LuxR and H-NS from GEO accessions GSE136050 (RNA-seq/ChIP-seq) and GSE136191, then clone github.com/Juliacvk/LuxR_H-NS, which provides FileUtil.pm (file-loading helper), reformatFasta.pl, findOverlaps.pl (overlap sets for Venn diagrams), getPeakAnno.pl and getGenePeakAnno2.pl (peak-to-ORF annotation), and findClosest2.pl (relating LuxR peaks to H-NS double peaks). Run findOverlaps.pl-style interval intersection on the LuxR and H-NS peak sets per strain background, filtering overlaps to peaks proximal to an ORF (≤500 bp) to reproduce the 88 co-occupied regions reported in the discussion. For doublet analysis, apply the authors' criteria — two H-NS peaks within 2 kb whose intervening trough drops to ≤80% of the lower peak — yielding 52 doublets genome-wide, then use findClosest2.pl to flag doublets with a LuxR peak within 1 kb (32 of 52). Finally, intersect co-occupied promoters with RNA-seq co-regulation data to obtain the 28 promoters driving 63 co-regulated genes, and confirm assignments in JBrowse.

## Related tools

- **Perl** (Scripting language for all peak-processing steps: overlap detection, ORF annotation, doublet detection, and LuxR/H-NS doublet linkage) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Visual inspection of peak-to-promoter assignments against the genome annotation)

## Examples

```
git clone https://github.com/Juliacvk/LuxR_H-NS && cd LuxR_H-NS && perl findClosest2.pl
```

## Evaluation signals

- Total H-NS doublets detected equals 52, and the subset with a LuxR peak within 1 kb equals 32
- LuxR/H-NS overlap analysis yields 68 overlapping regions in wild-type and 88 co-occupied loci proximal to an ORF (≤500 bp) in the reported ΔluxR analysis
- Intersection with RNA-seq co-regulation data reproduces 28 promoters driving 63 genes
- Within the 506 LuxR and 302 H-NS peaks, the overlap counts are internally consistent with the reported Venn-diagram output of findOverlaps.pl
- Every output row from getPeakAnno.pl/getGenePeakAnno2.pl carries valid ORF annotation within the ≤500 bp proximity filter

## Limitations

- The README lists only a file inventory with no CLI usage, argument formats, or parameter documentation, so users must read the scripts (FileUtil.pm dependencies) to determine invocation
- Thresholds (2 kb doublet spacing, ≤80% trough, ≤500 bp ORF proximity, 1 kb LuxR proximity) are specific to this V. harveyi analysis and may not transfer to other organisms
- The article notes transcriptional control of each QS promoter is likely different, so the luxCDABE-based findings may not generalize to all co-occupied promoters

## Evidence
