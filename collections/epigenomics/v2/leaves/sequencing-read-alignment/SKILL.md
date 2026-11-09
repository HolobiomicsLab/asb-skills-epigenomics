---
name: sequencing-read-alignment
description: Use when simulated (wgsim/LRSIM, PBSIM/DeepSimulator) or real sequencing
  reads must be mapped to a reference genome and produced as sorted, indexed BAM files
  as a prerequisite for fragment extraction (extractHAIRS) and haplotype assembly.
when_to_use_negative:
- Input is already a sorted, indexed BAM ready for extractHAIRS fragment extraction
  — skip alignment and proceed to fragment extraction
- The goal is direct phasing from pre-computed fragment files — this skill covers
  the upstream read-to-reference mapping step, not phasing itself
- Data are Hi-C or 10X barcode fragments already in extractHAIRS fragment format;
  alignment with these tools is not the relevant transformation
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: BWA-MEM
  role: Aligns simulated paired-end Illumina short reads to the human reference genome
- name: minimap2
  role: Aligns simulated PacBio SMRT and Oxford Nanopore long reads to the human reference
    genome
- name: htslib
  role: Required dependency used via samtools to sort and index the aligned BAM files
  repo: https://github.com/samtools/htslib
- name: samtools
  role: Sorts and indexes the resulting BAMs before fragment extraction with extractHAIRS
- name: SpecHap
  role: 'Downstream consumer: the sorted BAMs feed extractHAIRS fragment generation
    for SpecHap phasing benchmarking'
  repo: https://github.com/deepomicslab/SpecHap
- name: ExtractHair
  role: Modified HapCUT2 utility that consumes the sorted BAM and bgzipped, indexed
    VCF to produce fragment files; pinned htslib/samtools submodule commits are required
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:ef301b5e756da4e0e6492bb9aaf04b624d22bba9193678625ee2ea2e2e212bb4
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
  - ExtractHair
  - FastHare
  - HapCUT
  - HapCUT2
  - ReFHap
  - SpecHap
  - arpack-ng
  - htslib
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# sequencing-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns simulated or real sequencing reads to the human reference genome — BWA-MEM for paired-end Illumina short reads and minimap2 for PacBio SMRT and ONT long reads — then coordinate-sorts and indexes the BAMs with samtools/htslib so variant-supporting fragments can be extracted for haplotype phasing with SpecHap.

## When to use

Use when simulated (wgsim/LRSIM, PBSIM/DeepSimulator) or real sequencing reads must be mapped to a reference genome and produced as sorted, indexed BAM files as a prerequisite for fragment extraction (extractHAIRS) and haplotype assembly.

## When NOT to use

- Input is already a sorted, indexed BAM ready for extractHAIRS fragment extraction — skip alignment and proceed to fragment extraction
- The goal is direct phasing from pre-computed fragment files — this skill covers the upstream read-to-reference mapping step, not phasing itself
- Data are Hi-C or 10X barcode fragments already in extractHAIRS fragment format; alignment with these tools is not the relevant transformation

## Inputs

- Simulated paired-end Illumina reads (FASTQ, from wgsim/LRSIM at 10x/30x/50x coverage)
- Simulated 10X linked reads with barcodes (FASTQ, from LRSIM)
- Simulated PacBio SMRT long reads (FASTQ, from PBSIM)
- Simulated Oxford Nanopore long reads (FASTQ, from DeepSimulator)
- Human reference genome (FASTA)

## Outputs

- Coordinate-sorted BAM files of aligned reads
- BAM index files (e.g. .bai) from samtools/htslib, enabling position-ordered fragment extraction with extractHAIRS

## How to apply

Choose the aligner by read technology: BWA-MEM for paired-end Illumina short reads and minimap2 for error-prone third-generation long reads from PacBio SMRT or Oxford Nanopore, as done in the SpecHap benchmark on HG00403 chromosomes 1, 21 and 22. Align the reads against the human reference genome and pass the output through samtools/htslib to coordinate-sort and index each BAM, which is required downstream by the modified ExtractHair (from HapCUT2) when generating fragment files. Note that ExtractHair's htslib submodule is pinned to a specific commit (htslib at 26229a3, samtools at 255f97d) to ensure compatibility. Verify alignment quality before fragment extraction, since unaligned or malformed BAMs will propagate into the fragment files and distort phasing accuracy comparisons.

## Related tools

- **BWA-MEM** (Aligns simulated paired-end Illumina short reads to the human reference genome)
- **minimap2** (Aligns simulated PacBio SMRT and Oxford Nanopore long reads to the human reference genome)
- **htslib** (Required dependency used via samtools to sort and index the aligned BAM files) — https://github.com/samtools/htslib
- **samtools** (Sorts and indexes the resulting BAMs before fragment extraction with extractHAIRS)
- **SpecHap** (Downstream consumer: the sorted BAMs feed extractHAIRS fragment generation for SpecHap phasing benchmarking) — https://github.com/deepomicslab/SpecHap
- **ExtractHair** (Modified HapCUT2 utility that consumes the sorted BAM and bgzipped, indexed VCF to produce fragment files; pinned htslib/samtools submodule commits are required) — https://github.com/vibansal/HapCUT2

## Evaluation signals

- All simulated-read BAMs (Illumina PE, 10X, PacBio, ONT per coverage level 10x/30x/50x) are coordinate-sorted and indexed without errors from samtools
- extractHAIRS runs successfully on each sorted BAM together with the bgzipped, indexed VCF, producing non-empty fragment files in the same positional order as the VCF
- The ExtractHair build completes with the pinned submodules (htslib 26229a3, samtools 255f97d), confirming htslib compatibility for BAM reading
- Downstream phasing with SpecHap, HapCUT2, FastHare, DCHap and ReFHap yields accuracy values comparable to the 1000 Genomes trio-phased ground truth, indicating alignments support correct variant fragment extraction

## Limitations

- Alignment is only the upstream step of the benchmark; it does not itself validate phasing accuracy — that requires comparison against the 1000 Genomes trio-phased haplotype ground truth
- Fragment extraction requires strict sort order compatibility: 'sort -n -k6' for 10X/Hi-C or new-format fragments and 'sort -n -k3' for paired-end NGS, PacBio and ONT default-format fragments, in the same positional order as the VCF
- The modified ExtractHair depends on specific pinned submodule commits, so arbitrary htslib/samtools versions may not work

## Evidence

- [supplementary] BWA-MEM vs minimap2 choice: [evidence span withheld — anchor status: not_found]
