---
name: allele-specific-pet-assignment
description: Use when analyzing ChIA-PET bedpe data from a heterozygous sample (e.g.,
  GM12878 with Illumina Platinum Genomes phased genotypes) to determine which parental
  allele mediates each chromatin loop.
when_to_use_negative:
- The sample has no phased heterozygous genotypes available, or the dataset is homozygous/inbred
  — haplotype assignment of PETs is impossible without phased variants
- The goal is a standard (non-allele-specific) loop call from raw reads; the ChIA-PET2
  core pipeline (trimLinker, buildBedpe, removeDup, bedpe2Interaction) already handles
  that without genotype data
- PETs span no heterozygous sites, so they cannot be haplotype-assigned and would
  be excluded from allele-specific looping
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0102
tools:
- name: ChIA-PET2
  role: Provides bedpe2Phased to assign each PET to a parental haplotype from phased
    heterozygous variants and call allele-specific chromatin interactions
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bedtools
  role: pairToBed with '-type either' intersects the both-mapped bedpe against the
    phased genotype file to retain PETs overlapping heterozygous sites
  repo: https://github.com/arq5x/bedtools2
- name: bwa
  role: Upstream short-read alignment of ChIA-PET reads that feeds the bedpe used
    for allele assignment
  repo: lh3/bwa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:272878edc7313b5f1e2b74852ee5015de847f67e0c6e459a8db556d6ac5e91a7
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
  - bwa
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# allele-specific-pet-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign ChIA-PET paired-end tags (PETs) to parental haplotypes using phased genotype data, enabling allele-specific chromatin interaction calling with ChIA-PET2's bedpe2Phased workflow at imprinted loci such as H19/IGF2.

## When to use

Use when analyzing ChIA-PET bedpe data from a heterozygous sample (e.g., GM12878 with Illumina Platinum Genomes phased genotypes) to determine which parental allele mediates each chromatin loop.

## When NOT to use

- The sample has no phased heterozygous genotypes available, or the dataset is homozygous/inbred — haplotype assignment of PETs is impossible without phased variants
- The goal is a standard (non-allele-specific) loop call from raw reads; the ChIA-PET2 core pipeline (trimLinker, buildBedpe, removeDup, bedpe2Interaction) already handles that without genotype data
- PETs span no heterozygous sites, so they cannot be haplotype-assigned and would be excluded from allele-specific looping

## Inputs

- Deduplicated ChIA-PET bedpe file (e.g., CTCF GM12878), preferably filtered to retain only PETs with both ends sequence-mapped
- Phased genotype file for the same sample, 5 tab-delimited columns: chr, start, end, allele1, allele2 (e.g., from the Illumina Platinum Genomes Project)
- bedtools (v2.24.0+) for bedpe/genotype intersection

## Outputs

- Bedpe with each allele-spanning PET assigned to a parental haplotype (phased bedpe from bedpe2Phased)
- Count of allele-assigned PETs (expected >6 million, ~3.15% of valid PETs in GM12878 CTCF)
- Allele-specific chromatin loop calls (e.g., maternal- vs paternal-specific loops at H19/IGF2)

## How to apply

Start from the deduplicated CTCF GM12878 bedpe file and filter it with 'grep -v "*"' so only PET pairs with both ends sequence-mapped are retained. Intersect the both-mapped bedpe with the phased genotype file (5 tab-delimited columns: chr, start, end, allele1, allele2) using bedtools pairToBed with '-type either' to keep PETs overlapping any heterozygous site. Run ChIA-PET2's bedpe2Phased to assign each PET to one of the two parental haplotypes based on the phased heterozygous variants it spans. Confirm that more than 6 million PETs (about 3.15% of valid PETs in the GM12878 CTCF dataset) are allele-assigned; a much lower rate suggests the genotype file or intersection step failed. This is the mechanism by which ChIA-PET2 supports allele-specific interaction calling, e.g., recovering five maternal-specific loops at the H19/IGF2 locus with an attenuated maternal IGF2 loop.

## Related tools

- **ChIA-PET2** (Provides bedpe2Phased to assign each PET to a parental haplotype from phased heterozygous variants and call allele-specific chromatin interactions) — https://github.com/GuipengLi/ChIA-PET2
- **bedtools** (pairToBed with '-type either' intersects the both-mapped bedpe against the phased genotype file to retain PETs overlapping heterozygous sites) — https://github.com/arq5x/bedtools2
- **bwa** (Upstream short-read alignment of ChIA-PET reads that feeds the bedpe used for allele assignment) — lh3/bwa

## Examples

```
grep -v '*' dedup.bedpe > both_mapped.bedpe && bedtools pairToBed -type either -a both_mapped.bedpe -b phased_genotypes.txt && bedpe2Phased both_mapped.bedpe phased_genotypes.txt
```

## Evaluation signals

- Filter check: the both-mapped bedpe contains no PET pairs with '*' (unmapped) ends after the grep -v '*' step
- Assignment rate: >6 million PETs (~3.15% of valid PETs in the GM12878 CTCF dataset) receive a haplotype assignment
- Schema check: the phased genotype file has exactly 5 tab-delimited columns (chr, start, end, allele1, allele2) before intersection
- Biological validation: at the H19/IGF2 imprinted locus, five maternal-specific loops are recovered and the IGF2 loop is attenuated on the maternal chromosome with the opposite pattern on the paternal chromosome

## Limitations

- Only PETs spanning phased heterozygous variants can be allele-assigned; the fraction depends on genotype density (~3.15% of valid PETs in the published GM12878 analysis)
- ChIA-PET2 calls chromatin loops based on ChIP-Seq-like peaks, so allele-specific loop detection is bounded by peak-calling sensitivity
- The article does not state a ChIA-PET2 software version or versions for dependencies (bwa, samtools, bedtools, MACS2, g++)

## Evidence

- [abstract] ChIA-PET2 can use phased genotype data to call allele-specific chromatin interactions: "In addition, ChIA-PET2 can use phased genotype data to call allele-specific chromatin interactions."
- [discussion] Phased genotype data for GM12878 genome were extracted from the Illumina Platinum Genomes Project: "Phased genotype data for GM12878 genome were extracted from the Illumina Platinum Genomes Project"
- [readme] In addition, ChIA-PET2 can use phased genotype data to call allele-specific chromatin interactions.: "In addition, ChIA-PET2 can use phased genotype data to call allele-specific chromatin interactions. We applied ChIA-PET2 to different ChIA-PET datasets, demonstrating its significant improved"
- [intro] none of those tools support allele-specific ChIA-PET analysis or comprehensive data quality control measures: "none of those tools support allele-specific ChIA-PET analysis or comprehensive data quality control measures"
