---
name: barcode-span-annotation-processing
description: Use when phasing with 10× linked-reads and fragments must be filtered
  against barcode-sharing artifacts — e.g. requiring end-read MAPQ >= 30, capping
  barcode span at 60 kb, and bounding haplotype blocks over filtered variants.
when_to_use_negative:
- Input is Hi-C, PacBio SMRT or Oxford Nanopore data without 10× barcodes — the barcode-span
  annotation and 60 kb span cap are 10× linked-read specific
- Phasing organisms with ploidy higher than diploid — the graph bisection model does
  not fit increased ploidy
- Assembling haplotype-resolved structural variation — SpecHap supports only SNPs,
  short insertions and deletions
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Diploid phasing algorithm that consumes the filtered fragment file; spectral
    graph theory based with divide-and-conquer over user-defined intervals
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified ExtractHair utility used with --10x 1 to generate the
    10X fragment file from the BAM and bgzipped VCF
  repo: https://github.com/vibansal/HapCUT2
- name: htslib
  role: Provides bgzip and tabix used to compress and index barcode_spanning.bed
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:16dbbd9bf044957541b975c1cf2e3a75600ec9d89c1ab40bf5d1101eed737d87
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
  - HapCUT2
  - SpecHap
  - htslib
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# barcode-span-annotation-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A filtering and annotation skill for SpecHap 10× linked-reads phasing that counters reads sharing the same barcode but originating from different DNA molecules, by annotating each barcode's spanning range (BarcodeExtract / barcode_spanning.bed) and retaining only fragments passing mapping quality, span, and haplotype-block constraints.

## When to use

Use when phasing with 10× linked-reads and fragments must be filtered against barcode-sharing artifacts — e.g. requiring end-read MAPQ >= 30, capping barcode span at 60 kb, and bounding haplotype blocks over filtered variants.

## When NOT to use

- Input is Hi-C, PacBio SMRT or Oxford Nanopore data without 10× barcodes — the barcode-span annotation and 60 kb span cap are 10× linked-read specific
- Phasing organisms with ploidy higher than diploid — the graph bisection model does not fit increased ploidy
- Assembling haplotype-resolved structural variation — SpecHap supports only SNPs, short insertions and deletions

## Inputs

- 10× linked-reads BAM file
- bgzipped VCF of variant calls
- barcode whitelist (only fragments with white-listed barcodes are kept)
- barcode spanning annotations (barcode_spanning.bed, bgzipped and tabix-indexed)
- extractHAIRS 10X fragment file (sixth column sortable)

## Outputs

- barcode_spanning.bed — per-barcode inferred spanning ranges (bgzip + tabix indexed)
- filtered.frag — fragment file sorted in VCF position order containing only fragments passing MAPQ >= 30, 60 kb span, and block-size <= 30 constraints

## How to apply

Generate a 10X fragment file by running extractHAIRS (the modified utility originally from HapCUT2) with the --10x 1 flag on the linked-reads BAM and a bgzipped VCF. Annotate each barcode's inferred spanning range with BarcodeExtract to produce barcode_spanning.bed, then compress and index it with bgzip and tabix (from htslib). Sort the fragment file by its sixth column with 'sort -n -k6', as required for 10X phasing input. In the filtering module, remove variants failing the allele depth/quality threshold, reject any haplotype block striding over 30 continuously filtered variants, require barcode end reads with mapping quality >= 30, and cap the barcode span at 60 kb using the barcode spanning BED. Write the retained fragments, sorted in VCF position order, to filtered.frag for downstream SpecHap phasing.

## Related tools

- **SpecHap** (Diploid phasing algorithm that consumes the filtered fragment file; spectral graph theory based with divide-and-conquer over user-defined intervals) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified ExtractHair utility used with --10x 1 to generate the 10X fragment file from the BAM and bgzipped VCF) — https://github.com/vibansal/HapCUT2
- **htslib** (Provides bgzip and tabix used to compress and index barcode_spanning.bed) — https://github.com/samtools/htslib

## Examples

```
extractHAIRS --10x 1 --bam sample.bam --VCF variants.vcf.gz ; BarcodeExtract > barcode_spanning.bed ; bgzip barcode_spanning.bed && tabix barcode_spanning.bed.gz ; sort -n -k6 fragments.frag
```

## Evaluation signals

- Pipeline completes without error and filtered.frag exists
- filtered.frag entries are sorted in VCF position order
- All retained fragments have barcode end reads with mapping quality >= 30
- No retained fragment exceeds a 60 kb barcode span according to the barcode spanning BED
- No haplotype block strides over 30 continuously filtered variants

## Limitations

- There is no guarantee SpecHap provides optimal solutions, and the interval size parameter might affect the phasing result — a larger interval size might reduce efficiency
- A comprehensive analysis of the influence of technological-specific error over phasing quality has not yet been conducted for 10× linked-reads
- In the published benchmark, 10× linked-reads were filtered so that only fragments with white-listed barcodes were kept — barcodes absent from the whitelist are silently excluded

## Evidence

- [results] reads were filtered so that only fragments with white-listed barcodes were kept.: "reads were filtered so that only fragments with white-listed barcodes were kept."
- [discussion] SpecHap adopts divide-and-conquer to accelerate the computation by dividing chromosomes into intervals of user-defined size.: "SpecHap adopts divide-and-conquer to accelerate the computation by dividing chromosomes into intervals of user-defined size."
- [readme] A modified utility software ExtractHair, originally from HAPCUT2, is needed for fragment processing.: "A modified utility software ExtractHair, originally from [HAPCUT2](https://github.com/vibansal/HapCUT2), is needed for fragment processing."
- [readme] SpecHap requires at least a fragment file and a bgziped: "SpecHap requires at least a fragment file and a bgziped"
