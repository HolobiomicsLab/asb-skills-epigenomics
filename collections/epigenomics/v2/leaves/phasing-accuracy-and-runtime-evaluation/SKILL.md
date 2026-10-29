---
name: phasing-accuracy-and-runtime-evaluation
description: Use when comparing diploid single-individual haplotyping tools on simulated
  or real reads (Illumina, 10X, Hi-C, PacBio SMRT, ONT) and you need runtime, memory,
  and phasing-accuracy tables.
when_to_use_negative:
- Organism or data are not diploid — HapCUT2 and SpecHap's graph bisection model assume
  two haplotypes; higher ploidy is not supported
- Goal is haplotype-resolved structural variation — current SIH tools including SpecHap
  support only SNPs and short insertions/deletions
- Input is already a phased VCF from a population/trio phaser rather than fragment
  files — this skill evaluates fragment-based haplotype assembly
edam_operation: http://edamontology.org/operation_0466
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Phasing method under test; also the benchmarked tool being evaluated for speed
    and memory
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Baseline max-cut haplotype assembly method used for runtime/accuracy comparison
  repo: https://github.com/vibansal/HapCUT2
- name: FastHare
  role: Baseline fragment-partitioning phasing method used for comparison
- name: ReFHap
  role: Baseline graph-cut heuristic phasing method used for comparison
- name: DCHap
  role: Baseline divide-and-conquer MEC phasing method for third-generation data used
    for comparison
- name: HapCUT
  role: Baseline MEC-max-cut heuristic phasing method used for comparison
- name: ExtractHair
  role: Fragment extraction from sorted BAM and VCF for all methods (HapCUT2-derived,
    modified utility)
  repo: https://github.com/deepomicslab/SpecHap
- name: htslib
  role: Required dependency for BAM/VCF handling and bgzip/tabix indexing
  repo: https://github.com/samtools/htslib
- name: arpack-ng
  role: Eigen-decomposition dependency for SpecHap build
  repo: https://github.com/opencollab/arpack-ng
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:43c5204cfcaa76f8f9c3b9c7d7b7995a4be443ceebea37fbbf913d78277010ea
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
  edam_operation: http://edamontology.org/operation_0466
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# phasing-accuracy-and-runtime-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking protocol for haplotype-phasing tools (SpecHap vs HapCUT2, ReFHap, FastHare, DCHap) that measures CPU time and peak memory per run and scores phased haplotypes against 1000 Genomes trio-phased ground truth on simulated HG00403 data.

## When to use

Use when comparing diploid single-individual haplotyping tools on simulated or real reads (Illumina, 10X, Hi-C, PacBio SMRT, ONT) and you need runtime, memory, and phasing-accuracy tables.

## When NOT to use

- Organism or data are not diploid — HapCUT2 and SpecHap's graph bisection model assume two haplotypes; higher ploidy is not supported
- Goal is haplotype-resolved structural variation — current SIH tools including SpecHap support only SNPs and short insertions/deletions
- Input is already a phased VCF from a population/trio phaser rather than fragment files — this skill evaluates fragment-based haplotype assembly

## Inputs

- Sorted, indexed BAM files of simulated reads (BWA-MEM for short reads, minimap2 for long reads)
- bgzipped, indexed VCF of heterozygous variants (1000 Genomes trio-phased HG00403)
- Variant-supporting fragment files (from extractHAIRS) sorted with 'sort -n -k6' (10X/Hi-C) or 'sort -n -k3' (NGS/PacBio/ONT)
- 10X barcode-spanning bgzip/tabix-indexed BED file from BarcodeExtract
- Simulated datasets: PacBio SMRT and ONT reads (PBSIM/DeepSimulator), Illumina and 10X linked reads (wgsim/LRSIM) at 10x, 30x, 50x coverage

## Outputs

- benchmark_comparison.csv with phasing accuracy and CPU runtime per tool, per data type, per coverage
- Per-method phased haplotype output files
- CPU time and peak memory usage measurements for each phasing run

## How to apply

Simulate reads from trio-phased HG00403 haplotypes with wgsim/LRSIM (10x, 30x, 50x) and PBSIM/DeepSimulator for long reads, align with BWA-MEM (short) or minimap2 (long), and sort/index BAMs with samtools/htslib. Extract variant-supporting fragments with extractHAIRS from the sorted BAM and bgzipped indexed VCF, using --hic 1 for Hi-C, --10x 1 with a BarcodeExtract-generated bgzip/tabix barcode BED for 10X, and --pacbio 1 or --ont 1 with --ref for long reads. Sort fragment files with 'sort -n -k6' for 10X/Hi-C or new-format fragments and 'sort -n -k3' for paired-end NGS, PacBio and ONT default-format fragments, keeping positional order consistent with the VCF. Run SpecHap and each baseline (FastHare, HapCUT, DCHap, ReFHap, HapCUT2) recording CPU time and peak memory, then compare phased haplotypes to the 1000 Genomes trio ground truth to compute accuracy per tool and coverage.

## Related tools

- **SpecHap** (Phasing method under test; also the benchmarked tool being evaluated for speed and memory) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Baseline max-cut haplotype assembly method used for runtime/accuracy comparison) — https://github.com/vibansal/HapCUT2
- **FastHare** (Baseline fragment-partitioning phasing method used for comparison)
- **ReFHap** (Baseline graph-cut heuristic phasing method used for comparison)
- **DCHap** (Baseline divide-and-conquer MEC phasing method for third-generation data used for comparison)
- **HapCUT** (Baseline MEC-max-cut heuristic phasing method used for comparison)
- **ExtractHair** (Fragment extraction from sorted BAM and VCF for all methods (HapCUT2-derived, modified utility)) — https://github.com/deepomicslab/SpecHap
- **htslib** (Required dependency for BAM/VCF handling and bgzip/tabix indexing) — https://github.com/samtools/htslib
- **arpack-ng** (Eigen-decomposition dependency for SpecHap build) — https://github.com/opencollab/arpack-ng

## Examples

```
extractHAIRS --pacbio 1 --ref hg19.fa --vcf hg00403.vcf.gz --bam simulated.bam > frags.txt && sort -n -k3 frags.txt | SpecHap -f - -v hg00403.vcf.gz
```

## Evaluation signals

- On simulated PacBio SMRT data SpecHap should outperform all four baselines in CPU time and peak memory: ~40x faster than HapCUT2, ~100x faster than ReFHap, and as fast as FastHare
- Phasing accuracy computed against 1000 Genomes trio-phased ground truth should be comparable to state-of-the-art methods across all test instances
- Fragment file sort order must match VCF positional order (sort -n -k6 for 10X/Hi-C, sort -n -k3 for paired-end/PacBio/ONT formats) or accuracy comparison is invalid
- SpecHap graph-cut CPU time should show no statistical difference across 10x, 30x and 50x coverages
- All runs should complete within the usage boundary (other packages failed at 24 CPU hours for Homo sapiens or 5 CPU days / 40GB for Ambystoma mexicanum)

## Limitations

- No guarantee of optimal solutions; phasing results depend on the user-defined interval size, and a larger interval size may reduce SpecHap's efficiency
- Comprehensive analysis of technology-specific error influence on Hi-C and 10X linked-read phasing quality not yet conducted
- The current graph bisection model does not fit increased ploidy; only SNPs and short insertions/deletions are supported, not structural variation
- No software version stated for SpecHap; no changelog found

## Evidence

- [results] Simulated PacBio SMRT benchmark shows SpecHap faster and leaner than baselines: [evidence span withheld — anchor status: not_found]
