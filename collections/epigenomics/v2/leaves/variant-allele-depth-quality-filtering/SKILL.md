---
name: variant-allele-depth-quality-filtering
description: Use when preparing 10x linked-read fragment files (e.g. from extractHAIRS
  --10x 1 output) for phasing and you must remove low-depth/low-quality variants and
  enforce barcode-level constraints (MAPQ >= 30 ends, <= 60 kb span).
when_to_use_negative:
- Input is general WGS, Hi-C, PacBio SMRT or ONT data without barcodes — the 10x-specific
  barcode span/MAPQ heuristics and BarcodeExtract annotation do not apply
- Variants have not yet been fragment-extracted (no fragment file from extractHAIRS),
  so there are no allele depths or block structure to filter on
- The goal is de novo variant calling rather than pre-phasing cleanup of existing
  VCF calls
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: Provides the phasing pipeline whose 10x linked-reads filtering module removes
    depth/quality-failing variants and enforces block-size, MAPQ and barcode-span
    constraints
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified ExtractHair utility (run with --10x 1) that generates
    the fragment file from the linked-reads BAM and bgzipped VCF before filtering
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:ab2f85441517830abf5baa983da6f52cef12359b031a4bc0f69bd0c2f7500ec3
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# variant-allele-depth-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-phasing filtering heuristics from SpecHap's 10x linked-reads workflow: variants failing allele depth/quality thresholds are removed before fragment processing, and haplotype blocks may not stride over more than 30 continuously filtered variants, countering barcodes shared between distant DNA molecules.

## When to use

Use when preparing 10x linked-read fragment files (e.g. from extractHAIRS --10x 1 output) for phasing and you must remove low-depth/low-quality variants and enforce barcode-level constraints (MAPQ >= 30 ends, <= 60 kb span).

## When NOT to use

- Input is general WGS, Hi-C, PacBio SMRT or ONT data without barcodes — the 10x-specific barcode span/MAPQ heuristics and BarcodeExtract annotation do not apply
- Variants have not yet been fragment-extracted (no fragment file from extractHAIRS), so there are no allele depths or block structure to filter on
- The goal is de novo variant calling rather than pre-phasing cleanup of existing VCF calls

## Inputs

- 10x linked-reads BAM file
- bgzipped VCF of variant calls
- fragment file produced by extractHAIRS with --10x 1
- barcode_spanning.bed (barcode spanning ranges) with bgzip/tabix index

## Outputs

- filtered.frag: retained fragments sorted in VCF position order, passing MAPQ >= 30, 60 kb span cap, and block-size <= 30 constraints

## How to apply

Generate a 10x fragment file from the linked-reads BAM and a bgzipped VCF by running extractHAIRS with the --10x 1 flag, then annotate each barcode's inferred spanning range with BarcodeExtract into barcode_spanning.bed, compressed and indexed with bgzip and tabix. Sort the fragment file by its sixth column with 'sort -n -k6' as required for 10x phasing input. Apply the filtering module: drop variants failing the allele depth/quality threshold, reject any haplotype block that spans more than 30 continuously filtered variants, require barcode end reads with mapping quality >= 30, and cap barcode span at 60 kb using the barcode spanning BED. The rationale is that linked-reads barcodes can be shared by reads originating from different DNA molecules, so depth/quality, MAPQ, span, and block-size heuristics suppress spurious long-range links. Write the retained fragments, sorted in VCF position order, to filtered.frag.

## Related tools

- **SpecHap** (Provides the phasing pipeline whose 10x linked-reads filtering module removes depth/quality-failing variants and enforces block-size, MAPQ and barcode-span constraints) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified ExtractHair utility (run with --10x 1) that generates the fragment file from the linked-reads BAM and bgzipped VCF before filtering) — https://github.com/vibansal/HapCUT2

## Examples

```
extractHAIRS --10x 1 --bam linked_reads.bam --vcf variants.vcf.gz --frag frags.txt && sort -n -k6 frags.txt
```

## Evaluation signals

- The pipeline completes without error and filtered.frag exists
- filtered.frag is sorted by VCF position (fragments were sorted after 'sort -n -k6' preprocessing)
- Every fragment in filtered.frag has barcode end reads with mapping quality >= 30
- No retained fragment/barcode spans more than 60 kb according to barcode_spanning.bed
- No haplotype block straddles more than 30 continuously filtered variants

## Limitations

- The 30-continuously-filtered-variant block limit and 60 kb barcode span cap are fixed heuristics; a comprehensive analysis of technology-specific error influence over 10x linked-read phasing quality has not yet been conducted
- Only fragments with white-listed barcodes are kept, so barcodes outside the whitelist are silently dropped
- SpecHap provides no guarantee of optimal solutions, and downstream interval size affects the phasing result even after filtering

## Evidence

- [other] Filter: apply the 10x linked-reads variant and barcode filtering heuristics: [evidence span withheld — anchor status: not_found]
