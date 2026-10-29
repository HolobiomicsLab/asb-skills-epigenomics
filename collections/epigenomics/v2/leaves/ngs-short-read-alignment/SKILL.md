---
name: ngs-short-read-alignment
description: Use when raw FASTQ reads from ChIPseq (e.g. H3K4me3) or RNAseq experiments
  must be aligned as an automated pipeline step, with per-sample .bam outputs named
  after the samples in Targets.txt.
when_to_use_negative:
- Input is already an aligned .bam/.bai set stored and reorganized by the 'cleanfiles'
  module — proceed to filter/peakcalling instead
- Reads are long-read or third-generation data — NEAT's map module only dispatches
  to bwa, bowtie, or Tophat short-read aligners
- The sample type is unknown or Targets.txt lacks an experiment-type entry, since
  aligner choice (bwa/bowtie for ChIPseq vs Tophat for RNAseq) depends on it
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3308
tools:
- name: NEAT
  role: Framework whose 'map' module automates alignment dispatch and writes .bam
    outputs into the project folder
  repo: https://github.com/pschorderet/NEAT
- name: BWA
  role: Aligner used by the 'map' module for ChIPseq samples
- name: Bowtie
  role: Alternative aligner used by the 'map' module for ChIPseq samples
- name: Tophat
  role: Bowtie-based aligner dispatched by the 'map' module for RNAseq samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:83e25fbbd3854d4acecbbd647536270cc9df4a657ed3ce7228bf78bb194a2f61
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ngs-short-read-alignment

## Summary

Automated short-read alignment of ChIPseq/RNAseq FASTQ files using NEAT's 'map' module, which dispatches ChIPseq samples to bwa or bowtie and RNAseq samples to the bowtie-based Tophat, reading aligner parameters from AdvancedSettings.txt and writing per-sample .bam files.

## When to use

Use when raw FASTQ reads from ChIPseq (e.g. H3K4me3) or RNAseq experiments must be aligned as an automated pipeline step, with per-sample .bam outputs named after the samples in Targets.txt.

## When NOT to use

- Input is already an aligned .bam/.bai set stored and reorganized by the 'cleanfiles' module — proceed to filter/peakcalling instead
- Reads are long-read or third-generation data — NEAT's map module only dispatches to bwa, bowtie, or Tophat short-read aligners
- The sample type is unknown or Targets.txt lacks an experiment-type entry, since aligner choice (bwa/bowtie for ChIPseq vs Tophat for RNAseq) depends on it

## Inputs

- FASTQ read files (truncated H3K4me3 ChIPseq or RNAseq test data from mouse embryonic stem cells) in the project 'fatsq' folder
- Targets.txt file specifying each sample's experiment type and pipeline steps
- AdvancedSettings.txt with aligner parameters (gaps, gap extension, edit distance, threads, mismatch/gap penalties)

## Outputs

- .bam alignment files per sample, named by the user-defined 'FileName' entries
- .bai index files (organized by the downstream 'cleanfiles' module)

## How to apply

Load the unzipped, renamed fastq files (NEAT's 'fatsq' folder) produced by the 'unzip' module, then parse Targets.txt to determine each sample's experiment type and confirm 'map' appears in 'Steps_to_execute_pipe'. Read aligner options — maximum number of gaps, gap extension, maximum edit distance, number of threads, mismatch and gap penalty — from the centralized AdvancedSettings.txt file. Dispatch ChIPseq samples to bwa or bowtie and RNAseq samples to Tophat (bowtie-based), applying the configured parameters during read alignment. Write the resulting alignments as .bam files named consistently with the user-defined 'FileName' sample names into the project folder so the downstream 'cleanfiles' module can reorganize them alongside .bai files. The rationale is NEAT's vertical integration: alignment parameters live in one centralized project file so non-expert users never manipulate files manually.

## Related tools

- **NEAT** (Framework whose 'map' module automates alignment dispatch and writes .bam outputs into the project folder) — https://github.com/pschorderet/NEAT
- **BWA** (Aligner used by the 'map' module for ChIPseq samples)
- **Bowtie** (Alternative aligner used by the 'map' module for ChIPseq samples)
- **Tophat** (Bowtie-based aligner dispatched by the 'map' module for RNAseq samples)

## Evaluation signals

- A non-empty, valid .bam alignment file exists for every sample listed in Targets.txt
- The chosen aligner matches the sample's experiment type: bwa or bowtie for ChIPseq, Tophat (bowtie-based) for RNAseq
- Aligned .bam files carry names consistent with the user-defined 'FileName' sample names from Targets.txt
- Aligner invocations reflect the parameters read from AdvancedSettings.txt (max gaps, gap extension, max edit distance, threads, mismatch/gap penalties)
- The 'cleanfiles' module can successfully reorganize the .bam files alongside their .bai index files in the project folder

## Limitations

- NEAT development mode: the README notes 'NEAT is still under developent. If you encounter any bug, please contact us.'
- Operating system limited to Mac OSx per the article's discussion; no Linux/Windows support stated
- Test datasets are truncated (H3K4me3 ChIPseq or RNAseq from mouse embryonic stem cells), so alignment behavior on full-size or other organisms is not demonstrated in the bundled tutorial

## Evidence

- [methods] The ‘map’ module maps reads using either bwa [18] or bowtie [19].: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [methods] the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
