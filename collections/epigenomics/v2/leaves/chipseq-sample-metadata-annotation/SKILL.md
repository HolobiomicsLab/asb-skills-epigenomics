---
name: chipseq-sample-metadata-annotation
description: Use when setting up a new NEAT ChIPseq project on an institution's cluster
  and raw compressed fastq files must be mapped to user-defined sample names, replicates,
  antibodies and pipeline blocks before execution.
when_to_use_negative:
- The analysis will be run outside NEAT (e.g. on a Galaxy-style web platform) — Targets.txt
  is NEAT's own centralized project file and exact-word matching will not transfer
- Raw fastq files and sample-to-input/replicate relationships are unavailable or the
  data are already processed past the 'unzip' stage, so the metadata file cannot anchor
  the pipeline
- The working environment is not the institution's cluster under Mac OSx, the only
  operating system supported in the article's setup
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: NEAT
  role: Framework whose four modules (create project, build pipeline, transfer, analyze)
    consume Targets.txt; the file controls which automated modules (unzip, rename,
    QC, chiprx, map, filter, peakcalling) run
  repo: https://github.com/pschorderet/NEAT
- name: systemPipeR
  role: R/Bioconductor package invoked by NEAT's 'QC' module, one of the pipeline
    blocks selected in Targets.txt
  repo: https://github.com/tgirke/systemPipeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:cbc6d435d471807751d72ab6c350b2be981bb5185bae65f5ad45551b74e05f3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  - systemPipeR
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chipseq-sample-metadata-annotation

## Summary

Reconstructs and populates NEAT's centralized Targets.txt project file, which encodes sample names, replicates, sample-to-input relationships, H3K4me3 antibody specificities, and cluster-specific parameters as the single source of metadata for an automated ChIPseq pipeline.

## When to use

Use when setting up a new NEAT ChIPseq project on an institution's cluster and raw compressed fastq files must be mapped to user-defined sample names, replicates, antibodies and pipeline blocks before execution.

## When NOT to use

- The analysis will be run outside NEAT (e.g. on a Galaxy-style web platform) — Targets.txt is NEAT's own centralized project file and exact-word matching will not transfer
- Raw fastq files and sample-to-input/replicate relationships are unavailable or the data are already processed past the 'unzip' stage, so the metadata file cannot anchor the pipeline
- The working environment is not the institution's cluster under Mac OSx, the only operating system supported in the article's setup

## Inputs

- Targets.txt template file from the NEAT package
- Compressed FASTQ files (forward reads, plus '_R2'-suffixed reverse reads for paired-end runs)
- Sample metadata: user-defined sample names, replicate relationships, sample-to-input controls, H3K4me3 antibody specificities
- Cluster configuration values (queuing priority, node allocation, CPU count, wall time, virtual paths to reference genomes)
- AdvancedSettings.txt filtering parameters (duplicate-read handling, minimum and maximum fragment size)

## Outputs

- Completed project-specific Targets.txt containing sample info, alignment/filtering/peak-calling parameters and cluster settings
- Executable pipeline definition via the 'Steps_to_execute_pipe' line of plain-word module blocks

## How to apply

Copy the original Targets.txt template shipped with the NEAT package into the newly created project directory. Fill the upper portion with fixed cluster-specific parameters (queuing priority, node allocation, number of CPUs, wall time, virtual paths to reference genomes) since NEAT runs on the institution's own cluster. In the 'SAMPLES INFO' section, list the compressed fastq names under 'OriFileName', user-defined names under 'FileName', replicate and sample-to-input relationships, and the H3K4me3 antibody specificities of the test dataset; for paired-end runs, name reverse-read files as the forward name followed by '_R2' below the 'PE corresponding samples' mark. Set the mapping algorithm (bwa or bowtie), filtering parameters in AdvancedSettings.txt, and the peak caller (MACS or SPP), setting all ChIP-RX parameters to '-' when no spike-in control is used. Populate 'Steps_to_execute_pipe' with exact plain-word blocks (e.g. 'unzip', 'QC', 'chiprx', 'map', 'filter', 'peakcalling', 'cleanfiles', 'granges'), renaming unwanted blocks (e.g. 'chiprs_NO') because NEAT uses exact word matching.

## Related tools

- **NEAT** (Framework whose four modules (create project, build pipeline, transfer, analyze) consume Targets.txt; the file controls which automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling) run) — https://github.com/pschorderet/NEAT
- **systemPipeR** (R/Bioconductor package invoked by NEAT's 'QC' module, one of the pipeline blocks selected in Targets.txt) — https://github.com/tgirke/systemPipeR

## Evaluation signals

- Targets.txt exists in the project directory and the 'SAMPLES INFO' section maps every OriFileName to a FileName, a replicate/input relationship and an antibody
- All ChIP-RX fields are '-' if and only if no spike-in control was used
- Every entry on 'Steps_to_execute_pipe' exactly matches a NEAT module name (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) or is explicitly renamed with a suffix such as '_NO' to be skipped
- Paired-end reverse-read entries follow the '<forward_name>_R2' convention under 'PE corresponding samples'
- Cluster-specific fields (queuing priority, node allocation, CPUs, wall time, virtual genome paths) are filled and consistent with the institution's qsub/bsub setup

## Limitations

- NEAT matches pipeline block names by exact word matching, so typos or renames silently disable modules
- Operating system limited to Mac OSx; no Linux/Windows support stated
- NEAT version number is not stated and no changelog is provided in the repository
- Requires access to the institution's cluster (qsub or bsub) for the cluster-parameter fields to be meaningful

## Evidence

- [abstract] Centralized metadata file carries all project configuration: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [intro] Modules selectable as plain-word pipeline blocks: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] Test dataset used to validate sample annotation is H3K4me3 ChIPseq from mouse embryonic stem cells: "The test datasets, which are either H3K4me3 ChIPseq data or RNAseq data from mouse embryonic stem cells, have been truncated"
- [readme] Tutorials for novice users: "To start using NEAT, please follow the tutorials found in the 'Vignette' folder"
