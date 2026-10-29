---
name: half-linker-chia-pet-read-processing
description: Use when raw paired-end FASTQ reads from a half-linker (short-read) ChIA-PET
  experiment must be converted into significant chromatin loops, requiring linker
  trimming, bwa alignment, PET filtering, duplicate removal, peak calling and MICC
  significance testing.
when_to_use_negative:
- The data are from the new bridge-linker ChIA-PET protocol with long reads, which
  legacy tools such as Mango were not designed to analyze
- Significant loops have already been called and the task is downstream interpretation
  (e.g. transitivity or redundancy analysis of loops)
- The data are a different 3C-family assay (e.g. Hi-C) whose read structure lacks
  ChIA-PET linkers and paired-end tags
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0160
tools:
- name: ChIA-PET2
  role: Pipeline that integrates linker trimming, read mapping, duplicate removal,
    peak calling and chromatin loop calling in A/B-linker (half-linker) mode
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bwa
  role: Aligns linker-trimmed paired-end reads to the reference genome
  repo: https://github.com/lh3/bwa
- name: samtools
  role: Required dependency for SAM/BAM handling within the pipeline
  repo: https://github.com/samtools/samtools
- name: bedtools
  role: bed/bedpe file manipulation during PET construction
  repo: https://github.com/arq5x/bedtools2
- name: MACS2
  role: ChIP-Seq-like peak calling on tagAlign input to define loop anchors
  repo: https://github.com/macs3-project/MACS
- name: Mango
  role: Alternative half-linker pipeline used as the baseline whose loop calls are
    compared for reproducibility
- name: R
  role: Runs QCplots.R and MICC2.R for quality control plots and MICC significance
    estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:69aeec2f7ed077c8252102466df545da11db5e9ed2ec5d07a57650cb6d7b778a
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
  - MACS2
  - Mango
  - R
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# half-linker-chia-pet-read-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end processing of half-linker (short-read) ChIA-PET data from raw paired-end FASTQ to significant chromatin loop calls, using the ChIA-PET2 pipeline with its A/B-linker mode (-m 0). It yields substantially higher inter-replicate reproducibility than Mango (Jaccard 52.5% vs 25.6% on K562 POL2 replicates).

## When to use

Use when raw paired-end FASTQ reads from a half-linker (short-read) ChIA-PET experiment must be converted into significant chromatin loops, requiring linker trimming, bwa alignment, PET filtering, duplicate removal, peak calling and MICC significance testing.

## When NOT to use

- The data are from the new bridge-linker ChIA-PET protocol with long reads, which legacy tools such as Mango were not designed to analyze
- Significant loops have already been called and the task is downstream interpretation (e.g. transitivity or redundancy analysis of loops)
- The data are a different 3C-family assay (e.g. Hi-C) whose read structure lacks ChIA-PET linkers and paired-end tags

## Inputs

- Raw paired-end FASTQ files (.fastq.gz) from half-linker ChIA-PET replicates (e.g. K562 POL2 half-linker, GEO GSE72816)
- bwa genome index built from the reference genome FASTA
- Tab-delimited chromosome-size file for bedtools

## Outputs

- Linker-trimmed paired-end FASTQ files
- SAM/BAM alignments and a bedpe file of filtered paired-end tags (MAPQ >= 30)
- tagAlign file for MACS2 input and MACS2 peak calls at q-value 0.05
- interactions.intra.bedpe, interactions.inter.bedpe, and interactions.MICC files of significant chromatin loops

## How to apply

Run ChIA-PET2 in half-linker A/B-linker mode (-m 0) per replicate: trimLinker removes linker sequences from paired-end fastq(.gz) files; bwa aligns the trimmed reads; buildBedpe constructs a bedpe file from the two SAM files with PET filtering at MAPQ cutoff 30; removeDup removes duplicate PETs; buildTagAlign generates a tagAlign for MACS2 peak calling with default '-q 0.05'; and bedpe2Interaction/MICC2.R call statistically significant chromatin loops with the MICC default CUTOFFPET=2. The rationale is that anchor positions of most chromatin loops coincide with ChIP-Seq-like MACS2 peaks, and the MICC significance model on PET counts detects reproducible loops that Mango's conservative calling misses. Prepare a bwa genome index and a tab-delimited chromosome-size file for bedtools before running. Compare inter-replicate Jaccard indices of called loop sets to verify sensitivity and reproducibility at the same FDR.

## Related tools

- **ChIA-PET2** (Pipeline that integrates linker trimming, read mapping, duplicate removal, peak calling and chromatin loop calling in A/B-linker (half-linker) mode) — https://github.com/GuipengLi/ChIA-PET2
- **bwa** (Aligns linker-trimmed paired-end reads to the reference genome) — https://github.com/lh3/bwa
- **samtools** (Required dependency for SAM/BAM handling within the pipeline) — https://github.com/samtools/samtools
- **bedtools** (bed/bedpe file manipulation during PET construction) — https://github.com/arq5x/bedtools2
- **MACS2** (ChIP-Seq-like peak calling on tagAlign input to define loop anchors) — https://github.com/macs3-project/MACS
- **Mango** (Alternative half-linker pipeline used as the baseline whose loop calls are compared for reproducibility)
- **R** (Runs QCplots.R and MICC2.R for quality control plots and MICC significance estimation)

## Examples

```
bwa mem ref.fa read1.fq read2.fq | gzip -3 > aln-pe.sam.gz
```

## Evaluation signals

- On the K562 POL2 half-linker replicates (GSE72816), ChIA-PET2 should reproduce >9,000 significant loops per replicate versus <300 for Mango
- Inter-replicate Jaccard index of called loop sets should be near 52.5% for ChIA-PET2 (Mango target 25.6%)
- Recovery of Mango-called loops by ChIA-PET2 should be ~64% for replicate 1 and ~74% for replicate 2
- The bedpe file should contain PETs passing the MAPQ cutoff 30, and PCR duplicates should be removed by removeDup before peak calling
- MACS2 peaks should be called with '-q 0.05' and MICC significance estimated with the default CUTOFFPET=2

## Limitations

- Loop calling depends on ChIP-Seq-like peak calling sensitivity, since anchors are assumed to lie at MACS2-called peaks
- No software versions are stated for ChIA-PET2 or its required dependencies (bwa, samtools, bedtools, MACS2, g++), so exact reproduction may vary
- Mango, the alternative for half-linker data, is too conservative at significant loop calling with low reproducibility between replicates

## Evidence
