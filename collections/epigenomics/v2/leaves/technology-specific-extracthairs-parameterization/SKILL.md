---
name: technology-specific-extracthairs-parameterization
description: 'Use when preparing SpecHap input: an aligned BAM plus a variant VCF
  must be converted into per-read allele observations (a fragment file) for haplotype
  phasing.'
when_to_use_negative:
- The goal is haplotype assembly itself — SpecHap (the downstream spectral graph phasing
  step) handles that; this skill only produces its fragment input
- Variants include large structural rearrangements — SpecHap supports only SNPs and
  short insertions/deletions, so fragment extraction for haplotype-resolved structural
  variation is out of scope
- Ploidy is higher than diploid — the downstream SpecHap graph bisection model does
  not fit increased ploidy
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Downstream consumer of the extracted and sorted fragment file paired with
    the VCF
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified ExtractHAIRs utility that emits haplotype fragment
    information from aligned sequence and variants
  repo: https://github.com/vibansal/HapCUT2
- name: samtools/htslib
  role: Required dependency (bgzip/tabix) for variant and BED file compression and
    indexing
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:fa5aedb5ddddf06e4eeca120a23242c4207988524ef85a75b53dd8c298570b92
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
  - samtools/htslib
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# technology-specific-extracthairs-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates the haplotype fragment file that SpecHap consumes by running the modified ExtractHAIRs utility from HapCUT2 with technology-specific flags for WGS, Hi-C, 10X linked-reads, PacBio SMRT and Oxford Nanopore data, then sorting the fragment file to match the variant VCF order.

## When to use

Use when preparing SpecHap input: an aligned BAM plus a variant VCF must be converted into per-read allele observations (a fragment file) for haplotype phasing.

## When NOT to use

- The goal is haplotype assembly itself — SpecHap (the downstream spectral graph phasing step) handles that; this skill only produces its fragment input
- Variants include large structural rearrangements — SpecHap supports only SNPs and short insertions/deletions, so fragment extraction for haplotype-resolved structural variation is out of scope
- Ploidy is higher than diploid — the downstream SpecHap graph bisection model does not fit increased ploidy

## Inputs

- Aligned sequencing BAM file
- bgzipped, tabix-indexed, position-sorted VCF of heterozygous variants
- For 10X linked reads: whitelist barcodes for BarcodeExtract spanning-range BED generation

## Outputs

- Fragment file of per-read allele observations at heterozygous variants (unsorted fragment_file)
- Positionally sorted fragment file (sorted.frag), ordered consistently with the input VCF
- For 10X linked reads: barcode spanning-range BED file (bgzipped and tabix-indexed)

## How to apply

First verify the input VCF is bgzipped, tabix-indexed and sorted by genomic position, since the fragment file must be sorted in the same order to pair with the VCF. Run extractHAIRS (the modified ExtractHair utility from HapCUT2, built from submodules with pinned htslib commit 26229a3 and samtools commit 255f97d) with --bam pointing to the aligned BAM, --VCF to the variant file, and --out to the fragment file. Select the protocol-specific flag: --hic 1 for Hi-C, --10x 1 for 10X linked reads (also generating a barcode spanning-range BED via BarcodeExtract, then bgzip/tabix-indexing it), --pacbio 1 plus --ref for PacBio SMRT reads, or --ont for Oxford Nanopore reads. Finally sort the fragment file numerically with 'sort -n -k6' for 10X/Hi-C/new-format data or 'sort -n -k3' for paired-end NGS, PacBio SMRT or ONT data in the default format, producing sorted.frag.

## Related tools

- **SpecHap** (Downstream consumer of the extracted and sorted fragment file paired with the VCF) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified ExtractHAIRs utility that emits haplotype fragment information from aligned sequence and variants) — https://github.com/vibansal/HapCUT2
- **samtools/htslib** (Required dependency (bgzip/tabix) for variant and BED file compression and indexing) — https://github.com/samtools/htslib

## Examples

```
extractHAIRS --bam aligned.bam --VCF variants.vcf.gz --out fragment_file --hic 1 && sort -n -k6 fragment_file > sorted.frag
```

## Evaluation signals

- The produced fragment_file exists and is non-empty
- The fragment file is sorted in the same positional order as the input VCF so the two can be paired by SpecHap
- The VCF was verified as bgzipped, tabix-indexed and sorted by genomic position before extraction
- For 10X input, only fragments with white-listed barcodes are kept and the barcode spanning-range BED is bgzipped/tabix-indexed
- The correct technology flag (--hic 1, --10x 1, --pacbio 1 + --ref, or --ont) matches the sequencing protocol of the BAM

## Limitations

- The sort key differs by format: 'sort -n -k6' for 10X/Hi-C/new-format data versus 'sort -n -k3' for paired-end NGS, PacBio SMRT or ONT data in the default format; using the wrong key breaks pairing with the VCF
- Fragment extraction is only the precursor — downstream phasing results depend on the interval size parameter and no guarantee of optimal solutions is given
- Only SNPs, short insertions and deletions are supported; haplotype-resolved structural variation is not handled

## Evidence
