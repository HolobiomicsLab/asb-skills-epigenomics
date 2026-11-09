---
name: targets-file-configuration-for-ngs-workflows
description: Use when building a NEAT project from FASTQ inputs (e.g., truncated H3K4me3
  ChIPseq or RNAseq data from mouse embryonic stem cells) and you must declare samples,
  replicates, and which NEAT modules (unzip, QC, chiprx, map, filter, peakcalling,
  cleanfiles, granges) to execute.
when_to_use_negative:
- The analysis is not a NEAT-managed ChIPseq/RNAseq FASTQ pipeline (e.g., data are
  already processed feature counts or a non-NGS assay).
- You need a workflow engine on Linux or Windows; the article states NEAT's operating
  system is limited to Mac OSx.
- The pipeline requires tools outside NEAT's declared modules (unzip, rename, QC,
  chiprx, map, filter, peakcalling, wig, GRanges).
edam_operation: http://edamontology.org/operation_2409
tools:
- name: NEAT
  role: Framework whose 'New Project' application and Targets.txt file define the
    pipeline; double-clickable executables launch the declared module chain.
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R package invoked by NEAT's 'QC' module as declared in the Targets.txt pipeline
    steps.
  repo: https://github.com/tgirke/systemPipeR
- name: bwa
  role: One of the two aligners selectable through the 'map' module in Targets.txt
    for read mapping.
- name: samtools
  role: Underlies the 'filter' module's duplicate-read handling and fragment size
    filtering configured via AdvancedSettings.txt.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:3ac15ee15581c11cefbcea6e1db790cd50398ac747c7386588159acf24c588c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - MACS
  - NEAT
  - R
  - SPP
  - Tophat
  - bwa
  - samtools
  - systemPipeR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# targets-file-configuration-for-ngs-workflows

## Summary

Configure the single centralized Targets.txt file that drives a NEAT NGS pipeline (ChIPseq/RNAseq), encoding sample names, replicates, conditions, antibodies, alignment/filtering/peak-calling parameters, cluster settings, and the 'Steps_to_execute_pipe' module blocks so the full pipeline runs from one declarative input.

## When to use

Use when building a NEAT project from FASTQ inputs (e.g., truncated H3K4me3 ChIPseq or RNAseq data from mouse embryonic stem cells) and you must declare samples, replicates, and which NEAT modules (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) to execute.

## When NOT to use

- The analysis is not a NEAT-managed ChIPseq/RNAseq FASTQ pipeline (e.g., data are already processed feature counts or a non-NGS assay).
- You need a workflow engine on Linux or Windows; the article states NEAT's operating system is limited to Mac OSx.
- The pipeline requires tools outside NEAT's declared modules (unzip, rename, QC, chiprx, map, filter, peakcalling, wig, GRanges).

## Inputs

- FASTQ read files (possibly gzipped, hence the unzip module)
- Sample metadata: sample names, replicates, conditions, antibodies
- Alignment, filtering, and peak-calling parameter choices (AdvancedSettings.txt)
- Cluster settings (qsub/bsub, storage space, maximum job submissions, wall time)

## Outputs

- NEAT project folder with renamed FASTQ files (from the 'unzip' module)
- QC reports generated via systemPipeR
- .bam and .bai files reorganized by the 'cleanfiles' module
- GRanges R objects and .wig wiggle files
- Peak calls from MACS or SPP

## How to apply

Create the project with NEAT's 'New Project' application, then fill in the centralized Targets.txt with sample names, replicates, conditions, antibodies, and cluster settings, and write the 'Steps_to_execute_pipe' blocks naming the desired module chain (e.g., unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges). The QC module pulls in the R systemPipeR package, mapping is set to bwa or bowtie (Tophat for RNAseq), and filtering parameters (duplicate-read handling, minimum and maximum fragment size) are specified in AdvancedSettings.txt. Peak calling algorithm (MACS or SPP) is chosen in the peakcalling module block. Because the file is the sole centralized control point, editing Targets.txt rather than individual scripts is how sample metadata and pipeline topology are changed; launching the double-clickable executable then executes all declared modules and produces .bam/.bai files, GRanges objects, and wiggle files in the project folder.

## Related tools

- **NEAT** (Framework whose 'New Project' application and Targets.txt file define the pipeline; double-clickable executables launch the declared module chain.) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R package invoked by NEAT's 'QC' module as declared in the Targets.txt pipeline steps.) — https://github.com/tgirke/systemPipeR
- **bwa** (One of the two aligners selectable through the 'map' module in Targets.txt for read mapping.)
- **samtools** (Underlies the 'filter' module's duplicate-read handling and fragment size filtering configured via AdvancedSettings.txt.)

## Evaluation signals

- Every module listed in the 'Steps_to_execute_pipe' block (e.g., unzip + QC + chiprx + map + filter + peakcalling + cleanfiles + granges) completes without manual file manipulation between steps.
- All three declared output classes are produced per dataset: .bam files, GRanges objects, and .wig files.
- File-size ordering matches expectations: wig files smallest (~4–6 Mb), GRanges intermediate (~40–60 Mb), bam files largest (~4–6 Gb for full-scale data).
- Truncated test datasets complete the full pipeline in under two hours (well within the <24 h whole-run claim).
- Renamed FASTQs from the 'unzip' module land in a newly created folder within the project folder.

## Limitations

- NEAT is limited to Mac OSx; no Linux/Windows support is stated.
- NEAT is still under development per the README; bugs should be reported, and no changelog is provided.
- No version number is stated for the software, complicating reproducibility of a configured run.

## Evidence

- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] The 'filter' module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [methods] The 'QC' module uses the R systemPipeR package: "The ‘QC’ module uses the R systemPipeR package"
- [discussion] NEAT, including tutorials and test data, is publicly available on GitHub (https://github.com/pschorderet/NEAT).: "NEAT, including tutorials and test data, is publicly available on GitHub (https://github.com/pschorderet/NEAT)."
