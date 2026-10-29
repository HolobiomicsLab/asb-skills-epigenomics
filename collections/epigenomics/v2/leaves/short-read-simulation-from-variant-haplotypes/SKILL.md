---
name: short-read-simulation-from-variant-haplotypes
description: Use when benchmarking haplotype assembly or variant phasing tools requires
  synthetic reads whose true haplotype origin is known, e.g. from trio-phased 1000
  Genomes haplotypes across coverages (10x/30x/50x) and platforms.
when_to_use_negative:
- When the goal is to phase real WGS samples (e.g. NA12878/NA19240 1000 Genomes data)
  rather than validate phasing accuracy against simulated ground truth
- When haplotypes are not known or trio-phased — simulated-read accuracy evaluation
  requires ground-truth haplotype origin
- When benchmarking tools on data types they do not support (the article notes some
  packages 'either did not support a specific data type or failed to complete the
  task within the usage boundary')
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: wgsim
  role: Simulates paired-end Illumina short reads from the two haplotypes of HG00403
    at matched coverage
- name: LRSIM
  role: Simulates 10X linked reads with barcodes from the phased haplotypes
- name: PBSIM
  role: Simulates PacBio SMRT long reads at matched coverage profiles
- name: DeepSimulator
  role: Simulates Oxford Nanopore long reads at matched coverage profiles
- name: BWA-MEM
  role: Aligns simulated short reads to the human reference genome
- name: minimap2
  role: Aligns simulated PacBio/ONT long reads to the human reference genome
- name: htslib
  role: samtools/htslib sorts and indexes the alignment BAMs prior to fragment extraction
    with extractHAIRS
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:ae92e08be934c35b7568f9e8b661ff596e3533d0b722af2ac7d9ddded0d11f41
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/samtools/htslib
  tools:
  - BWA-MEM
  - DCHap
  - DeepSimulator
  - ExtractHair
  - FastHare
  - HapCUT
  - HapCUT2
  - LRSIM
  - PBSIM
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  - minimap2
  - wgsim
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# short-read-simulation-from-variant-haplotypes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate in silico sequencing reads from known diploid haplotypes (e.g. trio-phased HG00403 chromosomes 1, 21, 22) using wgsim/LRSIM for paired-end Illumina and 10X linked reads and PBSIM/DeepSimulator for PacBio SMRT and ONT long reads at 10x, 30x and 50x coverage, to create ground-truth benchmarks for haplotype assembly tools like SpecHap.

## When to use

Use when benchmarking haplotype assembly or variant phasing tools requires synthetic reads whose true haplotype origin is known, e.g. from trio-phased 1000 Genomes haplotypes across coverages (10x/30x/50x) and platforms.

## When NOT to use

- When the goal is to phase real WGS samples (e.g. NA12878/NA19240 1000 Genomes data) rather than validate phasing accuracy against simulated ground truth
- When haplotypes are not known or trio-phased — simulated-read accuracy evaluation requires ground-truth haplotype origin
- When benchmarking tools on data types they do not support (the article notes some packages 'either did not support a specific data type or failed to complete the task within the usage boundary')

## Inputs

- Two trio-phased haplotypes of HG00403 (1000 Genomes Project ground truth) for chromosomes 1, 21 and 22
- Variant call set (bgzipped, tabix-indexed VCF) defining heterozygous SNVs on the phased haplotypes
- Human reference genome for read simulation and alignment
- Target coverage profiles (10x, 30x, 50x) per sequencing platform

## Outputs

- Simulated paired-end Illumina FASTQ reads at 10x/30x/50x coverage
- Simulated 10X linked reads with barcodes
- Simulated PacBio SMRT and Oxford Nanopore long reads
- Sorted and indexed BAM files from BWA-MEM (short reads) and minimap2 (long reads) alignments

## How to apply

Start from the two haplotypes of a trio-phased individual (HG00403 chromosomes 1, 21 and 22 in the SpecHap benchmark) and simulate paired-end Illumina and 10X linked reads with wgsim/LRSIM at 10x, 30x and 50x coverage. Simulate third-generation long reads with PBSIM for PacBio SMRT and DeepSimulator for Oxford Nanopore using matched coverage profiles. Align simulated short reads with BWA-MEM and long reads with minimap2, then sort/index the resulting BAMs with samtools/htslib. The known haplotype origin of every simulated read provides the ground truth used later to compute phasing accuracy against the 1000 Genomes trio-phased haplotype.

## Related tools

- **wgsim** (Simulates paired-end Illumina short reads from the two haplotypes of HG00403 at matched coverage)
- **LRSIM** (Simulates 10X linked reads with barcodes from the phased haplotypes)
- **PBSIM** (Simulates PacBio SMRT long reads at matched coverage profiles)
- **DeepSimulator** (Simulates Oxford Nanopore long reads at matched coverage profiles)
- **BWA-MEM** (Aligns simulated short reads to the human reference genome)
- **minimap2** (Aligns simulated PacBio/ONT long reads to the human reference genome)
- **htslib** (samtools/htslib sorts and indexes the alignment BAMs prior to fragment extraction with extractHAIRS) — https://github.com/samtools/htslib

## Examples

```
wgsim -d 350 -s 50 -N 30000000 -1 150 -2 150 hap1.fa hap2.fa read1.fq read2.fq && bwa mem -t 16 ref.fa read1.fq read2.fq | samtools sort -o aln.bam
```

## Evaluation signals

- Simulated read coverage matches the target profiles of 10x, 30x and 50x per platform
- BAM files are coordinate-sorted and indexed (samtools/htslib) before fragment extraction
- Alignment choice matches read type: BWA-MEM for short reads, minimap2 for long reads
- Downstream phasing accuracy is computed against the 1000 Genomes trio-phased ground truth, confirming the simulation preserved haplotype structure
- Simulated benchmark reproduces the published ordering: SpecHap ~40x faster than HapCUT2 and ~100x faster than ReFHap on PacBio SMRT data

## Limitations

- In silico simulation may not capture all platform-specific error profiles; the article notes 'a comprehensive analysis on the influence of technological specific error over phasing quality may be conducted for Hi-C and 10× linked-reads'
- 10X simulation must apply the filter step keeping only fragments with white-listed barcodes to be realistic
- No software version is stated for the simulation tools used, limiting reproducibility

## Evidence

- [supplementary] Simulate paired-end Illumina and 10X linked reads from the two haplotypes of the trio-phased HG00403 individual using wgsim/LRSIM at 10x, 30x and 50x coverage.: [evidence span withheld — anchor status: not_found]
- [supplementary] Align the simulated reads to the human reference genome (BWA-MEM for short reads, minimap2 for long reads) and sort/index the resulting BAMs with samtools/htslib.: [evidence span withheld — anchor status: not_found]
