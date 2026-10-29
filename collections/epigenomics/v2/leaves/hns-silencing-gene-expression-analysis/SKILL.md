---
name: hns-silencing-gene-expression-analysis
description: 'Use when reproducing the H-NS regulon from V. harveyi RNA-seq data:
  raw FASTQ in GEO (e.g. GSE136191), strains wild-type/Δhns/ΔluxR/ΔluxR Δhns, and
  the goal is gene counts regulated by H-NS including luxCDABE recovery.'
when_to_use_negative:
- Input is already a processed gene count matrix or differential expression table
  rather than raw GEO reads — the read-processing and counting steps are redundant.
- The question is about LuxR/H-NS ChIP-seq peak overlap and co-occupancy statistics
  (e.g. 506 LuxR peaks, 302 H-NS peaks, 88 overlapping peaks) rather than RNA-seq
  differential expression.
- Analysis targets a growth phase other than OD600 = 1.0 or a strain combination other
  than the four deposited in GSE136191, since the validation thresholds are specific
  to that condition.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: Perl
  role: Implements the authors' read-processing, counting, and differential expression
    scripts used to call the H-NS regulon
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:f4ea932f22e1ab352fedbce277e2618714fa22a5ec9895f6930afe760dc6d48c
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
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hns-silencing-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the H-NS regulon in V. harveyi by reproducing the paper's RNA-seq differential expression comparisons (wild-type vs Δhns and ΔluxR vs ΔluxR Δhns at OD600 = 1.0) using the authors' Perl scripts, GEO raw reads (GSE136191), and thresholds of >2-fold change and P < 0.05.

## When to use

Use when reproducing the H-NS regulon from V. harveyi RNA-seq data: raw FASTQ in GEO (e.g. GSE136191), strains wild-type/Δhns/ΔluxR/ΔluxR Δhns, and the goal is gene counts regulated by H-NS including luxCDABE recovery.

## When NOT to use

- Input is already a processed gene count matrix or differential expression table rather than raw GEO reads — the read-processing and counting steps are redundant.
- The question is about LuxR/H-NS ChIP-seq peak overlap and co-occupancy statistics (e.g. 506 LuxR peaks, 302 H-NS peaks, 88 overlapping peaks) rather than RNA-seq differential expression.
- Analysis targets a growth phase other than OD600 = 1.0 or a strain combination other than the four deposited in GSE136191, since the validation thresholds are specific to that condition.

## Inputs

- RNA-seq raw reads (FASTQ) for wild-type, Δhns, ΔluxR, and ΔluxR Δhns V. harveyi at OD600 = 1.0 from GEO accession GSE136191
- V. harveyi reference genome (multi-FASTA, cleaned/reformatted via reformatFasta.pl)
- Authors' Perl analysis scripts from github.com/Juliacvk/LuxR_H-NS

## Outputs

- Gene-level count matrix for all four strains at OD600 = 1.0
- H-NS regulon gene lists for wild-type vs Δhns (repressed vs derepressed) and ΔluxR vs ΔluxR Δhns
- luxC fold change extracted from the wild-type vs Δhns comparison

## How to apply

Download the RNA-seq raw reads for wild-type, Δhns, ΔluxR, and ΔluxR Δhns samples grown to OD600 = 1.0 from NCBI GEO accession GSE136191. Set up the authors' read-processing and counting pipeline from the Perl scripts deposited at github.com/Juliacvk/LuxR_H-NS (FileUtil.pm, reformatFasta.pl, plus peak-to-gene annotation scripts such as getPeakAnno.pl and getGenePeakAnno2.pl). Map reads to the V. harveyi reference genome and generate a gene-level count matrix for all four strains at OD600 = 1.0. Run two differential expression comparisons — wild-type versus Δhns (the H-NS regulon) and ΔluxR versus ΔluxR Δhns (the regulon in the ΔluxR background) — retaining genes with >2-fold change and P < 0.05, and recording direction of change (repressed vs derepressed by H-NS). The rationale is that H-NS represses quorum sensing loci such as luxCDABE, and LuxR counter-silences them, so the ΔluxR comparison should recover the luxCDABE genes in the H-NS regulon.

## Related tools

- **Perl** (Implements the authors' read-processing, counting, and differential expression scripts used to call the H-NS regulon) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
git clone https://github.com/Juliacvk/LuxR_H-NS
```

## Evaluation signals

- Wild-type vs Δhns comparison yields 726 H-NS-regulated genes, of which 596 are repressed (thresholds >2-fold, P < 0.05).
- ΔluxR vs ΔluxR Δhns comparison yields 824 genes, of which 430 are repressed.
- luxCDABE genes are present in the ΔluxR-background H-NS regulon.
- luxC fold change in the wild-type vs Δhns comparison is consistent with the reported ~1.3-fold change.
- Count matrix contains gene-level counts for all four strains (wild-type, Δhns, ΔluxR, ΔluxR Δhns) at OD600 = 1.0.

## Limitations

- The deposited Perl scripts documented in the README (findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl, findClosest2.pl) primarily cover peak overlap and gene annotation; the differential expression comparison logic must be inferred from the full script set.
- Regulon counts depend on the specific thresholds (>2-fold, P < 0.05) and the OD600 = 1.0 condition; other growth phases will not match the validated numbers.
- Even in Δhns, luxCDABE remains repressed at low cell density by other uncharacterized transcriptional/post-transcriptional mechanisms, so Δhns-vs-wild-type expression alone does not fully explain luxCDABE control.
- luxCDABE and other H-NS/LuxR co-regulated genes show no clear HGT-origin signs by GC-content analysis, so genomic-origin inferences from expression data are speculative.

## Evidence

- [other] H-NS regulates 824 genes in ΔluxR background: [evidence span withheld — anchor status: not_found]
