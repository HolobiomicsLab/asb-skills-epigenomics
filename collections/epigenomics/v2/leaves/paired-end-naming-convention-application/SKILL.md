---
name: paired-end-naming-convention-application
description: Reference for setting up a NEAT project with paired-end sequencing data
  where reverse-read FASTQ files must be linked to their forward mates inside the
  centralized Targets.txt file.
when_to_use_negative:
- Data are single-end reads — no '_R2' reverse-mate entries should be added
- Running a non-NEAT pipeline that expects its own sample-sheet format (e.g. systemPipeR's
  own workflow control files) rather than the Targets.txt convention
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: NEAT
  role: Framework whose Targets.txt file enforces the '_R2' paired-end naming convention
    and consumes it during pipeline construction
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R package used by NEAT's QC module within the pipeline that the Targets.txt
    file configures
  repo: https://github.com/tgirke/systemPipeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:232545ed680baab2b88db6d42044a192c687196c167d60918bdc8008b462f1c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - systemPipeR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# paired-end-naming-convention-application

## Summary

Applies NEAT's Targets.txt naming convention for paired-end NGS runs: reverse-read FASTQ files are named as the forward-read name followed by '_R2' and listed under the 'PE corresponding samples' mark, so the pipeline can pair reads without extra configuration.

## When to use

Reference for setting up a NEAT project with paired-end sequencing data where reverse-read FASTQ files must be linked to their forward mates inside the centralized Targets.txt file.

## When NOT to use

- Data are single-end reads — no '_R2' reverse-mate entries should be added
- Running a non-NEAT pipeline that expects its own sample-sheet format (e.g. systemPipeR's own workflow control files) rather than the Targets.txt convention

## Inputs

- Compressed FASTQ files (forward and reverse reads) from paired-end sequencing runs
- Targets.txt template from the NEAT package
- Sample metadata (sample names, replicate and sample-to-input relationships, antibody specificities)

## Outputs

- Completed Targets.txt with 'OriFileName', 'FileName' and '_R2'-suffixed paired-end entries under 'PE corresponding samples'
- Project directory ready for NEAT pipeline execution (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges blocks)

## How to apply

Start from the Targets.txt template shipped with the NEAT package, copied into the newly created project directory. In the 'SAMPLES INFO' section, enter the compressed FASTQ file names under 'OriFileName' and user-defined sample names under 'FileName'. For paired-end runs, rename each reverse-read file as the forward-read name followed by '_R2' and list these entries below the 'PE corresponding samples' mark so NEAT can match mates by exact string convention. Complete the remaining fields (sample relationships such as replicate and sample-to-input, antibody, mapping algorithm bwa or bowtie, filtering parameters in AdvancedSettings.txt, peak caller MACS or SPP, and the 'Steps_to_execute_pipe' plain-word blocks) so the file is fully valid before executing the pipeline.

## Related tools

- **NEAT** (Framework whose Targets.txt file enforces the '_R2' paired-end naming convention and consumes it during pipeline construction) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R package used by NEAT's QC module within the pipeline that the Targets.txt file configures) — https://github.com/tgirke/systemPipeR

## Evaluation signals

- Every paired-end sample has a matching reverse-read entry whose name equals the forward 'FileName' plus the suffix '_R2'
- All '_R2' entries appear below the 'PE corresponding samples' mark in the 'SAMPLES INFO' section
- Each 'OriFileName' corresponds to an existing compressed FASTQ file in the project directory
- Every pipeline block listed on 'Steps_to_execute_pipe' exactly matches NEAT's plain-word module names (exact word matching)
- Targets.txt contains all required fields: sample names, relationships, antibody, alignment/filtering/peak-calling parameters and cluster paths

## Limitations

- NEAT uses exact word matching on module names, so any renaming or deletion in Targets.txt must still preserve valid plain-word blocks
- If no spike-in control is used, all ChIP-RX parameters must be set to '-' rather than omitted
- Operating system limited to Mac OSx per the discussion section

## Evidence

- [methods] Paired-end naming under 'PE corresponding samples': [evidence span withheld — anchor status: not_found]
- [abstract] Centralized project file contents: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [intro] Workflow blocks in Targets.txt: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] Pipeline module composition: "NEAT has been developed as four main modules: [ 1 ] Create a new project [ 2 ] Build a pipeline"
