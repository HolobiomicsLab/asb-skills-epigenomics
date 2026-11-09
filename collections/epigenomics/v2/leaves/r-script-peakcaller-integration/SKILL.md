---
name: r-script-peakcaller-integration
description: Use when a NEAT ChIPseq project needs an automated 'peakcalling' module
  run after filtering, with the algorithm and parameters defined in the centralized
  Targets.txt configuration file.
when_to_use_negative:
- The project is an RNAseq workflow — peak calling applies to ChIPseq enrichment peaks,
  not RNAseq differential expression analysis
- The alignment (.bam) files have not yet passed the 'filter' and 'cleanfiles' modules,
  so the input required by the peak caller does not exist
- A custom peak caller outside MACS or SPP is required — NEAT's 'peakcalling' module
  dispatches only the algorithm selected in Targets.txt
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: NEAT
  role: Framework hosting the automated 'peakcalling' module; algorithm selection
    and parameters are configured via the centralized Targets.txt file
  repo: https://github.com/pschorderet/NEAT
- name: MACS
  role: Peak calling algorithm dispatched via the PeakCaller_MACS.R script when selected
    in Targets.txt
- name: SPP
  role: Peak calling algorithm dispatched via the PeakCaller_SPP.R script when selected
    in Targets.txt
- name: R
  role: Runtime for the PeakCaller_MACS.R and PeakCaller_SPP.R scripts that execute
    the peak calling step
- name: SAMtools
  role: Alignment file manipulation in the pipeline preceding peak calling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:06cbae4c074d7e91f50ac2b17cdf2dbf90d85f3d358ffaccf36146f626dbc8dd
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
yes_rate: 1.0
n_claims: 12
---

# r-script-peakcaller-integration

## Summary

Integrates ChIPseq peak calling into an automated NEAT pipeline by dispatching MACS or SPP via dedicated R scripts (PeakCaller_MACS.R / PeakCaller_SPP.R), with algorithm selection and parameters configured centrally in Targets.txt.

## When to use

Use when a NEAT ChIPseq project needs an automated 'peakcalling' module run after filtering, with the algorithm and parameters defined in the centralized Targets.txt configuration file.

## When NOT to use

- The project is an RNAseq workflow — peak calling applies to ChIPseq enrichment peaks, not RNAseq differential expression analysis
- The alignment (.bam) files have not yet passed the 'filter' and 'cleanfiles' modules, so the input required by the peak caller does not exist
- A custom peak caller outside MACS or SPP is required — NEAT's 'peakcalling' module dispatches only the algorithm selected in Targets.txt

## Inputs

- Filtered ChIPseq alignment file (.bam) reorganized by the 'cleanfiles' module
- NEAT Targets.txt project configuration specifying the peak calling algorithm under 'Steps_to_execute_pipe'
- Peak calling parameters defined in the centralized Targets.txt file

## Outputs

- peaks.bed file of called ChIPseq enrichment peaks in the project folder, for downstream NEAT EDA modules

## How to apply

Load the NEAT project configuration from Targets.txt and select the peak calling algorithm under 'Steps_to_execute_pipe' using exact word matching (MACS or SPP). Read the filtered ChIPseq alignment (.bam) file reorganized by the 'cleanfiles' module, then dispatch to PeakCaller_MACS.R or PeakCaller_SPP.R. Run the selected peak caller over the alignment using the peak calling parameters defined in the centralized Targets.txt file. Write the called peaks to peaks.bed in the project folder so downstream NEAT EDA modules (e.g. wig/bigwig, GRanges, metagenomic analysis) can consume them. Validate that the script runs to completion and peaks.bed exists in the expected output location with non-empty peak records.

## Related tools

- **NEAT** (Framework hosting the automated 'peakcalling' module; algorithm selection and parameters are configured via the centralized Targets.txt file) — https://github.com/pschorderet/NEAT
- **MACS** (Peak calling algorithm dispatched via the PeakCaller_MACS.R script when selected in Targets.txt)
- **SPP** (Peak calling algorithm dispatched via the PeakCaller_SPP.R script when selected in Targets.txt)
- **R** (Runtime for the PeakCaller_MACS.R and PeakCaller_SPP.R scripts that execute the peak calling step)
- **SAMtools** (Alignment file manipulation in the pipeline preceding peak calling)

## Examples

```
Rscript PeakCaller_MACS.R  # dispatched by the NEAT 'peakcalling' module using the algorithm and parameters set in Targets.txt
```

## Evaluation signals

- The peakcalling script runs to completion without errors
- peaks.bed exists in the expected project output location
- peaks.bed contains non-empty peak records (the paper reports no numeric peak-quality threshold)
- The algorithm selected in 'Steps_to_execute_pipe' in Targets.txt matches exactly one of the supported dispatch targets (MACS or SPP) via word matching
- Downstream NEAT EDA modules can consume peaks.bed from the project folder

## Limitations

- The paper reports no numeric peak-quality threshold, so validation is limited to completion and non-empty output rather than peak-quality metrics
- Only the algorithms configured in Targets.txt are dispatched; unsupported algorithm names will not match during exact word matching
- NEAT is still under development per the README; bugs should be reported to the authors
- No version number for NEAT is stated in the article's discussion section

## Evidence

- [methods] The ‘peakcalling’ module specifies the algorithm used to call peaks.: "The ‘peakcalling’ module specifies the algorithm used to call peaks."
- [abstract] NEAT projects are built around a unique centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] README module list including Peakcalling after Filter: "(v)		Filter

    (vi)	Peakcalling"
