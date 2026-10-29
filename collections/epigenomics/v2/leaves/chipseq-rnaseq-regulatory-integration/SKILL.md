---
name: chipseq-rnaseq-regulatory-integration
description: Use when you have two genome-wide binding peak sets (e.g. MACS2 ChIP-seq
  peaks for two TFs) and RNA-seq differential expression results, and need to define
  promoters bound and co-regulated by both regulators.
when_to_use_negative:
- You only have single-factor ChIP-seq peaks with no RNA-seq expression data — binding
  alone cannot establish co-regulation, only co-occupancy.
- Your peaks are not proximal to annotated ORFs (≤500 bp upstream rule here); promoter-level
  intersection requires gene-proximal assignments.
- The regulatory question is post-transcriptional (e.g. the unexplained luxCDABE translational
  control) — ChIP-seq/RNA-seq intersection addresses promoter occupancy and transcription
  only.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Perl
  role: Scripted intersection of ChIP-seq co-occupied promoter regions with RNA-seq
    co-regulated gene sets and peak-to-gene annotation
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Genome-browser inspection of peak-to-promoter assignments against the V. harveyi
    annotation
- name: UCSC Genome Browser
  role: Alternative genome-browser verification of locus assignments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:9559c0fae8a59a78a2a0c2c773790b50d3607a32e2ff8d878a946ec464ec8f46
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# chipseq-rnaseq-regulatory-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates ChIP-seq binding data (co-occupied LuxR/H-NS regions) with RNA-seq differential-expression data to identify promoters that are both co-occupied and co-regulated — here yielding 28 promoters controlling 63 genes in V. harveyi quorum sensing.

## When to use

Use when you have two genome-wide binding peak sets (e.g. MACS2 ChIP-seq peaks for two TFs) and RNA-seq differential expression results, and need to define promoters bound and co-regulated by both regulators.

## When NOT to use

- You only have single-factor ChIP-seq peaks with no RNA-seq expression data — binding alone cannot establish co-regulation, only co-occupancy.
- Your peaks are not proximal to annotated ORFs (≤500 bp upstream rule here); promoter-level intersection requires gene-proximal assignments.
- The regulatory question is post-transcriptional (e.g. the unexplained luxCDABE translational control) — ChIP-seq/RNA-seq intersection addresses promoter occupancy and transcription only.

## Inputs

- MACS2 ChIP-seq peak lists for two transcription factors (LuxR: 506 peaks; H-NS: 302 peaks; GSE136191)
- Filtered/overlapped co-occupied regions (88 overlapping peaks within ≤500 bp upstream of an ORF; 65 after transcriptomic filtering)
- RNA-seq differential expression results (GSE136050) identifying genes co-regulated by LuxR and H-NS across population-density conditions
- Genome annotation (ORF coordinates) for V. harveyi
- Perl helper scripts from github.com/Juliacvk/LuxR_H-NS (FileUtil.pm, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl)

## Outputs

- Table of co-occupied promoters matched to downstream ORFs (peak-to-promoter assignments)
- Final set of 28 promoters driving 63 co-regulated genes
- Overlapping-peak sets for Venn diagram comparison (findOverlaps.pl)

## How to apply

Start from the overlapped ChIP-seq peaks (for LuxR and H-NS, 506 and 302 MACS2 peaks reduced to 88 overlapping peaks proximal to an ORF within ≤500 bp upstream). Annotate each retained co-occupied region to its nearest open reading frame using the deposited Perl scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl from github.com/Juliacvk/LuxR_H-NS) to build a peak-to-gene/promoter table. Then intersect this co-occupied promoter set with RNA-seq co-regulation calls (GSE136050, genes differentially controlled by LuxR and H-NS across population-density conditions), matching each promoter to its downstream ORF. Compile the intersected loci and confirm the reconstruction reproduces the reported result: 28 co-regulated, co-occupied promoters driving 63 genes. Visually inspect assignments in JBrowse/UCSC Genome Browser against the genome annotation before finalizing.

## Related tools

- **Perl** (Scripted intersection of ChIP-seq co-occupied promoter regions with RNA-seq co-regulated gene sets and peak-to-gene annotation) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Genome-browser inspection of peak-to-promoter assignments against the V. harveyi annotation)
- **UCSC Genome Browser** (Alternative genome-browser verification of locus assignments)

## Examples

```
perl getGenePeakAnno2.pl
```

## Evaluation signals

- The intersected promoter/gene table reproduces the published counts: 28 promoters controlling 63 co-regulated genes.
- Intermediate counts match the published funnel: 506 LuxR peaks + 302 H-NS peaks → 88 overlapping peaks proximal to an ORF (≤500 bp) → 65 filtered regions → 28 promoters.
- Every retained promoter maps to a downstream ORF within 500 bp upstream and appears in the RNA-seq co-regulation set.
- Peak-to-promoter assignments visually consistent with the genome annotation in JBrowse/UCSC.
- ChIP-seq peaks were validated by qPCR before integration.

## Limitations

- Peak-to-ORF assignment relies on the ≤500 bp upstream rule; co-occupied binding farther from ORFs may be missed.
- The intersection establishes co-occupancy and co-regulation but not the repression mechanism, which the authors speculate is promoter dependent.
- Co-regulated genes such as luxCDABE show no clear HGT signature by GC-content analysis, so the regulatory scheme's evolutionary generality remains open.

## Evidence

- [abstract] Using RNA-seq and ChIP-seq, we determined that H-NS and LuxR co-regulate and co-occupy 28 promoters: "Using RNA-seq and ChIP-seq, we determined that H-NS and LuxR co-regulate and co-occupy 28 promoters"
- [abstract] H-NS and LuxR co-regulate and co-occupy 28 promoters driving expression of 63 genes across the genome: "H-NS and LuxR co-regulate and co-occupy 28 promoters driving expression of 63 genes across the genome"
- [discussion] Within the 506 and 302 peaks identified for LuxR and H-NS, respectively, 88 peaks overlapped and were proximal to an ORF (≤500 bp).: "Within the 506 and 302 peaks identified for LuxR and H-NS, respectively, 88 peaks overlapped and were proximal to an ORF (≤500 bp)."
- [supplementary] All Perl scripts have been deposited in Github: https://github.com/Juliacvk/LuxR_H-NS: "All Perl scripts have been deposited in Github: https://github.com/Juliacvk/LuxR_H-NS"
