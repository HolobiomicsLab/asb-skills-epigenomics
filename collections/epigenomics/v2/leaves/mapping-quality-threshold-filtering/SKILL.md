---
name: mapping-quality-threshold-filtering
description: Use when phasing 10x linked-reads with SpecHap/HapCUT2 and fragments
  from different DNA molecules share the same barcode, inflating spurious linkage
  between distant variants.
when_to_use_negative:
- Input fragments are not 10x linked-reads (e.g. PacBio SMRT or ONT long reads) —
  the barcode-end MAPQ >= 30 and 60 kb span constraints are specific to barcoded linked-read
  data.
- Organism or assembly with ploidy higher than diploid — SpecHap's current graph bisection
  model does not fit increased ploidy.
- Input variants are already reliable high-quality calls from 30X Illumina short reads
  and the only goal is phasing, not fragment-level barcode filtering.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: Diploid phasing algorithm whose linked-read filtering module enforces the
    MAPQ >= 30, 60 kb span, and block-size <= 30 heuristics
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified ExtractHair utility used to extract 10x fragments (extractHAIRS
    --10x 1) from the BAM and bgzipped VCF
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:84cc8cd700807dad815dc44b04f21dd2be25c8afc4fb69ff86447a99247f71ed
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mapping-quality-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies SpecHap's 10x linked-read filtering heuristics — requiring barcode-end reads to have mapping quality >= 30, capping each barcode's inferred span at 60 kb via a barcode_spanning.bed file, and rejecting haplotype blocks that stride over more than 30 continuously filtered variants — so that reads sharing a barcode but originating from different DNA molecules do not corrupt phasing.

## When to use

Use when phasing 10x linked-reads with SpecHap/HapCUT2 and fragments from different DNA molecules share the same barcode, inflating spurious linkage between distant variants.

## When NOT to use

- Input fragments are not 10x linked-reads (e.g. PacBio SMRT or ONT long reads) — the barcode-end MAPQ >= 30 and 60 kb span constraints are specific to barcoded linked-read data.
- Organism or assembly with ploidy higher than diploid — SpecHap's current graph bisection model does not fit increased ploidy.
- Input variants are already reliable high-quality calls from 30X Illumina short reads and the only goal is phasing, not fragment-level barcode filtering.

## Inputs

- 10x linked-reads BAM file
- bgzipped VCF of variant calls
- fragment file produced by extractHAIRS with --10x 1
- barcode_spanning.bed (BarcodeExtract output, bgzip- and tabix-compressed)

## Outputs

- filtered.frag — retained fragments sorted in VCF position order, containing only fragments passing MAPQ >= 30, 60 kb span, and block-size <= 30 constraints

## How to apply

Generate a 10x fragment file from the linked-reads BAM and a bgzipped VCF by running extractHAIRS (modified from HapCUT2) with the --10x 1 flag. Annotate each barcode's inferred spanning range with BarcodeExtract into barcode_spanning.bed, then bgzip and tabix-index it. Sort the fragment file by its sixth column with 'sort -n -k6' as required for 10x phasing input. Implement the filtering module: remove variants failing the allele depth/quality threshold, require barcode end reads with mapping quality >= 30, cap barcode span at 60 kb using the barcode spanning bed, and disallow any haplotype block from striding over 30 continuously filtered variants. Write the retained fragments, in VCF position order, to filtered.frag. The rationale is that the MAPQ >= 30 cutoff on barcode-end reads plus the 60 kb span cap excludes fragments whose barcode bridges unrelated DNA molecules, while the block-size <= 30 rule prevents a block from walking across long runs of low-quality filtered variants.

## Related tools

- **SpecHap** (Diploid phasing algorithm whose linked-read filtering module enforces the MAPQ >= 30, 60 kb span, and block-size <= 30 heuristics) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified ExtractHair utility used to extract 10x fragments (extractHAIRS --10x 1) from the BAM and bgzipped VCF) — https://github.com/vibansal/HapCUT2

## Examples

```
extractHAIRS --10x 1 linked_reads.bam variants.vcf.gz > fragments.frag; sort -n -k6 fragments.frag
```

## Evaluation signals

- The pipeline runs without error and filtered.frag exists on disk.
- filtered.frag is sorted by VCF position order.
- Every retained fragment in filtered.frag has barcode-end reads with mapping quality >= 30.
- No retained fragment's barcode spans more than 60 kb according to barcode_spanning.bed.
- No haplotype block strides over more than 30 continuously filtered variants.

## Limitations

- A comprehensive analysis of the influence of technology-specific error over phasing quality for 10x linked-reads (and Hi-C) has not yet been conducted.
- SpecHap provides no guarantee of optimal solutions, and the chromosome interval size parameter can affect the phasing result and efficiency.
- SpecHap supports only SNPs, short insertions and deletions; haplotype-resolved structural variation is not supported.

## Evidence

- [methods] require barcode end reads with mapping quality >= 30, and cap barcode span at 60 kb using the barcode spanning bed: [evidence span withheld — anchor status: not_found]
- [supplementary] a haplotype block is disallowed from striding over 30 continuously filtered variants: [evidence span withheld — anchor status: not_found]
