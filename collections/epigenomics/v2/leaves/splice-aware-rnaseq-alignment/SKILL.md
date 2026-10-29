---
name: splice-aware-rnaseq-alignment
description: Use when RNAseq fastq files in a NEAT project's 'fatsq' folder need alignment
  and Targets.txt marks samples as RNAseq, requiring splice-aware mapping with Tophat.
when_to_use_negative:
- Input samples are ChIPseq (e.g. H3K4me3), which NEAT routes to bwa or bowtie rather
  than splice-aware Tophat
- Input is already aligned data (.bam), where the 'filter', 'peakcalling', 'GRanges'
  or 'cleanfiles' modules are appropriate instead
- The experiment requires a user-scripted, terminal-driven workflow rather than NEAT's
  double-clickable, GUI-based pipeline approach
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NEAT
  role: framework whose automated 'map' module parses Targets.txt and AdvancedSettings.txt
    and dispatches RNAseq samples to Tophat
  repo: https://github.com/pschorderet/NEAT
- name: Tophat
  role: bowtie-based splice-aware aligner invoked for RNAseq read alignment
- name: Bowtie
  role: short-read aligner underlying Tophat and one of the two aligners used for
    ChIPseq samples in the same module
- name: systemPipeR
  role: R package used by NEAT's 'QC' module, separate from the map module
  repo: https://github.com/tgirke/systemPipeR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:3f2499d8ce5b890b85df0806c44255b05bb3e700d020aa4e7439f5f1a2fd4a41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bowtie
  - NEAT
  - Tophat
  - systemPipeR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# splice-aware-rnaseq-alignment

## Summary

Performs automated read alignment for RNAseq samples by dispatching NEAT's 'map' module to the bowtie-based Tophat aligner, which handles splice junctions across the transcriptome, while ChIPseq samples are routed to bwa or bowtie instead.

## When to use

Use when RNAseq fastq files in a NEAT project's 'fatsq' folder need alignment and Targets.txt marks samples as RNAseq, requiring splice-aware mapping with Tophat.

## When NOT to use

- Input samples are ChIPseq (e.g. H3K4me3), which NEAT routes to bwa or bowtie rather than splice-aware Tophat
- Input is already aligned data (.bam), where the 'filter', 'peakcalling', 'GRanges' or 'cleanfiles' modules are appropriate instead
- The experiment requires a user-scripted, terminal-driven workflow rather than NEAT's double-clickable, GUI-based pipeline approach

## Inputs

- RNAseq fastq files (e.g. truncated mouse embryonic stem cell test dataset) in the NEAT 'fatsq' folder
- Targets.txt listing sample names, replicates, conditions and experiment type
- AdvancedSettings.txt with aligner options (max gaps, gap extension, max edit distance, threads, mismatch/gap penalties)
- Steps_to_execute_pipe entry selecting the 'map' module

## Outputs

- Non-empty valid .bam alignment files named per the user-defined 'FileName' sample names
- .bai index files reorganized by the downstream 'cleanfiles' module

## How to apply

Load the truncated test dataset fastq files (e.g. RNAseq data from mouse embryonic stem cells) into the project's 'fatsq' folder after the unzip step. Parse Targets.txt to confirm each sample's experiment type and the 'map' entry in 'Steps_to_execute_pipe', then read aligner options — maximum number of gaps, gap extension, maximum edit distance, number of threads, mismatch penalty and gap penalty — from AdvancedSettings.txt. NEAT dispatches RNAseq samples to the bowtie-based Tophat aligner, applying those configured parameters, while ChIPseq samples in the same pipeline are sent to bwa or bowtie. Write resulting alignments as .bam files named with the user-defined 'FileName' sample names into the project folder; the downstream 'cleanfiles' module then reorganizes them alongside .bai files. The central file containing alignment-, filtering- and peak calling parameters keeps the whole step reproducible.

## Related tools

- **NEAT** (framework whose automated 'map' module parses Targets.txt and AdvancedSettings.txt and dispatches RNAseq samples to Tophat) — https://github.com/pschorderet/NEAT
- **Tophat** (bowtie-based splice-aware aligner invoked for RNAseq read alignment)
- **Bowtie** (short-read aligner underlying Tophat and one of the two aligners used for ChIPseq samples in the same module)
- **systemPipeR** (R package used by NEAT's 'QC' module, separate from the map module) — https://github.com/tgirke/systemPipeR

## Evaluation signals

- A non-empty, valid .bam file exists for every sample listed in Targets.txt
- Each RNAseq sample's alignment was produced by Tophat while each ChIPseq sample used bwa or bowtie, matching experiment type
- Bam file names match the user-defined 'FileName' values in Targets.txt
- Configured AdvancedSettings.txt aligner parameters (gaps, gap extension, max edit distance, threads, penalties) were honored by the aligner
- .bai index files accompany the .bam files after the 'cleanfiles' module reorganizes the project folder

## Limitations

- NEAT is restricted to Mac OSx (operating system: Mac OSx); no Linux/Windows support is stated
- The aligner decision is driven by the experiment type recorded in Targets.txt, so mislabeled samples are dispatched to the wrong aligner
- Aligner behavior depends entirely on the values the user sets in the centralized AdvancedSettings.txt file

## Evidence

- [methods] The 'map' module maps reads using either bwa [18] or bowtie [19].: "The ‘map’ module maps reads using either bwa [18] or bowtie [19]."
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [methods] The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [methods] the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
