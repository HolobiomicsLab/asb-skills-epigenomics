---
name: fiedler-vector-allele-partitioning
description: Use when assembling diploid haplotypes (SIH) from WGS, Hi-C, 10X linked-reads,
  PacBio SMRT or ONT fragment files plus a bgzipped, tabix-indexed VCF, and you want
  a fast spectral alternative to MEC/max-cut optimization.
when_to_use_negative:
- The organism is polyploid — the current graph bisection model assumes diploidy and
  'might not fit when the ploidy increases'
- The goal is haplotype-resolved structural variation — SpecHap supports only SNPs,
  short insertions and deletions, and a refined linkage graph incorporating genome
  rearrangements is still expected
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: 'Main phasing tool: builds the linkage graph, eigendecomposes it via ARPACK,
    partitions alleles by Fiedler-vector sign, and writes the phased VCF'
  repo: https://github.com/deepomicslab/SpecHap
- name: ARPACK (arpack-ng)
  role: Dependency performing the eigendecomposition of the linkage graph to obtain
    the Fiedler vector
  repo: https://github.com/opencollab/arpack-ng
- name: htslib
  role: Required library supporting reading of the bgzipped VCF input
  repo: https://github.com/samtools/htslib
- name: ExtractHair
  role: Modified HapCUT2 utility used for fragment processing before phasing
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:b3f462bfbff796e4e061b88b30f79d1643573e9442bebc12483c3ee2b7719fd0
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
  - ARPACK (arpack-ng)
  - ExtractHair
  - SpecHap
  - htslib
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# fiedler-vector-allele-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

SpecHap's spectral-graph phasing routine: it builds a linkage graph from fragment data, computes the Fiedler vector via ARPACK eigendecomposition, and partitions alleles into two haplotypes by the sign of the Fiedler vector, recursing on sub-blocks when the vector does not directly resolve the haplotype.

## When to use

Use when assembling diploid haplotypes (SIH) from WGS, Hi-C, 10X linked-reads, PacBio SMRT or ONT fragment files plus a bgzipped, tabix-indexed VCF, and you want a fast spectral alternative to MEC/max-cut optimization.

## When NOT to use

- The organism is polyploid — the current graph bisection model assumes diploidy and 'might not fit when the ploidy increases'
- The goal is haplotype-resolved structural variation — SpecHap supports only SNPs, short insertions and deletions, and a refined linkage graph incorporating genome rearrangements is still expected

## Inputs

- bgzipped, tabix-indexed, position-sorted VCF of heterozygous variant calls
- fragment file sorted in the same variant order (10X/Hi-C/new-format: sort -n -k6; paired-end NGS, PacBio SMRT or ONT default: sort -n -k3)

## Outputs

- Phased VCF written to the path supplied via --out

## How to apply

First verify the input VCF is bgzipped, tabix-indexed, and sorted by position, and sort the fragment file consistently ('sort -n -k6 in.frag > sorted.frag' for 10X/Hi-C/new-format fragments, or 'sort -n -k3 in.frag > sorted.frag' for paired-end NGS, PacBio SMRT and ONT default format). Invoke SpecHap in phasing mode with the platform flag (e.g. --pacbio for PacBio SMRT reads) so the linkage graph is eigendecomposed through the ARPACK dependency and alleles are partitioned by the sign of the Fiedler vector, which acts as a min-cut heuristic on the linkage graph. When the Fiedler vector categorizes variants into two sub-blocks rather than directly resolving the haplotype, partition the variants recursively until the Fiedler vector yields a deducible haplotype; prune variants when a sub-block containing only two variants cannot be resolved. Chromosomes are divided into user-defined intervals as part of the divide-and-conquer strategy, and the phased VCF is written to the path given by --out.

## Related tools

- **SpecHap** (Main phasing tool: builds the linkage graph, eigendecomposes it via ARPACK, partitions alleles by Fiedler-vector sign, and writes the phased VCF) — https://github.com/deepomicslab/SpecHap
- **ARPACK (arpack-ng)** (Dependency performing the eigendecomposition of the linkage graph to obtain the Fiedler vector) — https://github.com/opencollab/arpack-ng
- **htslib** (Required library supporting reading of the bgzipped VCF input) — https://github.com/samtools/htslib
- **ExtractHair** (Modified HapCUT2 utility used for fragment processing before phasing) — https://github.com/vibansal/HapCUT2

## Examples

```
SpecHap --vcf /your/gzvcf/file --frag /your/fragment/file --out /phased/vcf --pacbio
```

## Evaluation signals

- Confirm the phased VCF has been written to the path supplied to --out and the run completed without exceeding the usage boundary
- Compare phasing accuracy against the 1000 Genomes Project trio-phased haplotype used as 'ground truth' in benchmarking
- Verify the input VCF was bgzipped, tabix-indexed and sorted by position, and that the fragment file was sorted in the same order before the run
- Check resource usage against published envelopes (e.g. 6 CPU hours and 945MB peak memory for the ~32 Gbp Ambystoma mexicanum genome at ~32x PacBio SMRT coverage)

## Limitations

- There is no guarantee that SpecHap provides optimal solutions, and the interval size parameter might affect the phasing result
- A larger interval size might affect SpecHap's efficiency
- The current graph bisection model does not fit increased ploidy; no high-ploidy generalization exists
- SIH support is limited to SNPs and short insertions/deletions; haplotype-resolved structural variation is not supported
- No comprehensive analysis of technology-specific error influence on phasing quality has been conducted for Hi-C and 10X linked-reads
- No software version is stated for SpecHap

## Evidence

- [results] Fiedler-vector fallback behavior: [evidence span withheld — anchor status: not_found]
- [discussion] Min-cut interpretation: "The allele partitioning guided by Fiedler Vector might be interpreted as a min-cut heuristic on our linkage graph"
- [discussion] Divide-and-conquer intervals: "SpecHap adopts divide-and-conquer to accelerate the computation by dividing chromosomes into intervals of user-defined size."
- [discussion] Linear algebra transformation: "SpecHap transforms diploid SIH into a linear algebra problem by applying spectral graph theory."
- [abstract] Benchmark scope: "SpecHap consumed less memory and required less CPU time, yet achieved comparable accuracy with state-of-art methods across all the test instances"
- [readme] Tool availability: "SpecHap is an ultra fast phasing algorithm based on spectral graph analysis. SpecHap currently support general WGS sequencing, Hi-C, 10X linked-reads, PacBio SMRT and Oxford Nanopore"
