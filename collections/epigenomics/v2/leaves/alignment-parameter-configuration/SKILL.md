---
name: alignment-parameter-configuration
description: Use when raw FASTQ reads (e.g. H3K4me3 ChIPseq or RNAseq from mouse embryonic
  stem cells) must be mapped as an automated pipeline step and per-sample aligner
  choice and parameters must be configured.
when_to_use_negative:
- Input is already aligned (.bam) data or a feature/count table — the map module expects
  raw fastq files produced by the unzip/rename step
- Quality assessment rather than alignment is needed — use NEAT's 'QC' module, which
  uses the R systemPipeR package
- Post-alignment cleanup is the actual task — use the 'cleanfiles' module to reorganize
  and store .bam and .bai files instead of re-running mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: NEAT
  role: Framework whose automated 'map' module dispatches reads to the appropriate
    aligner and writes .bam outputs
  repo: https://github.com/pschorderet/NEAT
- name: BWA
  role: Aligner invoked by the map module for ChIPseq samples
- name: Bowtie
  role: Aligner invoked by the map module for ChIPseq samples; Tophat used for RNAseq
    is bowtie-based
- name: Tophat
  role: Bowtie-based aligner dispatched for RNAseq samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:03836c31e35306628d81f50907ceb9bd03cdc1df1830236966f5adc5956e9361
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
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# alignment-parameter-configuration

## Summary

Configure and execute read alignment within NEAT's automated 'map' module, dispatching ChIPseq samples to bwa or bowtie and RNAseq samples to the bowtie-based Tophat aligner, with alignment options set in the centralized AdvancedSettings.txt.

## When to use

Use when raw FASTQ reads (e.g. H3K4me3 ChIPseq or RNAseq from mouse embryonic stem cells) must be mapped as an automated pipeline step and per-sample aligner choice and parameters must be configured.

## When NOT to use

- Input is already aligned (.bam) data or a feature/count table — the map module expects raw fastq files produced by the unzip/rename step
- Quality assessment rather than alignment is needed — use NEAT's 'QC' module, which uses the R systemPipeR package
- Post-alignment cleanup is the actual task — use the 'cleanfiles' module to reorganize and store .bam and .bai files instead of re-running mapping

## Inputs

- Truncated test fastq files (H3K4me3 ChIPseq or RNAseq from mouse embryonic stem cells) in the project 'fatsq' folder after the unzip step
- Targets.txt file listing sample names, replicates, conditions and experiment type (ChIPseq vs RNAseq)
- AdvancedSettings.txt with aligner options: maximum number of gaps, gap extension, maximum edit distance, number of threads, mismatch and gap penalty
- Steps_to_execute_pipe entry selecting the 'map' module

## Outputs

- .bam alignment files named with the user-defined 'FileName' sample names, stored in the project folder
- .bai index files reorganized by the downstream 'cleanfiles' module

## How to apply

First run the 'unzip' module so the fatsq files are unzipped, renamed and stored in a folder within the project folder. Parse Targets.txt to determine each sample's experiment type and confirm 'map' is listed in 'Steps_to_execute_pipe'; then read aligner options (maximum number of gaps, gap extension, maximum edit distance, number of threads, mismatch and gap penalty) from AdvancedSettings.txt, the same centralized file NEAT uses for alignment, filtering and peakcalling parameters. Dispatch ChIPseq samples to bwa or bowtie and RNAseq samples to the bowtie-based Tophat aligner, applying the configured parameters during read alignment. Write the resulting alignments as .bam files named consistently with the user-defined 'FileName' sample names into the project folder, so the downstream 'cleanfiles' module can reorganize the .bam and .bai files before further analysis.

## Related tools

- **NEAT** (Framework whose automated 'map' module dispatches reads to the appropriate aligner and writes .bam outputs) — https://github.com/pschorderet/NEAT
- **BWA** (Aligner invoked by the map module for ChIPseq samples)
- **Bowtie** (Aligner invoked by the map module for ChIPseq samples; Tophat used for RNAseq is bowtie-based)
- **Tophat** (Bowtie-based aligner dispatched for RNAseq samples)

## Evaluation signals

- A non-empty, valid .bam alignment file exists for every sample listed in Targets.txt
- The chosen aligner matches the sample's experiment type: bwa or bowtie for ChIPseq, Tophat for RNAseq
- Alignment parameters actually applied match the values recorded in AdvancedSettings.txt (gaps, gap extension, edit distance, threads, penalties)
- Output .bam file names are consistent with the user-defined 'FileName' sample names
- Downstream 'cleanfiles' module can find and reorganize the .bam and .bai files in the project folder

## Limitations

- Alignment settings are fixed at pipeline build time through the centralized project file, so per-sample overrides are not part of this workflow
- NEAT is described as still under development ('If you encounter any bug, please contact us') and the paper lists Mac OSx as the operating system
- Test datasets are truncated versions of mouse embryonic stem cell H3K4me3 ChIPseq or RNAseq data, so alignment statistics on them are not representative of full-scale runs

## Evidence

- [methods] map module dispatch: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [intro] automated modules including map: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] filter parameters via AdvancedSettings.txt: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [methods] cleanfiles reorganizes bam/bai: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
