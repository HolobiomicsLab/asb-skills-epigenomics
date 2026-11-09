---
name: sequencing-fragment-extraction
description: Use when preparing fragment files for haplotype assembly tools (SpecHap,
  HapCUT2, FastHare, DCHap, ReFHap) from BAM/CRAM alignments and bgzipped VCF variant
  calls across diverse sequencing platforms.
when_to_use_negative:
- The task is haplotype phasing of polyploid organisms — SpecHap's graph bisection
  model and ExtractHair's diploid fragment assumptions do not fit increased ploidy
- The goal is haplotype-resolved structural variation detection — the fragment/linkage
  graph supports only SNPs and short insertions/deletions
- Variants and genotypes span multiple individuals — HapCUT2/ExtractHair-based processing
  expects VCF input for a single diploid individual
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ExtractHair
  role: Modified HapCUT2 utility shipped in SpecHap's hair-src; converts alignments
    + variant VCF into fragment files with technology-specific flags (--hic 1, --10x
    1, --pacbio 1, --ont + --ref)
  repo: https://github.com/vibansal/HapCUT2
- name: SpecHap
  role: Provides the pinned ExtractHair submodules, BarcodeExtract, and the phasing
    engine that consumes the sorted fragment file
  repo: https://github.com/deepomicslab/SpecHap
- name: htslib
  role: Required dependency (pinned at 26229a3) providing bgzip/tabix support used
    by fragment extraction and VCF preprocessing
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:b00e9976cbe69ac955c60ac86a866ea8f4c9793406a1e669cc6934fc1db01048
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/vibansal/HapCUT2
  tools:
  - DCHap
  - ExtractHair
  - FastHare
  - HapCUT
  - HapCUT2
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# sequencing-fragment-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-fragment allele observations (read-backed haplotype fragments) from aligned reads and variant calls using the modified ExtractHair utility from SpecHap, with technology-specific flags for NGS, Hi-C, 10x linked-reads, PacBio SMRT and ONT data, then sort fragments for downstream haplotype assembly.

## When to use

Use when preparing fragment files for haplotype assembly tools (SpecHap, HapCUT2, FastHare, DCHap, ReFHap) from BAM/CRAM alignments and bgzipped VCF variant calls across diverse sequencing platforms.

## When NOT to use

- The task is haplotype phasing of polyploid organisms — SpecHap's graph bisection model and ExtractHair's diploid fragment assumptions do not fit increased ploidy
- The goal is haplotype-resolved structural variation detection — the fragment/linkage graph supports only SNPs and short insertions/deletions
- Variants and genotypes span multiple individuals — HapCUT2/ExtractHair-based processing expects VCF input for a single diploid individual

## Inputs

- Position-sorted, bgzipped VCF of variant calls (e.g. trio-phased 1000 Genomes variant sets for NA12878/NA19240)
- Aligned sequencing reads (BAM/CRAM) from NGS, 10x linked-reads, Hi-C, PacBio SMRT or ONT
- For 10x: bgzipped/tabixed barcode-spanning bed file produced by BarcodeExtract and the white-listed barcode set

## Outputs

- Technology-specific fragment file (allelic observations per read/linked fragment)
- Sorted fragment file ordered consistently with the position-sorted bgzipped VCF ('sort -n -k6' or 'sort -n -k3' depending on format)

## How to apply

Build the modified ExtractHair utility shipped in SpecHap's hair-src by pinning submodules (htslib at 26229a3, samtools at 255f97d) and running make; htslib and arpack-ng (built after BLAS/LAPACK with bootstrap/configure --enable-icb) must be installed first. For each technology, run extractHAIRS with the matching flag: --hic 1 for Hi-C proximity-ligation reads, --10x 1 plus a bgzipped/tabixed barcode-spanning bed from BarcodeExtract for 10x linked reads, --pacbio 1 for PacBio SMRT, and --ont plus --ref for Oxford Nanopore. For 10x, filter reads so only fragments with white-listed barcodes are kept. Sort fragment files consistently with the position-sorted bgzipped VCF: 'sort -n -k6' for 10x/Hi-C/new-format input and 'sort -n -k3' for paired-end NGS, PacBio and ONT default format. The resulting fragment file plus the trio-phased 1000 Genomes variant VCF are the inputs every phasing method consumes, so identical fragment files across methods make CPU time, memory, AN50, switch and mismatch error rates comparable.

## Related tools

- **ExtractHair** (Modified HapCUT2 utility shipped in SpecHap's hair-src; converts alignments + variant VCF into fragment files with technology-specific flags (--hic 1, --10x 1, --pacbio 1, --ont + --ref)) — https://github.com/vibansal/HapCUT2
- **SpecHap** (Provides the pinned ExtractHair submodules, BarcodeExtract, and the phasing engine that consumes the sorted fragment file) — https://github.com/deepomicslab/SpecHap
- **htslib** (Required dependency (pinned at 26229a3) providing bgzip/tabix support used by fragment extraction and VCF preprocessing) — https://github.com/samtools/htslib

## Examples

```
extractHAIRS --pacbio 1 --vcf <bgzipped.vcf.gz> --bam <aligned.bam>
```

## Evaluation signals

- Submodules are pinned correctly before building ExtractHair: htslib at 26229a3 and samtools at 255f97d, then 'make' in hair-src completes without errors
- Fragment file is position-sorted with the correct key for its format: 'sort -n -k6' for 10x/Hi-C/new-format input, 'sort -n -k3' for paired-end NGS, PacBio and ONT default format
- For 10x linked-reads, only fragments whose barcodes appear in the white-list are present in the output fragment file
- Every phasing method (SpecHap, FastHare, HapCUT, HapCUT2, DCHap, ReFHap) is run on the same fragment file and trio-phased 1000 Genomes variant set, so benchmark_results.tsv cells report AN50, phased variant count, switch error rate, mismatch error rate, CPU time and peak memory

## Limitations

- ExtractHair/SpecHap supports only SNPs and short insertions/deletions; haplotype-resolved structural variation is not supported
- SpecHap provides no guarantee of optimal solutions, and phasing results depend on the user-defined chromosome interval size — larger intervals may reduce efficiency
- No software version is stated for SpecHap, so reproducibility relies on the pinned submodule commits (htslib 26229a3, samtools 255f97d)

## Evidence
