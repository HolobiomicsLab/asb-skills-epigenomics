---
name: quorum-sensing-gene-expression-interpretation
description: Use when RNA-seq read counts from V. harveyi quorum sensing mutants (wild-type,
  Δhns, ΔluxR, ΔluxR Δhns) must be compared to enumerate H-NS-regulated genes and
  verify luxCDABE recovery.
when_to_use_negative:
- The goal is ChIP-seq peak calling or promoter occupancy analysis (506 LuxR peaks,
  302 H-NS peaks) rather than differential expression interpretation
- Input is ChIP-qPCR or ChIP-PCR occupancy data for autoinducer-dependent H-NS eviction
  rather than RNA-seq counts
- Samples are from growth conditions other than OD600 = 1.0, where the reported regulon
  sizes (726 and 824 genes) would not be valid acceptance baselines
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Perl
  role: Runs the deposited read-processing, gene-level counting, and differential
    expression scripts used to call the H-NS regulon
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:57625bc40fe7685dccf7b7bcca0e2cfff99215b6de0f6c74bf9db7ea6360ac2e
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# quorum-sensing-gene-expression-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets RNA-seq differential expression in V. harveyi quorum sensing genetics by calling the H-NS regulon in wild-type versus Δhns and ΔluxR versus ΔluxR Δhns backgrounds at OD600 = 1.0, to quantify H-NS-mediated repression and confirm LuxR-dependent counter-silencing of luxCDABE.

## When to use

Use when RNA-seq read counts from V. harveyi quorum sensing mutants (wild-type, Δhns, ΔluxR, ΔluxR Δhns) must be compared to enumerate H-NS-regulated genes and verify luxCDABE recovery.

## When NOT to use

- The goal is ChIP-seq peak calling or promoter occupancy analysis (506 LuxR peaks, 302 H-NS peaks) rather than differential expression interpretation
- Input is ChIP-qPCR or ChIP-PCR occupancy data for autoinducer-dependent H-NS eviction rather than RNA-seq counts
- Samples are from growth conditions other than OD600 = 1.0, where the reported regulon sizes (726 and 824 genes) would not be valid acceptance baselines

## Inputs

- RNA-seq raw reads (FASTQ) for wild-type, Δhns, ΔluxR, and ΔluxR Δhns strains at OD600 = 1.0 from GEO accession GSE136191
- V. harveyi reference genome for read mapping
- Perl scripts deposited at github.com/Juliacvk/LuxR_H-NS (read-processing, counting, and differential expression pipeline)

## Outputs

- Gene-level count matrix for all four strains at OD600 = 1.0
- H-NS regulon gene list for the wild-type versus Δhns comparison, with direction labels (repressed vs derepressed by H-NS)
- H-NS regulon gene list for the ΔluxR versus ΔluxR Δhns comparison
- luxC fold change summary and regulon gene counts (total and repressed) for both comparisons

## How to apply

Download RNA-seq raw reads from NCBI GEO accession GSE136191 for wild-type, Δhns, ΔluxR, and ΔluxR Δhns strains grown to OD600 = 1.0, and set up the authors' Perl read-processing and counting pipeline from github.com/Juliacvk/LuxR_H-NS. Map reads to the V. harveyi reference genome to build a gene-level count matrix, then run two differential expression comparisons: wild-type versus Δhns (calls the H-NS regulon in the LuxR-intact background) and ΔluxR versus ΔluxR Δhns (calls the regulon in the ΔluxR background), retaining genes with >2-fold change and P < 0.05 and labeling each as repressed vs derepressed by H-NS. Extract the luxC fold change from the wild-type versus Δhns comparison to interpret the LuxR-dependent activation defect. Interpretation rests on the counter-silencing model: LuxR evicts H-NS from quorum sensing promoters, so H-NS repression is expected to persist in the ΔluxR background and luxCDABE genes should remain in the H-NS regulon there.

## Related tools

- **Perl** (Runs the deposited read-processing, gene-level counting, and differential expression scripts used to call the H-NS regulon) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
perl reformatFasta.pl --input Vharveyi_genome.fasta && perl getPeakAnno.pl --peaks macs2_peaks.bed --annotation genes.gff
```

## Evaluation signals

- Wild-type versus Δhns comparison yields 726 H-NS-regulated genes (>2-fold, P < 0.05), of which 596 are repressed by H-NS
- ΔluxR versus ΔluxR Δhns comparison yields 824 H-NS-regulated genes, of which 430 are repressed
- luxCDABE genes are present in the ΔluxR-background H-NS regulon, confirming LuxR-dependent derepression is required for their expression
- luxC fold change from the wild-type versus Δhns comparison is consistent with the reported ~1.3-fold change
- Each regulon gene carries a direction-of-change label (repressed vs derepressed by H-NS)

## Limitations

- luxCDABE and other co-regulated genes show no clear HGT origin signs by GC-content analysis; the evolutionary origin of the H-NS/LuxR regulatory scheme remains speculative
- In an hns deletion strain, LuxR is no longer necessary for transcription activation of the bioluminescence genes, so regulon calls from the two genetic backgrounds are not interchangeable
- Other regulatory mechanisms acting at the transcriptional and/or post-transcriptional levels repress luxCDABE at LCD even in the absence of hns; these are unidentified
- The deposited Perl repository README documents only peak-overlap and annotation helpers (findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl, reformatFasta.pl, FileUtil.pm); the DE scripts must be located within the same repo

## Evidence

- [full_text] H-NS regulon sizes and luxCDABE recovery: [evidence span withheld — anchor status: not_found]
