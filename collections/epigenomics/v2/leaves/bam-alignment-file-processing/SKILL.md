---
name: bam-alignment-file-processing
description: Use when you have mapped ChIPseq/RNAseq reads in .bam files from a NEAT
  project and need to filter, reorganize, or route them into downstream modules such
  as peakcalling or GRanges.
when_to_use_negative:
- Input is raw FASTQ data not yet aligned — the 'map' module (bwa or bowtie) must
  run first to produce .bam files
- Analysis requires Linux or Windows support — NEAT is documented as running on Mac
  OSx only
- You need a tool with a changelog or explicit version control over parameter defaults
  — no changelog is available for the repository
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0080
tools:
- name: NEAT
  role: Framework hosting the 'filter', 'cleanfiles', and 'peakcalling' pipeline modules
    that operate on .bam files
  repo: https://github.com/pschorderet/NEAT
- name: SAMtools
  role: Referenced as a tool for manipulating big data files such as BAM alignments
- name: MACS
  role: Peak caller dispatched via PeakCaller_MACS.R on the filtered .bam alignment
- name: SPP
  role: Peak caller dispatched via PeakCaller_SPP.R on the filtered .bam alignment
- name: systemPipeR
  role: R/Bioconductor package used by the QC module; its repo appears in the NEAT
    project context
  repo: https://github.com/tgirke/systemPipeR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:7483efce5c5a2b7fcbfb6233a6dc98d6cd94c9c73a05c62396c619f640968160
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
  - systemPipeR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# bam-alignment-file-processing

## Summary

Reorganize, filter, and prepare BAM alignment files (.bam/.bai) within a NEAT ChIPseq/RNAseq pipeline so they are ready for downstream peak calling and analysis modules, using the centralized Targets.txt/AdvancedSettings.txt configuration.

## When to use

Use when you have mapped ChIPseq/RNAseq reads in .bam files from a NEAT project and need to filter, reorganize, or route them into downstream modules such as peakcalling or GRanges.

## When NOT to use

- Input is raw FASTQ data not yet aligned — the 'map' module (bwa or bowtie) must run first to produce .bam files
- Analysis requires Linux or Windows support — NEAT is documented as running on Mac OSx only
- You need a tool with a changelog or explicit version control over parameter defaults — no changelog is available for the repository

## Inputs

- .bam alignment files (from the 'map' module using bwa or bowtie)
- Targets.txt centralized project configuration (sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters)
- AdvancedSettings.txt with filtering parameters (duplicate read handling, minimum and maximum fragment sizes)
- .bai index files

## Outputs

- Reorganized and stored .bam/.bai files (via 'cleanfiles')
- Filtered ChIPseq alignment (.bam) file
- peaks.bed with called peaks in the project folder (from the peakcalling module)

## How to apply

Load the NEAT project configuration from the centralized Targets.txt file (sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters) and place the peakcalling algorithm under 'Steps_to_execute_pipe' via exact word matching. After the 'map' module maps reads with bwa or bowtie, use the 'filter' module with parameters specified in AdvancedSettings.txt, including how to manage duplicate reads and minimum/maximum fragment sizes. Use the 'cleanfiles' module to reorganize and store the resulting .bam and .bai files before downstream analysis. The filtered alignment is then consumed by the peakcalling module (MACS via PeakCaller_MACS.R or SPP via PeakCaller_SPP.R), which writes called peaks to peaks.bed in the project folder.

## Related tools

- **NEAT** (Framework hosting the 'filter', 'cleanfiles', and 'peakcalling' pipeline modules that operate on .bam files) — https://github.com/pschorderet/NEAT
- **SAMtools** (Referenced as a tool for manipulating big data files such as BAM alignments)
- **MACS** (Peak caller dispatched via PeakCaller_MACS.R on the filtered .bam alignment)
- **SPP** (Peak caller dispatched via PeakCaller_SPP.R on the filtered .bam alignment)
- **systemPipeR** (R/Bioconductor package used by the QC module; its repo appears in the NEAT project context) — https://github.com/tgirke/systemPipeR

## Examples

```
Rscript PeakCaller_MACS.R
```

## Evaluation signals

- peakcalling script runs to completion and peaks.bed exists in the expected project output location with non-empty peak records
- cleanfiles module has reorganized and stored both .bam and .bai files in the correct project subfolder before downstream analysis
- the algorithm selected under 'Steps_to_execute_pipe' in Targets.txt is matched by exact word match (e.g. 'peakcalling') and dispatched to the correct script
- filtering parameters (duplicate read management, fragment size min/max) applied match those declared in AdvancedSettings.txt

## Limitations

- NEAT is documented for Mac OSx only; no Linux/Windows support is stated
- No numeric peak-quality threshold is reported for validating peaks.bed contents
- NEAT is still under development and no changelog is provided for the repository
- The paper states no concrete quantitative results for the pipeline — outputs (Fig. 3) serve as the tangible results

## Evidence

- [methods] The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments: "The ‘filter’ module allows the user to specify filtering parameters (AdvancedSettings.txt) including how to manage duplicate reads, minimum and maximum size of fragments"
- [methods] the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis: "the ‘cleanfiles’ module helps reorganize and store the different .bam and .bai files before proceeding to downstream analysis"
- [intro] NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc): "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [abstract] NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters"
- [discussion] Operating system: Mac OSx: "Operating system: Mac OSx"
