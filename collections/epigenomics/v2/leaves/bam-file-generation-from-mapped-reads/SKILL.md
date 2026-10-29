---
name: bam-file-generation-from-mapped-reads
description: Use when raw fastq files (e.g. H3K4me3 ChIPseq or RNAseq from mouse embryonic
  stem cells) must be aligned as an automated pipeline step and .bam outputs are needed
  for downstream QC, filtering, or peak calling.
when_to_use_negative:
- The input is already aligned — e.g. .bam files exist and only filtering (duplicate
  handling, fragment size limits) or peak calling is required
- The sample is RNAseq that requires a splice-aware aligner other than the bowtie-based
  Tophat dispatch that the 'map' module provides
- The pipeline does not use NEAT and the user needs a standalone aligner invocation
  outside Targets.txt / AdvancedSettings.txt configuration
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3169
tools:
- name: NEAT
  role: Framework whose automated 'map' module dispatches reads to the correct aligner
    and writes .bam files per sample
  repo: https://github.com/pschorderet/NEAT
- name: BWA
  role: Aligner dispatched by the 'map' module for ChIPseq samples
- name: Bowtie
  role: Alternative aligner dispatched by the 'map' module for ChIPseq samples
- name: Tophat
  role: Bowtie-based aligner dispatched by the 'map' module for RNAseq samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:d542a9988d5617f1cd3a93166d9e3a9cae5295bb290ecf5ffe78ea3afa59e326
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - BWA
  - Bowtie
  - NEAT
  - Tophat
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bam-file-generation-from-mapped-reads

## Summary

Generates .bam alignment files by mapping ChIPseq and RNAseq reads with NEAT's 'map' module, which dispatches ChIPseq samples to bwa or bowtie and RNAseq samples to the bowtie-based Tophat aligner. It converts the raw fastq inputs from the 'fatsq' folder into alignment files that downstream filtering and peak-calling modules consume.

## When to use

Use when raw fastq files (e.g. H3K4me3 ChIPseq or RNAseq from mouse embryonic stem cells) must be aligned as an automated pipeline step and .bam outputs are needed for downstream QC, filtering, or peak calling.

## When NOT to use

- The input is already aligned — e.g. .bam files exist and only filtering (duplicate handling, fragment size limits) or peak calling is required
- The sample is RNAseq that requires a splice-aware aligner other than the bowtie-based Tophat dispatch that the 'map' module provides
- The pipeline does not use NEAT and the user needs a standalone aligner invocation outside Targets.txt / AdvancedSettings.txt configuration

## Inputs

- Truncated test fastq files (H3K4me3 ChIPseq or RNAseq from mouse embryonic stem cells) in the 'fatsq' folder
- Targets.txt defining sample names, replicates, conditions, and experiment type
- AdvancedSettings.txt with aligner parameters (max gaps, gap extension, max edit distance, threads, mismatch and gap penalty)
- 'Steps_to_execute_pipe' entry enabling the 'map' module

## Outputs

- .bam alignment files named per the user-defined 'FileName' sample names
- .bam files staged in the project folder for the 'cleanfiles' module to reorganize with .bai index files

## How to apply

Load the unzipped fastq files from the project's 'fatsq' folder and parse Targets.txt to determine each sample's experiment type and the 'map' entry in 'Steps_to_execute_pipe'. Read the aligner options — maximum number of gaps, gap extension, maximum edit distance, number of threads, and mismatch and gap penalty — from AdvancedSettings.txt, the centralized settings file NEAT projects are built around. Dispatch ChIPseq samples to bwa or bowtie and RNAseq samples to the bowtie-based Tophat aligner, applying the configured parameters during read alignment. Write the resulting alignments as .bam files named consistently with the user-defined 'FileName' sample names in the project folder, so the 'cleanfiles' module can later reorganize them alongside .bai index files. Validate by confirming a non-empty, valid .bam exists for every sample in Targets.txt and that the aligner matches the sample's experiment type.

## Related tools

- **NEAT** (Framework whose automated 'map' module dispatches reads to the correct aligner and writes .bam files per sample) — https://github.com/pschorderet/NEAT
- **BWA** (Aligner dispatched by the 'map' module for ChIPseq samples)
- **Bowtie** (Alternative aligner dispatched by the 'map' module for ChIPseq samples)
- **Tophat** (Bowtie-based aligner dispatched by the 'map' module for RNAseq samples)

## Evaluation signals

- A non-empty, valid .bam alignment file exists for every sample listed in Targets.txt
- The chosen aligner matches the sample's experiment type: bwa or bowtie for ChIPseq, Tophat (bowtie-based) for RNAseq
- .bam file names match the user-defined 'FileName' sample names in Targets.txt
- Configured parameters from AdvancedSettings.txt (maximum number of gaps, gap extension, maximum edit distance, threads, mismatch and gap penalty) were applied to the alignment
- .bam and .bai files are correctly staged so the 'cleanfiles' module can reorganize them for downstream analysis

## Limitations

- NEAT was documented as running on Mac OSx only; no Linux/Windows support was stated
- The bundled test datasets are truncated H3K4me3 ChIPseq or RNAseq data from mouse embryonic stem cells, so behavior on full-size or other data types must be verified separately
- Only the ChIPseq/RNAseq dispatch defined in the 'map' module (bwa, bowtie, Tophat) is available; other aligners require pipeline modification

## Evidence

- [methods] The ‘map’ module maps reads using either bwa [18] or bowtie [19].: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
