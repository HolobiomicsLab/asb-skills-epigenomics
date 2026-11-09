---
name: chipseq-peak-calling
description: 'Use when running the peakcalling step of a NEAT ChIPseq project: a filtered
  alignment (.bam) exists and an algorithm (MACS or SPP) is selected in Targets.txt
  under ''Steps_to_execute_pipe''.'
when_to_use_negative:
- The project is an RNAseq pipeline — peak calling applies only to the NEAT ChIPseq
  (chiprx/map/filter) workflow
- The alignment has not yet passed the NEAT 'filter' module (duplicate-read and fragment-size
  filtering), since peakcalling consumes filtered .bam files
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: NEAT
  role: Pipeline framework that automates the peakcalling module and centralizes its
    parameters in Targets.txt
  repo: https://github.com/pschorderet/NEAT
- name: MACS
  role: Peak-calling algorithm dispatched via the PeakCaller_MACS.R script
- name: SPP
  role: Peak-calling algorithm dispatched via the PeakCaller_SPP.R script
- name: R
  role: Runtime for the PeakCaller_MACS.R and PeakCaller_SPP.R scripts
- name: SAMtools
  role: Alignment file handling for .bam/.bai inputs in the pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:fb2b804265e206249549591e14488e61569f9088de0d20f0990dbabd38762a57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - MACS
  - NEAT
  - R
  - SAMtools
  - SPP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chipseq-peak-calling

## Summary

Automated ChIPseq peak-calling module from the NEAT pipeline: dispatches MACS or SPP over filtered .bam alignments to produce a peaks.bed file, with all peak-calling parameters centralized in the project's Targets.txt file.

## When to use

Use when running the peakcalling step of a NEAT ChIPseq project: a filtered alignment (.bam) exists and an algorithm (MACS or SPP) is selected in Targets.txt under 'Steps_to_execute_pipe'.

## When NOT to use

- The project is an RNAseq pipeline — peak calling applies only to the NEAT ChIPseq (chiprx/map/filter) workflow
- The alignment has not yet passed the NEAT 'filter' module (duplicate-read and fragment-size filtering), since peakcalling consumes filtered .bam files

## Inputs

- Filtered ChIPseq alignment file (.bam), reorganized by the NEAT 'cleanfiles' module
- Targets.txt project configuration specifying 'peakcalling' in 'Steps_to_execute_pipe' and the peak-calling parameters

## Outputs

- peaks.bed in the project folder, consumed by downstream NEAT EDA modules

## How to apply

Load the NEAT project configuration from Targets.txt, which selects the peak-calling algorithm via exact word matching under 'Steps_to_execute_pipe' and holds the centralized peak-calling parameters. Read the filtered ChIPseq alignment (.bam) reorganized by the 'cleanfiles' module, then dispatch to the matching R script: PeakCaller_MACS.R for MACS or PeakCaller_SPP.R for SPP. The peak caller runs over the alignment to identify ChIPseq enrichment peaks and writes results to peaks.bed in the project folder, where they feed downstream NEAT EDA modules (e.g. GRanges objects and wig/bigwig generation). Positioning matters: peakcalling executes after filter in the automated module sequence (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files). Validate by confirming the script completes and peaks.bed exists in the expected project output location with non-empty peak records.

## Related tools

- **NEAT** (Pipeline framework that automates the peakcalling module and centralizes its parameters in Targets.txt) — https://github.com/pschorderet/NEAT
- **MACS** (Peak-calling algorithm dispatched via the PeakCaller_MACS.R script)
- **SPP** (Peak-calling algorithm dispatched via the PeakCaller_SPP.R script)
- **R** (Runtime for the PeakCaller_MACS.R and PeakCaller_SPP.R scripts)
- **SAMtools** (Alignment file handling for .bam/.bai inputs in the pipeline)

## Evaluation signals

- The peakcalling script runs to completion without error
- peaks.bed exists in the expected project output location
- peaks.bed is non-empty (contains peak records); the paper reports no numeric peak-quality threshold
- The algorithm name in Targets.txt ('Steps_to_execute_pipe') matches exactly one of the supported words and dispatched the corresponding R script (MACS or SPP)
- Downstream NEAT EDA modules (GRanges/wig creation) can read peaks.bed without missing-file errors

## Limitations

- The paper reports no numeric peak-quality threshold for validating peak calls; validation is presence/non-emptiness of peaks.bed
- Peak calling is limited to the two dispatched algorithms, MACS and SPP; no other callers are described
- Operating system support in the article is limited to Mac OSx
- NEAT is noted as still under development ('If you encounter any bug, please contact us')

## Evidence

- [methods] The ‘peakcalling’ module specifies the algorithm used to call peaks.: "The ‘peakcalling’ module specifies the algorithm used to call peaks."
- [abstract] Centralized parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [intro] Automated module sequence: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [methods] cleanfiles module: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
- [readme] Peakcalling in README module list: "NEAT has been developed as four main modules: [ 1 ] Create a new project [ 2 ] Build a pipeline (i) Unzip and rename (ii) QC (iii) ChIP-rx normalization (iv) Map (v) Filter (vi) Peakcalling"
