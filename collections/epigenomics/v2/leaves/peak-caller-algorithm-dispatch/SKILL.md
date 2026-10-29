---
name: peak-caller-algorithm-dispatch
description: Use when a ChIPseq pipeline must call enrichment peaks from filtered
  .bam alignments and the peak calling algorithm and its parameters are configured
  in a centralized NEAT Targets.txt file.
when_to_use_negative:
- The input is raw FASTQ data still awaiting QC, ChIP-rx normalization, mapping, or
  filtering — peak calling sits after filter in the NEAT module sequence
- The task is RNAseq analysis (e.g. differential gene expression) rather than ChIPseq
  enrichment peak identification
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: NEAT
  role: Pipeline framework hosting the peakcalling module and centralizing configuration
    in Targets.txt
  repo: https://github.com/pschorderet/NEAT
- name: MACS
  role: Peak calling algorithm dispatched via the PeakCaller_MACS.R script
- name: SPP
  role: Peak calling algorithm dispatched via the PeakCaller_SPP.R script
- name: R
  role: Runtime for the PeakCaller_MACS.R and PeakCaller_SPP.R dispatch scripts
- name: SAMtools
  role: Alignment file handling within the ChIPseq pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:c953b25652a02c854a7e5e179ae4a98f6af6aac141d436f87b99c5403ba35377
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
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# peak-caller-algorithm-dispatch

## Summary

Dispatches ChIPseq peak calling within a NEAT pipeline by reading the algorithm selection from the centralized Targets.txt file and routing the filtered .bam alignment to either MACS (PeakCaller_MACS.R) or SPP (PeakCaller_SPP.R), writing results to peaks.bed.

## When to use

Use when a ChIPseq pipeline must call enrichment peaks from filtered .bam alignments and the peak calling algorithm and its parameters are configured in a centralized NEAT Targets.txt file.

## When NOT to use

- The input is raw FASTQ data still awaiting QC, ChIP-rx normalization, mapping, or filtering — peak calling sits after filter in the NEAT module sequence
- The task is RNAseq analysis (e.g. differential gene expression) rather than ChIPseq enrichment peak identification

## Inputs

- filtered ChIPseq alignment file (.bam) reorganized by the cleanfiles module
- NEAT project Targets.txt configuration specifying the peak calling algorithm and parameters

## Outputs

- peaks.bed file in the project folder containing called ChIPseq enrichment peaks

## How to apply

Load the NEAT project configuration from Targets.txt and identify the selected peak caller under 'Steps_to_execute_pipe' via exact word matching. Read the filtered ChIPseq .bam alignment reorganized by the 'cleanfiles' module, then dispatch to MACS via PeakCaller_MACS.R or SPP via PeakCaller_SPP.R depending on the configuration. Run the caller using peak calling parameters defined centrally in Targets.txt, and write the resulting peaks to peaks.bed in the project folder for downstream NEAT EDA modules. Validate by confirming the script runs to completion and peaks.bed exists with non-empty peak records.

## Related tools

- **NEAT** (Pipeline framework hosting the peakcalling module and centralizing configuration in Targets.txt) — https://github.com/pschorderet/NEAT
- **MACS** (Peak calling algorithm dispatched via the PeakCaller_MACS.R script)
- **SPP** (Peak calling algorithm dispatched via the PeakCaller_SPP.R script)
- **R** (Runtime for the PeakCaller_MACS.R and PeakCaller_SPP.R dispatch scripts)
- **SAMtools** (Alignment file handling within the ChIPseq pipeline)

## Examples

```
Rscript PeakCaller_MACS.R
```

## Evaluation signals

- The peakcalling script runs to completion without error
- peaks.bed exists in the expected NEAT project output location
- peaks.bed is non-empty, containing peak records
- The algorithm dispatched matches the exact word match in Targets.txt 'Steps_to_execute_pipe' (MACS or SPP)

## Limitations

- The paper reports no numeric peak-quality threshold applied to called peaks; validation relies on script completion and non-empty output
- Only MACS and SPP are the documented dispatch targets; no other peak callers are described
- Operating system limited to Mac OSx; no Linux/Windows support stated in the discussion
- NEAT is still under development according to the README

## Evidence

- [methods] The 'peakcalling' module specifies the algorithm used to call peaks.: [evidence span withheld — anchor status: not_found]
- [abstract] Centralized parameters file: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [intro] Automated module sequence: "NEAT offers several automated modules (unzip, rename, QC, chiprx, map, filter, peakcalling, creation of wig files, etc)"
- [readme] Peakcalling in README module list: "(v)		Filter

    (vi)	Peakcalling"
- [methods] cleanfiles module: [evidence span withheld — anchor status: not_found]
