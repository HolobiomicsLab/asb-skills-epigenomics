---
name: long-read-phasing-workflow-configuration
description: Use when phasing diploid individuals from PacBio SMRT, ONT, Hi-C, 10x
  linked-reads, or paired-end NGS data and you need to build SpecHap, extract fragments
  with ExtractHair, and run the phaser on position-sorted bgzipped VCF plus fragment
  files.
when_to_use_negative:
- Ploidy is higher than diploid — SpecHap's graph bisection model does not fit increased
  ploidy
- The goal is haplotype-resolved structural variation — SIH tools including SpecHap
  support only SNPs and short insertions/deletions
- Input fragments are not sorted consistently with the position-sorted bgzipped VCF
  — sort order must match before running
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: spectral graph theory-based diploid phaser run on fragment + VCF inputs with
    platform-specific flags
  repo: https://github.com/deepomicslab/SpecHap
- name: ExtractHair (from HapCUT2)
  role: modified fragment extraction utility built from pinned submodules to convert
    alignments into fragment files
  repo: https://github.com/vibansal/HapCUT2
- name: htslib
  role: prerequisite library for SpecHap and ExtractHair; provides bgzip/tabix for
    VCF and barcode BED
  repo: https://github.com/samtools/htslib
- name: arpack-ng
  role: ARPACK eigensolver dependency for SpecHap's spectral calculations, built with
    --enable-icb after BLAS/LAPACK
  repo: https://github.com/opencollab/arpack-ng
- name: HapCUT2
  role: comparator phaser run on the same fragment files for cross-tool benchmarking
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:b02aba8c9ff9f000f021631238e63ac34fa97479861568a2196c2e99a32ffb0d
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
  - DCHap
  - ExtractHair (from HapCUT2)
  - FastHare
  - HapCUT
  - HapCUT2
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# long-read-phasing-workflow-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configures a haplotype phasing workflow (SpecHap) for long-read and multi-platform WGS data: building SpecHap and its ExtractHair fragment extractor with pinned htslib (26229a3) and samtools (255f97d) submodules, generating technology-specific fragment files, and running phasing against bgzipped VCF inputs.

## When to use

Use when phasing diploid individuals from PacBio SMRT, ONT, Hi-C, 10x linked-reads, or paired-end NGS data and you need to build SpecHap, extract fragments with ExtractHair, and run the phaser on position-sorted bgzipped VCF plus fragment files.

## When NOT to use

- Ploidy is higher than diploid — SpecHap's graph bisection model does not fit increased ploidy
- The goal is haplotype-resolved structural variation — SIH tools including SpecHap support only SNPs and short insertions/deletions
- Input fragments are not sorted consistently with the position-sorted bgzipped VCF — sort order must match before running

## Inputs

- bgzipped, position-sorted VCF of heterozygous variant calls
- technology-specific fragment files produced by extractHAIRS from BAM/ alignments
- barcode-spanning BED (bgzipped/tabixed) for 10x linked-reads
- reference FASTA (--ref) for PacBio/ONT fragment extraction

## Outputs

- phased VCF (--out) with haplotype assignments per variant
- sorted fragment files consistent with VCF ordering
- CPU time and peak memory measurements per run (for benchmarking)

## How to apply

Build SpecHap from source with cmake/make, after installing prerequisites arpack-ng (bootstrap then configure --enable-icb, requiring BLAS/LAPACK) and htslib (autoheader/autoconf, configure, make install). Build the modified ExtractHair utility from pinned submodules (htslib at 26229a3, samtools at 255f97d) with make in hair-src. Generate fragment files with extractHAIRS using technology-specific flags: --hic 1 for Hi-C, --10x 1 with a bgzipped/tabixed barcode-spanning BED from BarcodeExtract for 10x linked reads, and --pacbio 1 or --ont plus --ref for PacBio SMRT / Oxford Nanopore. Sort fragment files consistently with the position-sorted bgzipped VCF — 'sort -n -k6' for 10x/Hi-C/new-format input, 'sort -n -k3' for paired-end NGS, PacBio and ONT default format — then run SpecHap with --vcf, --frag, --out and the platform flag (e.g. --pacbio).

## Related tools

- **SpecHap** (spectral graph theory-based diploid phaser run on fragment + VCF inputs with platform-specific flags) — https://github.com/deepomicslab/SpecHap
- **ExtractHair (from HapCUT2)** (modified fragment extraction utility built from pinned submodules to convert alignments into fragment files) — https://github.com/vibansal/HapCUT2
- **htslib** (prerequisite library for SpecHap and ExtractHair; provides bgzip/tabix for VCF and barcode BED) — https://github.com/samtools/htslib
- **arpack-ng** (ARPACK eigensolver dependency for SpecHap's spectral calculations, built with --enable-icb after BLAS/LAPACK) — https://github.com/opencollab/arpack-ng
- **HapCUT2** (comparator phaser run on the same fragment files for cross-tool benchmarking) — https://github.com/vibansal/HapCUT2

## Examples

```
SpecHap --vcf variants.gz.vcf --frag fragments.sorted.txt --out phased.vcf --pacbio
```

## Evaluation signals

- All pinned submodule commits (htslib 26229a3, samtools 255f97d) are checked out before building ExtractHair; build succeeds with make in hair-src
- Fragment files are sorted consistently with the position-sorted bgzipped VCF: 'sort -n -k6' for 10x/Hi-C/new-format, 'sort -n -k3' for paired-end NGS, PacBio and ONT default format
- Phased VCF output contains assignments for every input variant position and can be evaluated against 1000 Genomes trio-phased ground truth (AN50, phased variant count, switch and mismatch error rates)
- Runtime/memory sanity: on ~44x NA12878 PacBio SMRT data, SpecHap completes in ~13 CPU minutes with ~120MB peak memory; large deviations suggest misconfiguration
- SpecHap --help lists supported options and platform flags (--hic, --10x, --pacbio, --ont) resolve without errors

## Limitations

- No guarantee of optimal solutions; interval size parameter affects the phasing result and larger interval sizes may reduce SpecHap's efficiency
- No software version is stated for SpecHap, so exact reproducibility depends on the repository state at time of use
- A comprehensive analysis of technology-specific error for Hi-C and 10x linked-reads phasing quality has not been conducted

## Evidence

- [methods] mkdir build
cd build
cmake ..
make && make install: "mkdir build
cd build
cmake ..
make && make install"
- [results] The trio-phased haplotype from the 1000 Genomes Project was taken as the 'ground truth': [evidence span withheld — anchor status: not_found]
