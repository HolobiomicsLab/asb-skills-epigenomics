---
name: imprinting-locus-allele-specific-loop-interpretation
description: Use when a deduplicated ChIA-PET bedpe (e.g., GM12878 CTCF) is available
  together with phased heterozygous genotypes, and the goal is to determine which
  chromatin loops at an imprinted locus are maternal- versus paternal-specific.
when_to_use_negative:
- No phased heterozygous genotype data exist for the sample, so PETs cannot be assigned
  to a parental haplotype and the allele-specific step is meaningless.
- The goal is allele-aggregated loop calling from raw reads (linker trimming, alignment,
  duplicate removal, peak calling) — use the standard ChIA-PET2 pipeline workflow
  instead of the allele-specific interpretation step.
- The locus of interest has no heterozygous sites spanned by PETs, so no allele assignment
  can be made there.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Provides the bedpe2Phased step that assigns each PET to a parental haplotype
    using phased genotype data, enabling allele-specific chromatin interaction calling
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bedtools
  role: pairToBed with '-type either' intersects the both-mapped bedpe with the phased
    heterozygous genotype intervals
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:d9c5ffd85917b5517db029846f44d8b67dd053305fba3e5b0a88c12a56e28db2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GuipengLi/ChIA-PET2
  repo_url: https://github.com/GuipengLi/ChIA-PET2
  tools:
  - ChIA-PET2
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# imprinting-locus-allele-specific-loop-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets allele-specific chromatin loops at imprinted loci (e.g., H19/IGF2 in GM12878) by combining ChIA-PET2's phased-genotype loop calling with bedtools-based filtering of both-mapped PETs against Illumina Platinum Genomes phased variants.

## When to use

Use when a deduplicated ChIA-PET bedpe (e.g., GM12878 CTCF) is available together with phased heterozygous genotypes, and the goal is to determine which chromatin loops at an imprinted locus are maternal- versus paternal-specific.

## When NOT to use

- No phased heterozygous genotype data exist for the sample, so PETs cannot be assigned to a parental haplotype and the allele-specific step is meaningless.
- The goal is allele-aggregated loop calling from raw reads (linker trimming, alignment, duplicate removal, peak calling) — use the standard ChIA-PET2 pipeline workflow instead of the allele-specific interpretation step.
- The locus of interest has no heterozygous sites spanned by PETs, so no allele assignment can be made there.

## Inputs

- Deduplicated ChIA-PET bedpe file (e.g., GM12878 CTCF, ENCODE GSM970213 / GSE72816)
- Phased genotype file for GM12878 (5 tab-delimited columns: chr, start, end, allele1, allele2) from the Illumina Platinum Genomes Project
- ChIA-PET2 installation (bedpe2Phased module) and bedtools v2.24.0+

## Outputs

- Both-mapped bedpe (PETs with both ends sequence-mapped)
- PETs overlapping heterozygous phased sites (pairToBed '-type either' output)
- Allele-assigned (haplotype-tagged) PET set with >6 million PETs (~3.15% of valid PETs)
- Allele-specific chromatin loop calls at the H19/IGF2 locus, stratified by parental haplotype

## How to apply

Filter the deduplicated CTCF GM12878 bedpe with 'grep -v "*"' so only PETs with both ends sequence-mapped are retained. Intersect this bedpe with the phased genotype file (5 tab-delimited columns: chr, start, end, allele1, allele2) using bedtools pairToBed with '-type either' to keep PETs overlapping any heterozygous site. Run ChIA-PET2's bedpe2Phased on the filtered bedpe to assign each PET to one of the two parental haplotypes via the phased heterozygous variants it spans. Verify that more than 6 million PETs (about 3.15% of valid PETs) receive an allele assignment before interpreting results. At the H19/IGF2 imprinted locus, expect five maternal-specific loops with anchors close to the H19 gene, while the loop to the IGF2 region is absent or greatly attenuated on the maternal chromosome, with the opposite pattern on the paternal chromosome.

## Related tools

- **ChIA-PET2** (Provides the bedpe2Phased step that assigns each PET to a parental haplotype using phased genotype data, enabling allele-specific chromatin interaction calling) — https://github.com/GuipengLi/ChIA-PET2
- **bedtools** (pairToBed with '-type either' intersects the both-mapped bedpe with the phased heterozygous genotype intervals) — https://github.com/arq5x/bedtools2

## Examples

```
grep -v "*" ctcf_gm12878.bedpe | bedtools pairToBed -type either -a - -b phased_genotypes.txt > het_overlapping.bedpe
```

## Evaluation signals

- After 'grep -v "*"', the bedpe contains only PET pairs where both ends are sequence-mapped.
- More than 6 million PETs — about 3.15% of valid PETs — are assigned to a parental allele by bedpe2Phased.
- At the H19/IGF2 imprinted locus, exactly five maternal-specific chromatin loops are recovered with anchors close to the H19 gene.
- The loop to the IGF2 region is absent or greatly attenuated on the maternal chromosome, with the opposite (paternal-enriched) pattern, matching known H19/IGF2 imprinting.
- Allele assignments are consistent with the Illumina Platinum Genomes phased haplotypes used as input.

## Limitations

- Loop calling depends on ChIP-Seq-like peak calling sensitivity: ChIA-PET2 assumes loop anchors are located at ChIP-Seq-like peaks.
- Only a few examples of the transitivity, redundancy and robustness of chromatin loops were provided in the article.
- No software version is stated for ChIA-PET2 or for its required dependencies (bwa, samtools, bedtools, MACS2, g++), which may affect reproducibility.

## Evidence
