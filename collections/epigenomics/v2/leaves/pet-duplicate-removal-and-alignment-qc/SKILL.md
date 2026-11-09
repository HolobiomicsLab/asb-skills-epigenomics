---
name: pet-duplicate-removal-and-alignment-qc
description: Use when processing raw ChIA-PET FASTQ (bridge-linker or half-linker)
  data and you need PCR duplicate removal and per-step QC before PET construction,
  MACS2 peak calling, and MICC loop calling.
when_to_use_negative:
- Input data are not ChIA-PET paired-end reads (e.g. plain ChIP-Seq or Hi-C libraries)
  — linker trimming, PET construction and PET duplicate removal do not apply.
- You only need statistically significant loop calls from an existing bedpe file and
  already have deduplicated PETs — use bedpe2Interaction/MICC2.R directly instead.
- Data come from a long-read ChIA-PET protocol requiring long-read alignment mode
  rather than short-read bwa alignment handled here.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: ChIA-PET2
  role: Pipeline that integrates trimLinker, buildBedpe, removeDup, buildTagAlign
    and QCplots.R steps for duplicate removal and QC
  repo: https://github.com/GuipengLi/ChIA-PET2
- name: bwa
  role: Short-read aligner used after linker trimming to map PET reads before duplicate
    removal
  repo: https://github.com/lh3/bwa
- name: samtools
  role: SAM/BAM handling for alignment files feeding PET construction
  repo: https://github.com/samtools/samtools
- name: bedtools
  role: bed/bedpe file manipulation during PET filtering and deduplication
  repo: https://github.com/arq5x/bedtools2
- name: MACS2
  role: Consumes the buildTagAlign output (default q 0.05) for ChIP-Seq-like anchor
    peak calling after dedup
  repo: https://github.com/macs3-project/MACS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkw809
    title: 'ChIA-PET2: a versatile and flexible pipeline for ChIA-PET data analysis'
  source_content_hash: sha256:cbbcf19a83def45713ed1e988cdb02e20e75db98e30d623638df550ed4ce12a9
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pet-duplicate-removal-and-alignment-qc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Remove PCR-duplicate paired-end tags (PETs) and run quality-control plots after linker trimming and bwa alignment in the ChIA-PET2 pipeline, so that downstream peak calling and chromatin loop calling operate on deduplicated, quality-filtered PETs.

## When to use

Use when processing raw ChIA-PET FASTQ (bridge-linker or half-linker) data and you need PCR duplicate removal and per-step QC before PET construction, MACS2 peak calling, and MICC loop calling.

## When NOT to use

- Input data are not ChIA-PET paired-end reads (e.g. plain ChIP-Seq or Hi-C libraries) — linker trimming, PET construction and PET duplicate removal do not apply.
- You only need statistically significant loop calls from an existing bedpe file and already have deduplicated PETs — use bedpe2Interaction/MICC2.R directly instead.
- Data come from a long-read ChIA-PET protocol requiring long-read alignment mode rather than short-read bwa alignment handled here.

## Inputs

- paired-end FASTQ or FASTQ(.gz) ChIA-PET reads (bridge-linker or half-linker protocol)
- bwa genome index for the reference genome
- SAM alignment files from bwa for the two reads of each PET pair

## Outputs

- deduplicated bedpe file of filtered PETs
- tagAlign file formatted for MACS2 peak-calling input
- QC plots (QCplots.R) summarizing alignment and duplicate-removal quality
- chromosome-size tab-delimited file usage for downstream bedtools/bedpe manipulation

## How to apply

After trimLinker trims linker sequences from paired-end FASTQ(.gz) files, align reads with bwa (short-read mode; for half-linker K562 POL2 data use A/B-linker mode -m 0) and build the bedpe file with buildBedpe from the two SAM files. Apply the PET mapping-quality cutoff (MAPQ 30 in the K562 POL2 half-linker workflow) and then run removeDup with N threads to remove duplicate PETs. Convert the deduplicated bedpe to a tagAlign file with buildTagAlign for MACS2 input (default MACS2 q-value 0.05), and generate per-step quality-control figures with QCplots.R. This dedup/QC step is essential because PCR duplicates inflate PET counts at anchors and would otherwise bias MICC significance estimation of chromatin loops (MICC2.R, default CUTOFFPET=2).

## Related tools

- **ChIA-PET2** (Pipeline that integrates trimLinker, buildBedpe, removeDup, buildTagAlign and QCplots.R steps for duplicate removal and QC) — https://github.com/GuipengLi/ChIA-PET2
- **bwa** (Short-read aligner used after linker trimming to map PET reads before duplicate removal) — https://github.com/lh3/bwa
- **samtools** (SAM/BAM handling for alignment files feeding PET construction) — https://github.com/samtools/samtools
- **bedtools** (bed/bedpe file manipulation during PET filtering and deduplication) — https://github.com/arq5x/bedtools2
- **MACS2** (Consumes the buildTagAlign output (default q 0.05) for ChIP-Seq-like anchor peak calling after dedup) — https://github.com/macs3-project/MACS

## Examples

```
ChIA-PET2.sh -m 0 -i rep1_R1.fq.gz -i2 rep1_R2.fq.gz -g hg38.fa --mapq 30 --removeDup --micc
```

## Evaluation signals

- QCplots.R output shows the expected fractions of PETs passing linker trimming, alignment and duplicate-removal stages without anomalies.
- After removeDup, PCR duplicate PETs are eliminated so that downstream MACS2 peaks and MICC loop calls are not inflated by duplicate PET counts.
- On the K562 POL2 half-linker replicates (GSE72816), applying MAPQ cutoff 30 and removeDup in ChIA-PET2 A/B-linker mode (-m 0) yields >9,000 significant loops per replicate, supporting later reproducibility checks (Jaccard 52.5% ChIA-PET2 vs 25.6% Mango).
- bedpe and tagAlign outputs are schema-valid (bedpe columns, tagAlign format) for MACS2 and MICC2.R to consume without format errors.

## Limitations

- Loop calling depends on ChIP-Seq-like peak calling sensitivity: ChIA-PET2 calls chromatin loops based on ChIP-Seq-like peaks, assuming loop anchors lie at MACS2-called peaks.
- No software versions were stated for ChIA-PET2 or its required dependencies (bwa, samtools, bedtools, MACS2, g++), so exact reproduction may vary.
- Users must choose thresholds (e.g. MAPQ cutoff, MICC CUTOFFPET) to balance sensitivity and specificity of chromatin loop calling.

## Evidence
