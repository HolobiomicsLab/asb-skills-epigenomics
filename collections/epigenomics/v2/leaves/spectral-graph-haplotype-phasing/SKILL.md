---
name: spectral-graph-haplotype-phasing
description: Use when assembling haplotypes for a single diploid individual from long
  reads (PacBio SMRT, ONT), Hi-C, or 10X linked-reads, especially very large genomes
  or high coverage where max-cut/MEC tools exceed memory or time limits.
when_to_use_negative:
- 'Organism is polyploid or ploidy > 2: the current graph bisection model does not
  fit increased ploidy and high-ploidy generalization is not implemented'
- 'Goal is haplotype-resolved structural variation / genome rearrangements: SpecHap
  supports only SNPs and short insertions and deletions'
- Input is already a fragment file from an incompatible protocol — note 10X linked-reads
  require filtering to keep only white-listed barcode fragments before phasing
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: 'Primary phasing tool: builds the linkage graph and partitions alleles via
    Fiedler-vector spectral bisection; supports WGS, Hi-C, 10X linked-reads, PacBio
    SMRT and ONT'
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2 extractHAIRS
  role: Modified ExtractHair utility (vendored as a submodule) used for fragment processing/extraction
    from alignments prior to phasing
  repo: https://github.com/vibansal/HapCUT2
- name: arpack-ng
  role: Dependency providing ARPACK eigensolver routines for SpecHap's eigen-decomposition
    (requires BLAS/LAPACK)
  repo: https://github.com/opencollab/arpack-ng
- name: htslib
  role: Required library dependency (also vendored as a pinned submodule for extractHAIRS
    build)
  repo: https://github.com/samtools/htslib
- name: HapCUT2
  role: Comparison phasing method (max-cut heuristic) benchmarked against SpecHap;
    its failure on gigantic genomes motivates this skill
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:c2874c0c2c5eceae7985775fbcff0e80ad9e813125ed7f27477ff6ff793647be
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
  - HapCUT2
  - HapCUT2 extractHAIRS
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# spectral-graph-haplotype-phasing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Phases diploid single-individual haplotypes by transforming the SIH problem into a linear-algebra problem: build a linkage graph from read fragments, then partition alleles using spectral graph theory (Fiedler-vector bisection) with divide-and-conquer over chromosome intervals. Enables phasing of gigantic genomes (e.g. 32 Gbp axolotl) where MEC/max-cut tools fail.

## When to use

Use when assembling haplotypes for a single diploid individual from long reads (PacBio SMRT, ONT), Hi-C, or 10X linked-reads, especially very large genomes or high coverage where max-cut/MEC tools exceed memory or time limits.

## When NOT to use

- Organism is polyploid or ploidy > 2: the current graph bisection model does not fit increased ploidy and high-ploidy generalization is not implemented
- Goal is haplotype-resolved structural variation / genome rearrangements: SpecHap supports only SNPs and short insertions and deletions
- Input is already a fragment file from an incompatible protocol — note 10X linked-reads require filtering to keep only white-listed barcode fragments before phasing

## Inputs

- Position-sorted, bgzipped VCF of variant calls (SNPs, short insertions/deletions) for one diploid individual
- Read-fragment file extracted from alignments (e.g. via extractHAIRS with '--pacbio 1' from a PacBio SMRT BAM and reference)
- Coordinate-sorted fragment file ('sort -n -k3' for default PacBio fragment format)
- BAM alignment file and reference genome for fragment extraction

## Outputs

- Phased VCF written to the --out path
- Resource-usage log (CPU time and peak resident memory, e.g. /usr/bin/time -v)

## How to apply

Extract read-fragment data from alignments using the HapCUT2-derived extractHAIRS utility (e.g. '--pacbio 1' plus BAM, VCF, and reference arguments), then sort the fragment file in coordinate order with 'sort -n -k3' to match a position-sorted, bgzipped VCF. Run SpecHap with '--vcf <gzipped VCF> --frag <sorted fragment file> --out <phased VCF> --pacbio' (platform flag varies by data type; SpecHap supports general WGS, Hi-C, 10X linked-reads, PacBio SMRT and ONT). Internally, SpecHap builds a linkage graph, divides chromosomes into user-defined intervals (divide-and-conquer), and partitions alleles via the Fiedler vector — a min-cut heuristic — rather than iteratively optimizing MEC or haplotype likelihood. Wrap the run in a resource monitor such as '/usr/bin/time -v' to capture CPU time and peak resident memory, and build SpecHap with cmake/make after installing arpack-ng (needs BLAS/LAPACK) and htslib, plus the pinned htslib/samtools submodules for extractHAIRS. Accuracy can be benchmarked against trio-phased 1000 Genomes Project haplotypes as ground truth.

## Related tools

- **SpecHap** (Primary phasing tool: builds the linkage graph and partitions alleles via Fiedler-vector spectral bisection; supports WGS, Hi-C, 10X linked-reads, PacBio SMRT and ONT) — https://github.com/deepomicslab/SpecHap
- **HapCUT2 extractHAIRS** (Modified ExtractHair utility (vendored as a submodule) used for fragment processing/extraction from alignments prior to phasing) — https://github.com/vibansal/HapCUT2
- **arpack-ng** (Dependency providing ARPACK eigensolver routines for SpecHap's eigen-decomposition (requires BLAS/LAPACK)) — https://github.com/opencollab/arpack-ng
- **htslib** (Required library dependency (also vendored as a pinned submodule for extractHAIRS build)) — https://github.com/samtools/htslib
- **HapCUT2** (Comparison phasing method (max-cut heuristic) benchmarked against SpecHap; its failure on gigantic genomes motivates this skill) — https://github.com/vibansal/HapCUT2

## Examples

```
SpecHap --vcf variants.vcf.gz --frag fragments.sorted.txt --out phased.vcf --pacbio
```

## Evaluation signals

- A complete phased VCF is written to the --out path without error
- Recorded CPU time and peak resident memory fall within the reported envelope (e.g. axolotl benchmark: ~6 CPU hours, 945 MB peak memory, vs 5 CPU-day / 40 GB limits)
- Switch rate / accuracy against a trio-phased 1000 Genomes Project haplotype ground truth is comparable to state-of-the-art methods (HapCUT2, DCHap, ReFHap, FastHare)
- Fragment file is coordinate-sorted ('sort -n -k3') and matches the position-sorted bgzipped VCF before running SpecHap
- SpecHap graph-cut routine CPU time shows no statistical difference across 10x, 30x and 50x coverages in benchmark settings

## Limitations

- No guarantee of optimal solutions; the user-defined interval size can affect phasing results, and a larger interval size may reduce efficiency
- Graph bisection model does not generalize to increased ploidy (high-ploidy phasing not implemented)
- Only SNPs, short insertions and deletions are supported; haplotype-resolved structural variation is not supported
- Comprehensive analysis of technology-specific error influence on Hi-C and 10X linked-reads phasing quality has not been conducted
- No software version stated for SpecHap; no changelog available

## Evidence
