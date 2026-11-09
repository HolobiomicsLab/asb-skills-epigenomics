---
name: linked-read-fragment-filtering
description: Use when phasing 10× linked-read data (e.g. with SpecHap or HapCUT2)
  and fragments must be filtered so that barcoded fragments spanning distant loci
  or low-quality reads do not corrupt haplotype blocks.
when_to_use_negative:
- Input is Hi-C, PacBio SMRT, or Oxford Nanopore data rather than 10× linked-reads
  — the barcode-span and MAPQ heuristics are 10X-specific
- Input fragments have not been extracted from the linked-reads BAM yet; run extractHAIRS
  fragment extraction first
- The goal is haplotype-resolved structural variation — SpecHap supports only SNPs
  and short insertions/deletions
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SpecHap
  role: provides the linked-read filtering module (allele depth/quality variant filtering,
    MAPQ, barcode-span and block-size constraints) ahead of spectral graph phasing
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: source of the modified ExtractHair utility used to generate 10X fragment files
    from the linked-reads BAM with --10x 1
  repo: https://github.com/vibansal/HapCUT2
- name: BarcodeExtract
  role: annotates each barcode's inferred spanning range to barcode_spanning.bed,
    used to enforce the 60 kb barcode span cap
  repo: https://github.com/deepomicslab/SpecHap
- name: htslib
  role: supplies bgzip and tabix for compressing and indexing the VCF and barcode_spanning.bed
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:8e577e4ba86a12d792a1b0a684ad297447ce23d5f01c2214083f2aafe89b3e57
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/deepomicslab/SpecHap
  tools:
  - BarcodeExtract
  - HapCUT2
  - SpecHap
  - htslib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# linked-read-fragment-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements SpecHap's variant and barcode filtering heuristics for 10× linked-read phasing, countering the problem that reads sharing a barcode may originate from different DNA molecules. Variants failing allele depth/quality thresholds are removed, and fragments failing MAPQ, barcode-span, and block-size constraints are dropped before phasing.

## When to use

Use when phasing 10× linked-read data (e.g. with SpecHap or HapCUT2) and fragments must be filtered so that barcoded fragments spanning distant loci or low-quality reads do not corrupt haplotype blocks.

## When NOT to use

- Input is Hi-C, PacBio SMRT, or Oxford Nanopore data rather than 10× linked-reads — the barcode-span and MAPQ heuristics are 10X-specific
- Input fragments have not been extracted from the linked-reads BAM yet; run extractHAIRS fragment extraction first
- The goal is haplotype-resolved structural variation — SpecHap supports only SNPs and short insertions/deletions

## Inputs

- 10× linked-reads BAM file
- bgzipped VCF of variant calls
- 10X fragment file produced by extractHAIRS with --10x 1
- barcode_spanning.bed produced by BarcodeExtract (bgzipped and tabix-indexed)

## Outputs

- filtered.frag: fragment file retaining only fragments passing MAPQ >= 30, 60 kb barcode-span cap, and block-size <= 30 constraints, sorted in VCF position order

## How to apply

First generate a 10X fragment file from the linked-reads BAM and a bgzipped VCF by running the modified HapCUT2 extractHAIRS with the --10x 1 flag, then annotate each barcode's inferred spanning range with BarcodeExtract into barcode_spanning.bed, compressing and indexing it with bgzip and tabix. Sort the fragment file by its sixth column ('sort -n -k6') as required for 10X phasing input. Apply the filtering module: remove variants failing the allele depth/quality threshold, require barcode end reads to have mapping quality >= 30, cap the barcode span at 60 kb using the barcode spanning bed, and disallow any haplotype block from striding over 30 continuously filtered variants — this guards against barcodes whose reads come from different DNA molecules. Write the retained fragments, sorted in VCF position order, to filtered.frag.

## Related tools

- **SpecHap** (provides the linked-read filtering module (allele depth/quality variant filtering, MAPQ, barcode-span and block-size constraints) ahead of spectral graph phasing) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (source of the modified ExtractHair utility used to generate 10X fragment files from the linked-reads BAM with --10x 1) — https://github.com/vibansal/HapCUT2
- **BarcodeExtract** (annotates each barcode's inferred spanning range to barcode_spanning.bed, used to enforce the 60 kb barcode span cap) — https://github.com/deepomicslab/SpecHap
- **htslib** (supplies bgzip and tabix for compressing and indexing the VCF and barcode_spanning.bed) — https://github.com/samtools/htslib

## Examples

```
sort -n -k6 fragments.frag > fragments.sorted.frag
```

## Evaluation signals

- The pipeline completes without error and filtered.frag exists
- filtered.frag is sorted by VCF variant position
- Every fragment in filtered.frag has end-read mapping quality >= 30
- No retained fragment exceeds the 60 kb barcode span derived from barcode_spanning.bed
- No haplotype block strides over more than 30 continuously filtered variants

## Limitations

- A comprehensive analysis of the influence of technology-specific error over phasing quality has not yet been conducted for Hi-C and 10× linked-reads
- Only fragments with white-listed barcodes are kept, so barcodes outside the whitelist are silently dropped
- SpecHap supports only SNPs, short insertions and deletions; structural variation is not handled

## Evidence
