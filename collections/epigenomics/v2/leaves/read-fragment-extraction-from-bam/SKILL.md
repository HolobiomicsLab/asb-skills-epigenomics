---
name: read-fragment-extraction-from-bam
description: Use when a diploid phasing pipeline needs a fragment file from sequencing
  reads (WGS paired-end, Hi-C, 10X linked-reads, PacBio SMRT, or ONT) before running
  SpecHap graph construction.
when_to_use_negative:
- Variant calls are not heterozygous SNVs/short indels — SpecHap/ExtractHAIRs supports
  only SNPs, short insertions and deletions, not structural variants
- Ploidy is higher than diploid — SpecHap's graph bisection model does not fit increased
  ploidy
- You only need haplotype assembly without fragment preprocessing — i.e., a fragment
  file already exists and is sorted consistently with the VCF
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Downstream consumer of the sorted fragment file paired with the VCF for spectral-graph-based
    diploid phasing
  repo: https://github.com/deepomicslab/SpecHap
- name: ExtractHAIRs (modified ExtractHair from HapCUT2)
  role: Generates haplotype fragment information from aligned sequence and variants;
    built from SpecHap submodules with pinned htslib (26229a3) and samtools (255f97d)
    commits
  repo: https://github.com/vibansal/HapCUT2
- name: htslib
  role: Required library dependency for fragment extraction and for bgzip/tabix indexing
    of the VCF and barcode BED files
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:2d23ada5340285f3c7d549f702551f5d9c7c117980555b1ea913badfdc763bad
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
  - ExtractHAIRs (modified ExtractHair from HapCUT2)
  - HapCUT2
  - SpecHap
  - htslib
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# read-fragment-extraction-from-bam

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-read haplotype-informative allele observations from an aligned BAM and a variant VCF using the modified ExtractHAIRs utility (from HapCUT2) shipped as a SpecHap submodule, producing a sorted fragment file that pairs with the VCF for SpecHap phasing.

## When to use

Use when a diploid phasing pipeline needs a fragment file from sequencing reads (WGS paired-end, Hi-C, 10X linked-reads, PacBio SMRT, or ONT) before running SpecHap graph construction.

## When NOT to use

- Variant calls are not heterozygous SNVs/short indels — SpecHap/ExtractHAIRs supports only SNPs, short insertions and deletions, not structural variants
- Ploidy is higher than diploid — SpecHap's graph bisection model does not fit increased ploidy
- You only need haplotype assembly without fragment preprocessing — i.e., a fragment file already exists and is sorted consistently with the VCF

## Inputs

- Aligned BAM file of sequencing reads
- bgzipped, tabix-indexed VCF of heterozygous variant calls sorted by genomic position
- Sequencing technology identifier (WGS paired-end, Hi-C, 10X linked-reads, PacBio SMRT, ONT)
- For 10X linked reads: barcode whitelist and a barcode spanning-range BED file generated via BarcodeExtract

## Outputs

- Unsorted haplotype fragment file (fragment_file) of per-read allele observations
- Positionally sorted fragment file (sorted.frag) ordered consistently with the input VCF

## How to apply

Verify the input VCF is bgzipped, tabix-indexed, and sorted by genomic position, because the fragment file must later be sorted in the same order. Run extractHAIRS with --bam pointing to the aligned BAM, --VCF pointing to the variant file, and --out fragment_file to emit per-read allele observations at heterozygous variants. Choose technology-specific flags: --hic 1 for Hi-C, --10x 1 for 10X linked reads, --pacbio 1 plus --ref for PacBio SMRT reads, and --ont for Oxford Nanopore reads; for 10X data also generate a barcode spanning-range BED with BarcodeExtract and bgzip/tabix-index it. Sort the fragment file numerically with 'sort -n -k6' for 10X/Hi-C/new-format data or 'sort -n -k3' for paired-end NGS, PacBio SMRT, and ONT data in the default format, producing sorted.frag. The rationale is that SpecHap consumes the fragment file paired with the VCF, so positional ordering between the two must match.

## Related tools

- **SpecHap** (Downstream consumer of the sorted fragment file paired with the VCF for spectral-graph-based diploid phasing) — https://github.com/deepomicslab/SpecHap
- **ExtractHAIRs (modified ExtractHair from HapCUT2)** (Generates haplotype fragment information from aligned sequence and variants; built from SpecHap submodules with pinned htslib (26229a3) and samtools (255f97d) commits) — https://github.com/vibansal/HapCUT2
- **htslib** (Required library dependency for fragment extraction and for bgzip/tabix indexing of the VCF and barcode BED files) — https://github.com/samtools/htslib

## Examples

```
extractHAIRS --bam aligned.bam --VCF variants.vcf.gz --out fragment_file --hic 1 && sort -n -k6 fragment_file > sorted.frag
```

## Evaluation signals

- The produced fragment_file exists and is non-empty
- The fragment file is sorted in the same positional order as the input VCF so the two can be paired by SpecHap
- The VCF was verified as bgzipped, tabix-indexed, and coordinate-sorted before extraction
- The technology-specific extractHAIRS flag used (e.g. --hic 1, --10x 1, --pacbio 1 --ref, --ont) matches the actual sequencing protocol of the BAM
- For 10X data, the barcode spanning-range BED is present, bgzipped, and tabix-indexed alongside the fragment file

## Limitations

- ExtractHAIRs/SpecHap supports only SNPs, short insertions and deletions; haplotype-resolved structural variation and genome rearrangements are not handled
- SpecHap is diploid-only (like HapCUT2 at present); the graph bisection model does not fit increased ploidy
- Fragment file sort key differs by data type and format: 'sort -n -k6' for 10X/Hi-C/new-format, 'sort -n -k3' for paired-end NGS/PacBio/ONT default format — using the wrong key breaks VCF pairing
- A comprehensive analysis of technology-specific error on Hi-C and 10X linked-reads phasing quality has not been conducted, so fragment quality for these protocols may vary

## Evidence

- [methods] Fragment extraction is performed with a refined version of ExtractHAIRs: [evidence span withheld — anchor status: not_found]
