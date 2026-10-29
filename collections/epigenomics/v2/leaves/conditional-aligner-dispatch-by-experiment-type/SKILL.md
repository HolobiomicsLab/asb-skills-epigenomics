---
name: conditional-aligner-dispatch-by-experiment-type
description: Use when building an automated ChIPseq/RNAseq mapping step where samples
  in a project must be routed to bwa, bowtie, or Tophat according to the experiment
  type recorded per sample.
when_to_use_negative:
- The data are single-cell or long-read reads for which bwa/bowtie/Tophat are not
  the intended aligners
- The sample is not a ChIPseq or RNAseq library, since the experiment-type dispatch
  is defined only for those two NEAT pipeline types
- Reads are already aligned (e.g. BAM input), as this skill maps raw fastq files listed
  in Targets.txt
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NEAT
  role: framework whose 'map' module performs the automated dispatch and executes
    alignment within the pipeline
  repo: https://github.com/pschorderet/NEAT
- name: BWA
  role: aligner used for ChIPseq samples dispatched by the 'map' module
- name: Bowtie
  role: alternative aligner for ChIPseq samples dispatched by the 'map' module
- name: Tophat
  role: bowtie-based aligner used for RNAseq samples dispatched by the 'map' module
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:76780c1aea8cbd22eb8316cad220e4b2fd13fbd2710808b3bfe2aa13623fb348
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Conditional aligner dispatch by experiment type

## Summary

Implements NEAT's read-alignment logic in which the 'map' module dispatches each sample to a different aligner based on its experiment type: bwa or bowtie for ChIPseq FASTQ reads and the bowtie-based Tophat for RNAseq, reading all parameters from AdvancedSettings.txt and Targets.txt.

## When to use

Use when building an automated ChIPseq/RNAseq mapping step where samples in a project must be routed to bwa, bowtie, or Tophat according to the experiment type recorded per sample.

## When NOT to use

- The data are single-cell or long-read reads for which bwa/bowtie/Tophat are not the intended aligners
- The sample is not a ChIPseq or RNAseq library, since the experiment-type dispatch is defined only for those two NEAT pipeline types
- Reads are already aligned (e.g. BAM input), as this skill maps raw fastq files listed in Targets.txt

## Inputs

- FASTQ read files (fatsq folder, e.g. truncated H3K4me3 ChIPseq or RNAseq data from mouse embryonic stem cells)
- Targets.txt file listing sample names, replicates, conditions, antibodies and experiment type
- AdvancedSettings.txt with aligner parameters (max gaps, gap extension, max edit distance, threads, mismatch/gap penalty)
- Steps_to_execute_pipe entry selecting the 'map' module

## Outputs

- .bam alignment files named per the user-defined 'FileName' sample names, stored in the project folder
- .bai index files reorganized by the 'cleanfiles' module

## How to apply

Parse the project's Targets.txt file to determine each sample's experiment type (ChIPseq vs RNAseq) and read the 'map' entry in 'Steps_to_execute_pipe' to confirm mapping is active. Load aligner options (maximum number of gaps, gap extension, maximum edit distance, number of threads, mismatch and gap penalty) from AdvancedSettings.txt rather than hardcoding them. For each sample, load the fastq file from the 'fatsq' folder created by the unzip module and dispatch: ChIPseq samples go to bwa or bowtie; RNAseq samples go to the bowtie-based Tophat aligner. Write the resulting alignments as .bam files named consistently with the user-defined 'FileName' sample names so the downstream 'cleanfiles' module can reorganize them alongside .bai files. The rationale is that NEAT automates this dispatch so non-expert users never invoke aligners manually.

## Related tools

- **NEAT** (framework whose 'map' module performs the automated dispatch and executes alignment within the pipeline) — https://github.com/pschorderet/NEAT
- **BWA** (aligner used for ChIPseq samples dispatched by the 'map' module)
- **Bowtie** (alternative aligner for ChIPseq samples dispatched by the 'map' module)
- **Tophat** (bowtie-based aligner used for RNAseq samples dispatched by the 'map' module)

## Evaluation signals

- A non-empty, valid .bam alignment file exists for every sample listed in Targets.txt
- The chosen aligner matches the sample's experiment type: bwa or bowtie for ChIPseq, Tophat for RNAseq
- Alignment parameters used match the values configured in AdvancedSettings.txt (max gaps, gap extension, max edit distance, threads, mismatch and gap penalty)
- Output .bam file names match the user-defined 'FileName' sample names
- .bai companion index files are present after the 'cleanfiles' module reorganizes the project folder

## Limitations

- NEAT and this mapping step are documented only for Mac OSx; no Linux/Windows support is stated
- Only bwa, bowtie and the bowtie-based Tophat are supported dispatch targets; no plug-in aligners are described
- The bundled test fastq datasets are truncated, so mapping results are representative rather than full-scale
- NEAT is still under development and users are asked to report bugs encountered

## Evidence

- [methods] map module dispatch: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [intro] automated modules list: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] pipeline modules: "(iv)	Map

(v)		Filter

(vi)	Peakcalling"
- [methods] bam reorganization: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
