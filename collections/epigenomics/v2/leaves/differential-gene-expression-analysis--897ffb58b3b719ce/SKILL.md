---
name: differential-gene-expression-analysis
description: Use when RNA-seq differential expression results must be integrated with
  ChIP-seq peak-to-promoter annotations to identify loci where two regulators both
  bind and co-regulate the same genes.
when_to_use_negative:
- Input is only raw RNA-seq counts without matching ChIP-seq occupancy data — this
  skill requires co-occupied promoter regions, not expression data alone
- The analysis targets a single promoter (e.g. luxCDABE only) rather than a genome-wide
  intersection of co-occupied peaks with co-regulated genes
- Regulators of interest do not have ChIP-seq peak sets that overlap; the 500 bp upstream
  peak-to-ORF filtering step is the core precondition
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3169
tools:
- name: Perl
  role: Custom scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl, findOverlaps.pl)
    intersect ChIP-seq peaks with gene annotations and co-regulated gene sets
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: JBrowse
  role: Genome browser session used to inspect and verify peak-to-promoter assignments
- name: UCSC Genome Browser
  role: Alternative genome browser for visually confirming co-occupied promoter assignments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:d2d9dcdb7840fc87ed198b9113fa3841de6ba7add515fcabcc0c0167616b7dc5
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Intersects ChIP-seq co-occupied LuxR/H-NS promoter regions with RNA-seq differential co-regulation data (GSE136050) to derive the set of promoters and downstream genes co-regulated by both regulators. In the V. harveyi quorum sensing study this yielded 28 promoters controlling 63 co-regulated genes.

## When to use

Use when RNA-seq differential expression results must be integrated with ChIP-seq peak-to-promoter annotations to identify loci where two regulators both bind and co-regulate the same genes.

## When NOT to use

- Input is only raw RNA-seq counts without matching ChIP-seq occupancy data — this skill requires co-occupied promoter regions, not expression data alone
- The analysis targets a single promoter (e.g. luxCDABE only) rather than a genome-wide intersection of co-occupied peaks with co-regulated genes
- Regulators of interest do not have ChIP-seq peak sets that overlap; the 500 bp upstream peak-to-ORF filtering step is the core precondition

## Inputs

- Filtered co-occupied LuxR/H-NS ChIP-seq peak regions (Macs2 peaks, peaks within 500 bp upstream of an ORF)
- RNA-seq differential expression results (GSE136050) for LuxR/H-NS co-regulated genes
- Genome annotation with ORF coordinates
- Deposited Perl scripts from github.com/Juliacvk/LuxR_H-NS

## Outputs

- Table of co-regulated promoters and their associated downstream ORFs (expected: 28 promoters, 63 genes)
- Annotated peak-to-gene assignments verifiable in JBrowse/UCSC Genome Browser

## How to apply

Start from the filtered co-occupied ChIP-seq peak set (88 LuxR/H-NS overlapping peaks reduced to 65 after transcriptomic and gene-organization filtering) and annotate each peak to the nearest ORF, retaining peaks within 500 bp upstream. Load RNA-seq differential expression results (GSE136050) identifying genes co-regulated by LuxR and H-NS across population-density conditions. Use the deposited Perl scripts from github.com/Juliacvk/LuxR_H-NS to intersect the co-occupied promoter set with the co-regulated gene set, matching each promoter to its downstream ORF (getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl). Compile the intersected loci into a table of promoters and associated genes, and confirm the intersection reproduces the reported 28 promoters driving 63 genes. Inspect assignments in JBrowse to verify peak-to-promoter calls against the genome annotation.

## Related tools

- **Perl** (Custom scripts (getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl, findOverlaps.pl) intersect ChIP-seq peaks with gene annotations and co-regulated gene sets) — https://github.com/Juliacvk/LuxR_H-NS
- **JBrowse** (Genome browser session used to inspect and verify peak-to-promoter assignments)
- **UCSC Genome Browser** (Alternative genome browser for visually confirming co-occupied promoter assignments)

## Examples

```
perl getGenePeakAnno2.pl --peaks luxR_hns_cooccupied.bed --gff Vibrio_harveyi.gff --upstream 500 --rnaseq GSE136050_coregulated.tsv
```

## Evaluation signals

- The intersected promoter/gene table reproduces the reported 28 promoters driving 63 genes
- Every retained peak maps to an ORF within 500 bp upstream, matching the article's filtering threshold
- Peak-to-promoter assignments are visually consistent with the genome annotation when inspected in JBrowse/UCSC
- Overlapping peak sets reproduce the 88 overlapping LuxR/H-NS peaks reported from the 506 LuxR and 302 H-NS peak lists
- RNA-seq and ChIP-seq inputs match the deposited GEO accessions GSE136050 and GSE136191

## Limitations

- Post-transcriptional regulation controlling luxCDABE translation (candidate sRNA, RNA-binding protein, riboswitch, or RNA thermometer) remains unidentified and is not captured by this intersection approach
- Whether LuxR also functions alongside H-NS to repress gene expression is unresolved and requires additional experimentation
- Transcriptional control of each QS promoter is likely different; findings at luxCDABE may not generalize to all co-regulated promoters

## Evidence
