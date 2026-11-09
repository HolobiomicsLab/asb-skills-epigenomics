---
name: long-read-haplotype-assembly
description: Use when you have a bgzipped, tabix-indexed VCF plus a sorted fragment
  file (10X/Hi-C, PacBio SMRT, ONT, or paired-end NGS) and need fast, memory-efficient
  diploid phasing, especially for error-prone third-generation long reads.
when_to_use_negative:
- 'The organism is polyploid: the current graph bisection model assumes two haplotypes
  and does not fit when ploidy increases.'
- 'You need haplotype-resolved structural variation: SpecHap supports only SNPs, short
  insertions and deletions; a linkage graph incorporating genome rearrangements is
  not yet implemented.'
- 'You require a guaranteed optimal phasing solution: SpecHap provides no optimality
  guarantee and results depend on the interval size parameter.'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: 'Primary phasing tool: builds the linkage graph, eigendecomposes it via ARPACK,
    and partitions alleles by Fiedler-vector sign to assemble diploid haplotypes'
  repo: https://github.com/deepomicslab/SpecHap
- name: ExtractHair
  role: Modified utility originally from HapCUT2 used to extract variant-supporting
    fragments from aligned BAMs and a bgzipped, indexed VCF
  repo: https://github.com/vibansal/HapCUT2
- name: arpack-ng
  role: Dependency for the eigendecomposition (Fiedler-vector computation) of the
    linkage graph; recommended for stable utilization
  repo: https://github.com/opencollab/arpack-ng
- name: htslib
  role: Required library underlying fragment/VCF I/O and the bgzip/tabix preprocessing
    of input files
  repo: https://github.com/samtools/htslib
- name: HapCUT2
  role: Baseline comparator (max-cut maximum-likelihood phasing) for benchmarking
    SpecHap's accuracy, CPU time and peak memory
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:cfc07739ae6ddcc9444a3df3ee539d64e01c24aa407422cc0c560b3dc49ecdd0
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
  - ExtractHair
  - FastHare
  - HapCUT
  - HapCUT2
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# long-read-haplotype-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble diploid haplotypes from sequencing fragments using SpecHap, which transforms single individual haplotyping (SIH) into a linear algebra problem via spectral graph theory, partitioning variants of the linkage graph by Fiedler-vector sign rather than iteratively optimizing MEC or haplotype likelihood.

## When to use

Use when you have a bgzipped, tabix-indexed VCF plus a sorted fragment file (10X/Hi-C, PacBio SMRT, ONT, or paired-end NGS) and need fast, memory-efficient diploid phasing, especially for error-prone third-generation long reads.

## When NOT to use

- The organism is polyploid: the current graph bisection model assumes two haplotypes and does not fit when ploidy increases.
- You need haplotype-resolved structural variation: SpecHap supports only SNPs, short insertions and deletions; a linkage graph incorporating genome rearrangements is not yet implemented.
- You require a guaranteed optimal phasing solution: SpecHap provides no optimality guarantee and results depend on the interval size parameter.

## Inputs

- bgzipped, tabix-indexed, position-sorted VCF of heterozygous variant calls for a single diploid individual
- fragment file of variant-supporting reads sorted in the same positional order as the VCF (10X/Hi-C/new-format: sort -n -k6; paired-end NGS/PacBio/ONT default: sort -n -k3)
- optional sorted/indexed BAM of reads aligned to the reference (BWA-MEM for short reads, minimap2 for long reads) for fragment extraction with ExtractHair

## Outputs

- phased VCF written to the path supplied to --out, containing haplotype-resolved variant assignments for the diploid individual

## How to apply

First verify the input VCF is bgzipped, tabix-indexed, and sorted by position, and sort the fragment file in the same positional order ('sort -n -k6 in.frag > sorted.frag' for 10X/Hi-C or new-format fragments; 'sort -n -k3 in.frag > sorted.frag' for paired-end NGS, PacBio SMRT or ONT default format). Fragments can be extracted from aligned BAMs with the modified ExtractHair utility (from HapCUT2), using --hic 1 for Hi-C, --10x 1 with a barcode-spanning BED for 10X, or --pacbio 1/--ont 1 with --ref for long reads. Invoke SpecHap with the platform flag (e.g. --pacbio) so the linkage graph is eigendecomposed via the ARPACK dependency and alleles are partitioned by Fiedler-vector sign; when the Fiedler vector yields two sub-blocks instead of a direct resolution, SpecHap recursively bisects until the haplotype is deducible, pruning two-variant sub-blocks it cannot resolve. SpecHap divides chromosomes into user-defined intervals (divide-and-conquer), so choose interval sizes carefully: a larger interval size might affect efficiency and the interval size may affect the phasing result. Build SpecHap with cmake/make, and install the prerequisites arpack-ng (recommended for stable eigen-calculation) and htslib.

## Related tools

- **SpecHap** (Primary phasing tool: builds the linkage graph, eigendecomposes it via ARPACK, and partitions alleles by Fiedler-vector sign to assemble diploid haplotypes) — https://github.com/deepomicslab/SpecHap
- **ExtractHair** (Modified utility originally from HapCUT2 used to extract variant-supporting fragments from aligned BAMs and a bgzipped, indexed VCF) — https://github.com/vibansal/HapCUT2
- **arpack-ng** (Dependency for the eigendecomposition (Fiedler-vector computation) of the linkage graph; recommended for stable utilization) — https://github.com/opencollab/arpack-ng
- **htslib** (Required library underlying fragment/VCF I/O and the bgzip/tabix preprocessing of input files) — https://github.com/samtools/htslib
- **HapCUT2** (Baseline comparator (max-cut maximum-likelihood phasing) for benchmarking SpecHap's accuracy, CPU time and peak memory) — https://github.com/vibansal/HapCUT2

## Examples

```
SpecHap --vcf /your/gzvcf/file --frag /your/fragment/file --out /phased/vcf --pacbio
```

## Evaluation signals

- The phased VCF exists at the path given to --out and the run completed without exceeding usage boundaries (e.g. 24 CPU hours for Homo sapiens, 5 CPU days / 40 GB for Ambystoma mexicanum).
- Compare phased haplotypes against 1000 Genomes trio-phased ground truth to compute phasing accuracy; expect accuracy comparable to HapCUT2, ReFHap, FastHare and DCHap.
- Record CPU time and peak memory per run: on simulated PacBio SMRT data SpecHap should be ~40x faster than HapCUT2, ~100x faster than ReFHap, as fast as FastHare, and consume less memory.
- Verify input preprocessing invariants: VCF is bgzipped and tabix-indexed, and fragment file is numerically sorted ('sort -n -k6' or 'sort -n -k3') in the same positional order as the VCF.
- On huge genomes (e.g. Ambystoma mexicanum ~32 Gbp at ~32x PacBio SMRT), completion within ~6 CPU hours and ~945 MB peak memory indicates correct, efficient application.

## Limitations

- No software version is stated for SpecHap and no changelog was found.
- No guarantee of optimal solutions; phasing results depend on the interval size parameter, and a larger interval size might reduce efficiency.
- A comprehensive analysis of technology-specific error influence on phasing quality for Hi-C and 10X linked-reads has not yet been conducted.
- The current graph bisection model does not fit increased ploidy; high-ploidy generalization is not implemented.
- Haplotype-resolved structural variation determination is not supported; only SNPs, short insertions and deletions are handled.
- 10X linked-read pipelines filter to keep only fragments with white-listed barcodes, so barcodes outside the whitelist are dropped.

## Evidence

- [results] When the Fiedler vector categorizes variants into two sub-blocks rather than directly resolving the haplotype, SpecHap partitions the variants recursively.: [evidence span withheld — anchor status: not_found]
