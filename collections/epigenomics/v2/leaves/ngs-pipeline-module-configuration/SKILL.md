---
name: ngs-pipeline-module-configuration
description: Use when building or re-running a NEAT NGS project and you must select
  which pipeline modules (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles,
  granges) execute and with which user parameters.
when_to_use_negative:
- The analysis target is RNAseq differential expression via NEAT's EDA module rather
  than pipeline module selection and execution order
- The pipeline framework is not NEAT — exact-word Targets.txt block matching ('Steps_to_execute_pipe')
  does not transfer to other workflow engines
- Input data are already processed alignments or peak calls, so no upstream module
  configuration (unzip, QC, map, filter, peakcalling) is needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3169
tools:
- name: NEAT
  role: Framework whose modules are configured via the centralized Targets.txt file;
    exact-word matching dispatches each pipeline block
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Backend for the 'filter' module whose parameters (duplicate handling, fragment
    size bounds) are set in AdvancedSettings.txt
- name: MACS
  role: Peak caller selectable in Targets.txt, dispatched via PeakCaller_MACS.R when
    'peakcalling' is listed
- name: SPP
  role: Alternative peak caller selectable in Targets.txt, dispatched via PeakCaller_SPP.R
- name: systemPipeR
  role: R package underlying the 'QC' module configured in the pipeline block list
  repo: https://github.com/tgirke/systemPipeR
- name: bwa
  role: Mapping algorithm selectable alongside bowtie in Targets.txt for the 'map'
    module
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_005
  - task_007
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:df2e13bbe93827e9d27654ca9281d337d00539a40f0e900cf2e530e47e240237
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - Bioconductor
  - MACS
  - NEAT
  - R
  - SAMtools
  - SPP
  - bwa
  - systemPipeR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# ngs-pipeline-module-configuration

## Summary

Configure NEAT's automated ChIPseq/RNAseq pipeline modules by populating the centralized Targets.txt project file with sample metadata, algorithm choices (bwa/bowtie, MACS/SPP), cluster parameters, and an exact-word 'Steps_to_execute_pipe' block list.

## When to use

Use when building or re-running a NEAT NGS project and you must select which pipeline modules (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges) execute and with which user parameters.

## When NOT to use

- The analysis target is RNAseq differential expression via NEAT's EDA module rather than pipeline module selection and execution order
- The pipeline framework is not NEAT — exact-word Targets.txt block matching ('Steps_to_execute_pipe') does not transfer to other workflow engines
- Input data are already processed alignments or peak calls, so no upstream module configuration (unzip, QC, map, filter, peakcalling) is needed

## Inputs

- Compressed FASTQ files (fastq) registered in Targets.txt under 'OriFileName'
- Targets.txt template shipped with the NEAT package
- AdvancedSettings.txt for filtering parameters (duplicate reads, min/max fragment size)
- Cluster-specific parameters (queuing priority, CPUs, wall time, reference genome paths)

## Outputs

- Completed Targets.txt with sample info, algorithm and peak-caller selections, and exact-word module list
- Executed pipeline producing QC pdf reports, filtered .bam/.bai, peaks.bed, and .wig/.bigwig/GRanges track files

## How to apply

Copy the Targets.txt template from the NEAT package into the new project directory and fill its upper portion with cluster-specific parameters (queuing priority, node allocation, number of CPUs, wall time, virtual paths to reference genomes) since NEAT runs on the institution's own cluster. Populate the 'SAMPLES INFO' section with compressed fastq names under 'OriFileName', user-defined names under 'FileName', replicate and sample-to-input relationships, and antibody specificities (e.g. H3K4me3 for the mouse embryonic stem cell test dataset); for paired-end runs, name reverse-read files as the forward name plus '_R2'. Choose the mapping algorithm (bwa or bowtie), set filtering parameters (duplicate-read policy, min/max fragment sizes) in AdvancedSettings.txt, and pick the peak caller (MACS or SPP); if no spike-in control is used, set all ChIP-RX parameters to '-'. On the 'Steps_to_execute_pipe' line list plain-word building blocks such as 'unzip', 'QC', 'chiprx', 'map', 'filter', 'peakcalling', 'cleanfiles', 'granges', deleting or renaming (e.g. 'chiprs_NO') blocks that should not run, because NEAT dispatches modules by exact word matching.

## Related tools

- **NEAT** (Framework whose modules are configured via the centralized Targets.txt file; exact-word matching dispatches each pipeline block) — https://github.com/pschorderet/NEAT
- **SAMtools** (Backend for the 'filter' module whose parameters (duplicate handling, fragment size bounds) are set in AdvancedSettings.txt)
- **MACS** (Peak caller selectable in Targets.txt, dispatched via PeakCaller_MACS.R when 'peakcalling' is listed)
- **SPP** (Alternative peak caller selectable in Targets.txt, dispatched via PeakCaller_SPP.R)
- **systemPipeR** (R package underlying the 'QC' module configured in the pipeline block list) — https://github.com/tgirke/systemPipeR
- **bwa** (Mapping algorithm selectable alongside bowtie in Targets.txt for the 'map' module)

## Examples

```
Rscript NEAT_create_project.R && # fill Targets.txt: SAMPLES INFO (OriFileName/FileName), map=bwa, peakcalling=MACS, Steps_to_execute_pipe: unzip QC map filter peakcalling cleanfiles granges
```

## Evaluation signals

- Targets.txt exists in the project directory and contains all required fields: sample names, replicate/sample-to-input relationships, antibody, alignment/filtering/peak-calling parameters, and cluster paths
- Every entry on 'Steps_to_execute_pipe' matches one of NEAT's plain-word module names exactly (unzip, QC, chiprx, map, filter, peakcalling, cleanfiles, granges); typos silently skip modules
- Disallowed blocks are removed or renamed (e.g. 'chiprs_NO') rather than left active
- Downstream artifacts appear in expected locations: peaks.bed is non-empty after 'peakcalling', .wig files land in the reported ~4–6 Mb range, .bam is accompanied by .bai
- ChIP-RX parameters are set to '-' consistently when no spike-in control is used

## Limitations

- NEAT uses exact word matching on 'Steps_to_execute_pipe', so any misspelled or renamed block name causes that module to be silently skipped
- NEAT is limited to Mac OSx per the paper's discussion section; no Linux/Windows support is stated
- No NEAT version number or repository changelog is provided, so reproducibility across versions is not documented
- NEAT is still under development; the README asks users to report bugs

## Evidence

- [other] Set the mapping algorithm (bwa or bowtie), filtering parameters in AdvancedSettings.txt and the peak caller (MACS or SPP); if no spike-in control is used, set all ChIP-RX parameters to '-'.: [evidence span withheld — anchor status: not_found]
- [other] Populate the 'Steps_to_execute_pipe' line with exact plain-word building blocks (e.g. 'unzip', 'QC', 'chiprx', 'map', 'filter', 'peakcalling', 'cleanfiles', 'granges'), deleting or renaming (e.g. 'chiprs_NO') any block that should not run: [evidence span withheld — anchor status: not_found]
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [abstract] Storage space, maximum number of job submissions, wall time and cluster-specific parameters can be customized as NEAT is run on the institution's cluster: "Storage space, maximum number of job submissions, wall time and cluster-specific parameters can be customized as NEAT is run on the institution's cluster"
