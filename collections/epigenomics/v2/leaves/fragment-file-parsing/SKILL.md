---
name: fragment-file-parsing
description: Use when you have a fragment file (from 10X/Hi-C linked reads, paired-end
  NGS, PacBio SMRT or ONT data) plus a VCF of heterozygous variants and need to build
  SpecHap's linkage graph for spectral-based haplotype assembly.
when_to_use_negative:
- Ploidy is higher than diploid — the article states the current graph bisection model
  might not fit when ploidy increases
- The goal is haplotype-resolved structural variation — the current linkage graph
  supports only SNPs and short insertions/deletions, not genome rearrangements
- You already have a phased haplotype or VCF with phase blocks and do not need fragment-level
  linkage reconstruction
edam_operation: http://edamontology.org/operation_0487
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Downstream consumer of the parsed linkage graph; performs spectral (Fiedler-Vector)
    allele partitioning on it
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified ExtractHair utility used to generate and parse the
    fragment file into the linkage graph
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:e7e3564c31d9ba2197e209aea139fcc36cd7d40ffaba26405ea2bfabb2f22cfa
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
  edam_operation: http://edamontology.org/operation_0487
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fragment-file-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parses the position-sorted fragment file produced by ExtractHair (modified from HapCUT2) and converts fragment-observed heterozygous variant loci into a weighted undirected linkage graph whose edge weights are truncated log-likelihood ratios (LLRs) computed from fragment Phred probabilities, as used by SpecHap for spectral phasing.

## When to use

Use when you have a fragment file (from 10X/Hi-C linked reads, paired-end NGS, PacBio SMRT or ONT data) plus a VCF of heterozygous variants and need to build SpecHap's linkage graph for spectral-based haplotype assembly.

## When NOT to use

- Ploidy is higher than diploid — the article states the current graph bisection model might not fit when ploidy increases
- The goal is haplotype-resolved structural variation — the current linkage graph supports only SNPs and short insertions/deletions, not genome rearrangements
- You already have a phased haplotype or VCF with phase blocks and do not need fragment-level linkage reconstruction

## Inputs

- Position-sorted fragment file from ExtractHair (modified from HapCUT2), consistent with VCF variant order
- bgzipped VCF of heterozygous variant calls (SpecHap requires at least a fragment file and a bgzipped VCF)
- Per-base Phred quality probabilities carried in the fragment file

## Outputs

- linkage_graph.tsv: tab-delimited weighted undirected linkage graph (node = heterozygous variant locus, edge = truncated log-likelihood-ratio weight)

## How to apply

First confirm the extractHAIRS output fragment file is sorted consistently with the VCF order: use 'sort -n -k6' for 10X/Hi-C or new-format fragments, and 'sort -n -k3' for paired-end NGS, PacBio SMRT or Oxford Nanopore default format. Define the heterozygous variant loci covered by fragments as the nodes of an undirected graph. For every pair of loci co-observed on the same fragment, compute the log-likelihood ratios of the two conflicting haplotype-pair assignments (H1 vs H2) from the fragment's base Phred probabilities. Truncate each LLR — assigning max(log(p(H1)/p(H2)), 0) for conflicting pairs — and use this truncated value as the edge weight. Emit the resulting weighted undirected graph (node = variant locus, edge = truncated LLR weight) as a tab-delimited linkage_graph.tsv, which SpecHap then consumes for Fiedler-Vector-guided allele partitioning.

## Related tools

- **SpecHap** (Downstream consumer of the parsed linkage graph; performs spectral (Fiedler-Vector) allele partitioning on it) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified ExtractHair utility used to generate and parse the fragment file into the linkage graph) — https://github.com/vibansal/HapCUT2

## Examples

```
sort -n -k6 fragments.txt > fragments.sorted.txt && SpecHap --fragments fragments.sorted.txt --vcf variants.vcf.gz --output phased.txt
```

## Evaluation signals

- linkage_graph.tsv exists and is tab-delimited
- Every edge weight is a finite numeric log-likelihood-ratio value (truncated at 0 for conflicting haplotype pairs)
- The graph's node set exactly matches the heterozygous loci covered by fragments in the input fragment file
- The fragment file was sorted consistently with VCF order (correct sort key: -k6 for 10X/Hi-C, -k3 for paired-end NGS/PacBio/ONT)
- Edges appear only between loci co-observed on the same fragment

## Limitations

- Phasing results depend on the user-defined interval size used by SpecHap's divide-and-conquer splitting; larger interval size might reduce efficiency, and no guarantee of optimal solutions is provided
- Only SNPs, short insertions and deletions are supported; a refined linkage graph incorporating genome rearrangements is still expected
- No software version is stated for SpecHap, so reproduction may be version-sensitive
- The influence of technology-specific error (Hi-C, 10X linked-reads) on phasing quality has not been comprehensively analyzed

## Evidence

- [intro] SpecHap assembles haplotype efficiently by transforming diploid SIH into a linear algebra problem using spectral graph theory: "SpecHap assembles haplotype efficiently by transforming diploid SIH into a linear algebra problem using spectral graph theory"
