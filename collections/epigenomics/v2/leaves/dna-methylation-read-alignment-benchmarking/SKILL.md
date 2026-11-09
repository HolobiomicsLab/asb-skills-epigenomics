---
name: dna-methylation-read-alignment-benchmarking
description: Decision support for choosing a methylation read aligner, or when reproducing/comparing
  alignment speed, memory, mapping efficiency, and accuracy across 3-letter and 4-letter
  methylation protocols.
when_to_use_negative:
- Evaluating aligners on real, chemically imperfect BS-seq/TAPS data — the benchmark
  uses simulated reads with defined conversion rates and the article notes real TAPS
  data had imperfect chemical treatment
- Assessing downstream methylation-call or visualization quality of the toolkit —
  this skill benchmarks alignment only, ending at mapping metrics
- Data from protocols other than TAPS/BS or 5hmC-CATCH — 4-letter mode only supports
  TAPS/5hmC-CATCH with low non-CpG methylation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: Msuite
  role: Aligner under test in both 3-letter (-3) and 4-letter (-4) TAPS modes; also
    provides build.index.sh for index construction and performs adapter-/quality-trimming
  repo: https://github.com/hellosunking/Msuite/
- name: Bismark
  role: 3-letter bisulfite aligner baseline run with default settings (v0.22.3)
- name: BWA-meth
  role: 3-letter wild-card aligner baseline run with default settings (v0.2.2)
- name: SHERMAN script
  role: Generates in-silico paired-end TAPS-protocol reads with defined conversion
    and error rates
- name: bowtie2
  role: Underlying aligner engine used by both Msuite and Bismark
  repo: https://github.com/BenLangmead/bowtie2
- name: samtools
  role: Converts SAM alignment output into BAM format
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100127
    title: 'Msuite: A High-Performance and Versatile DNA Methylation Data-Analysis
      Toolkit'
  source_content_hash: sha256:7095cceb42799e2028691dee66c96cb7c747a8116cf27b9c8a7b3052f76309b1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hellosunking/Msuite
  repo_url: https://github.com/hellosunking/Msuite
  tools:
  - BWA-meth
  - Bismark
  - Msuite
  - SHERMAN script
  - bowtie2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# dna-methylation-read-alignment-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark workflow for comparing DNA methylation read aligners (Msuite in 3-letter and 4-letter TAPS modes, Bismark, BWA-meth) on simulated bisulfite-free reads, measuring mapping efficiency, accuracy, speed, and peak memory.

## When to use

Decision support for choosing a methylation read aligner, or when reproducing/comparing alignment speed, memory, mapping efficiency, and accuracy across 3-letter and 4-letter methylation protocols.

## When NOT to use

- Evaluating aligners on real, chemically imperfect BS-seq/TAPS data — the benchmark uses simulated reads with defined conversion rates and the article notes real TAPS data had imperfect chemical treatment
- Assessing downstream methylation-call or visualization quality of the toolkit — this skill benchmarks alignment only, ending at mapping metrics
- Data from protocols other than TAPS/BS or 5hmC-CATCH — 4-letter mode only supports TAPS/5hmC-CATCH with low non-CpG methylation

## Inputs

- Reference genome FASTA (e.g., GRCh38) plus RefSeq annotation
- In-silico paired-end 100-bp and 36-bp TAPS-protocol FASTQ read sets (1 million read pairs each) generated with SHERMAN
- Precompiled aligner binaries: Msuite 1.1.x, Bismark v0.22.3, BWA-meth v0.2.2, bowtie2, samtools

## Outputs

- Aggregated benchmark table (e.g., table2_benchmark.tsv) with one row per tool/mode and columns for read length, mapping efficiency (%), accuracy (%), running time, and peak memory
- Msuite alignment outputs from 3-letter and 4-letter TAPS runs (SAM/BAM alignments and mapping statistics)

## How to apply

Generate in-silico paired-end 100-bp and 36-bp TAPS-protocol read pairs (1 million pairs each) with the SHERMAN script using 20% C->T conversion at CpG sites, 0.5% at CpH sites, and 0.1% sequencing error, sampling from CT/GA-rich regions (CT or GA proportion >=80%). Build genome indices with Msuite's build.index.sh supplying the reference FASTA (e.g., GRCh38) and RefSeq annotation, then run Msuite in 3-letter (-3 -m TAPS) and 4-letter (-4 -m TAPS) modes by launching the generated makefile in the output directory. Run Bismark v0.22.3 and BWA-meth v0.2.2 on the same reads with default settings for the 3-letter comparison, counting Msuite's adapter-/quality-trimming time in its total. Profile running time and peak memory with /usr/bin/time and aggregate mapping efficiency, accuracy, time, and memory into a per-tool/mode table.

## Related tools

- **Msuite** (Aligner under test in both 3-letter (-3) and 4-letter (-4) TAPS modes; also provides build.index.sh for index construction and performs adapter-/quality-trimming) — https://github.com/hellosunking/Msuite/
- **Bismark** (3-letter bisulfite aligner baseline run with default settings (v0.22.3))
- **BWA-meth** (3-letter wild-card aligner baseline run with default settings (v0.2.2))
- **SHERMAN script** (Generates in-silico paired-end TAPS-protocol reads with defined conversion and error rates)
- **bowtie2** (Underlying aligner engine used by both Msuite and Bismark) — https://github.com/BenLangmead/bowtie2
- **samtools** (Converts SAM alignment output into BAM format)

## Examples

```
build.index/build.index.sh GENOME.FA REFSEQ.txt GRCh38 && msuite -x GRCh38 -1 reads_R1.fq -2 reads_R2.fq -4 -m TAPS -o out.dir
```

## Evaluation signals

- Mapping efficiency reproduces Table 2 values within rounding tolerance: Msuite 3-letter 96.19%, Msuite 4-letter 95.75%, Bismark 95.79%, BWA-meth 95.58% on 100-bp reads
- Msuite shows lower total running time than Bismark and BWA-meth even though its trimming step time is included
- Msuite peak memory usage is lower than both Bismark and BWA-meth despite using the same bowtie2 engine
- Simulated read QC: conversion rates match SHERMAN parameters (20% CpG, 0.5% CpH) and reads are restricted to CT/GA-rich regions (>=80% CT or GA)
- Benchmark table has exactly one row per tool/mode with non-missing efficiency, accuracy, time, and memory values

## Limitations

- Four in-silico datasets only (paired-/single-end 36-/100-bp); results may not generalize to other read lengths or real-data artifacts
- No strong evidence that 4-letter mode is more accurate on inconsistent CpG sites; further validation needed
- Accuracy of 4- and 3-letter modes should be validated against orthogonal methods such as microarrays; real TAPS data had imperfect chemical treatment

## Evidence
