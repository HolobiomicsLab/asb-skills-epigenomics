---
name: chia-pet-pipeline-execution
description: Use when raw paired-end FASTQ reads from ChIA-PET (short 2×21 bp or long
  2×150 bp), HiChIP, or PLAC-seq libraries need automated processing into peaks, loops,
  contact domains, and 2D contact map visualization files.
when_to_use_negative:
- Input is already processed peak/loop BED files or alignment output rather than raw
  FASTQ reads
- The assay is plain Hi-C without a protein-factor linker/junction structure — ChIA-PIPE
  is built for ChIA-PET-family (ChIA-PET/HiChIP/PLAC-seq) data
- Only a fine-grained downstream sub-step is needed (e.g. MAPQ ≥30 dedup via CPU dedup
  module) rather than full end-to-end pipeline execution
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0607
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline performing linker filtering, read mapping, peak calling,
    loop calling, quality control, and .hic/.cool generation
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: Juicebox
  role: Web-based 2D contact map viewer consuming ChIA-PIPE-generated .hic files
- name: HiGlass
  role: Web-based 2D contact map viewer consuming ChIA-PIPE-generated .cool files
- name: BASIC Browser
  role: Dockerized loop browser for interactive, high-resolution visualization of
    ChIA-PIPE peaks and loops
- name: CPU (ChIA-PET Utilities)
  role: Collection of modularized executables (e.g. dedup module retaining unique
    MAPQ ≥30 nonredundant PETs) integrated into ChIA-PIPE
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:92e2236cddea536ccc585d41e2a918c8ad3f1c4e264acd66385d332ca031e421
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BASIC Browser
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0607
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chia-pet-pipeline-execution

## Summary

Execute the fully automated ChIA-PIPE pipeline (v1.0) to process ChIA-PET, HiChIP, or PLAC-seq FASTQ data end-to-end — linker filtering, read mapping, peak calling, loop calling, and automated quality control — producing browser-ready .hic and .cool contact maps plus peak and loop call outputs.

## When to use

Use when raw paired-end FASTQ reads from ChIA-PET (short 2×21 bp or long 2×150 bp), HiChIP, or PLAC-seq libraries need automated processing into peaks, loops, contact domains, and 2D contact map visualization files.

## When NOT to use

- Input is already processed peak/loop BED files or alignment output rather than raw FASTQ reads
- The assay is plain Hi-C without a protein-factor linker/junction structure — ChIA-PIPE is built for ChIA-PET-family (ChIA-PET/HiChIP/PLAC-seq) data
- Only a fine-grained downstream sub-step is needed (e.g. MAPQ ≥30 dedup via CPU dedup module) rather than full end-to-end pipeline execution

## Inputs

- Paired-end FASTQ files (e.g. LDK0004-ds_*.fastq.gz) from ChIA-PET, HiChIP, or PLAC-seq libraries
- ChIA-PIPE shell configuration file (e.g. chia_pipe-master/example_config_file.sh)
- Optionally, an input-control sample for peak calling

## Outputs

- Peak call outputs
- Loop call outputs (loops with PET count ≥3 and peak support)
- Chromatin contact domains (CCDs, ≥25 kb regions with continuous loop coverage)
- Browser-ready .hic and .cool contact map files for Juicebox and HiGlass
- Automated quality control assessment per dataset

## How to apply

Initialize an outer working directory and clone ChIA-PIPE from github.com/TheJacksonLaboratory/ChIA-PIPE, then install all pipeline dependencies locally into a dedicated dependency directory with local_install_chia_pipe_dependencies.sh -i ${dep_dir}. Copy test or production FASTQ files into a fastq/ subdirectory (e.g. LDK0004-ds_*.fastq.gz from the Dropbox share). Launch the full workflow with bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh, or on an HPC cluster submit chia_pipe-master/0.chia_pipe_hpc.pbs via qsub passing the config through --conf. The pipeline handles different junction sequences between ChIA-PET and HiChIP/PLAC-seq via pseudolinker handling, performs linker filtering, read mapping, peak calling (with input-control support and stringent defaults to reduce false positives), loop calling, and automated quality control. Finally generate .hic and .cool files so contact maps render immediately in Juicebox and HiGlass, and inspect loops interactively in the BASIC Browser.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline performing linker filtering, read mapping, peak calling, loop calling, quality control, and .hic/.cool generation) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **Juicebox** (Web-based 2D contact map viewer consuming ChIA-PIPE-generated .hic files)
- **HiGlass** (Web-based 2D contact map viewer consuming ChIA-PIPE-generated .cool files)
- **BASIC Browser** (Dockerized loop browser for interactive, high-resolution visualization of ChIA-PIPE peaks and loops)
- **CPU (ChIA-PET Utilities)** (Collection of modularized executables (e.g. dedup module retaining unique MAPQ ≥30 nonredundant PETs) integrated into ChIA-PIPE) — https://github.com/cheehongsg/CPU

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- The pipeline run completes on the test dataset (LDK0004-ds FASTQs) without errors through linker filtering, mapping, peak calling, and loop calling stages
- Browser-ready .hic and .cool contact map files are produced and render immediately in Juicebox and HiGlass
- Peak and loop call outputs are present with expected schema (loops with PET count ≥3 and peak support, each loop assigned a unique ID)
- 2D heat maps across ChIA-PET, HiChIP, and PLAC-seq outputs show comparable chromatin interactions in the BASIC Browser
- Quality control assessment is generated automatically for the dataset with no missing QC reports

## Limitations

- Acceptance for the section is qualitative presence of complete, viewable outputs — no numeric threshold is reported in this context
- Only PET reads with detected linker and both ends having genomic tags are used for loop detection, so PETs failing linker categorization are excluded
- PETs with genomic span ≤8 kb are classified as self-ligation rather than interligation, capping the span of usable interaction PETs for loop detection
- Peak calling without an input-control sample on very large datasets results in an abundance of false-positive peaks
- The reference genome version used for alignment is not specified in the provided section

## Evidence
