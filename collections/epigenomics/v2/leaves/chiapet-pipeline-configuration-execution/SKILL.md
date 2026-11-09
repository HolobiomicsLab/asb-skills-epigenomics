---
name: chiapet-pipeline-configuration-execution
description: Use when raw ChIA-PET (or HiChIP/PLAC-seq) paired-end FASTQ reads need
  end-to-end processing into chromatin interaction loops and peaks, with automated
  QC assessment.
when_to_use_negative:
- Input is already processed into interaction/loop callsets and only downstream steps
  (allele-specific resolution, CCD calling, enhancer-promoter annotation) are needed
- Data type is incompatible with ChIA-PET/HiChIP/PLAC-seq junction structures that
  the pipeline's linker filtering and tag mapping expect
- A custom, manually controlled per-module analysis is required, since ChIA-PIPE is
  designed as a fully automated pipeline
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChIA-PIPE
  role: Fully automated pipeline orchestrating linker filtering, BWA mapping, peak
    calling, loop calling, QC, and visualization file generation
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables called within ChIA-PIPE (e.g., 'dedup' module retains
    unique MAPQ >= 30 nonredundant PETs)
  repo: https://github.com/cheehongsg/CPU
- name: Juicebox
  role: Renders the .hic contact maps produced by the pipeline
- name: HiGlass
  role: Renders the .cool contact maps produced by the pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:04c6503d7c759e3e91daca8fd70bd37fdda63bdfb3c9119fc6a84387c8aee6b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chiapet-pipeline-configuration-execution

## Summary

Configure and execute ChIA-PIPE, the fully automated ChIA-PET analysis pipeline, to run linker filtering, BWA read mapping, peak calling, and loop calling on ChIA-PET FASTQ data (e.g., GM12878 CTCF MiSeq reads), producing peaks, loops, and visualization files.

## When to use

Use when raw ChIA-PET (or HiChIP/PLAC-seq) paired-end FASTQ reads need end-to-end processing into chromatin interaction loops and peaks, with automated QC assessment.

## When NOT to use

- Input is already processed into interaction/loop callsets and only downstream steps (allele-specific resolution, CCD calling, enhancer-promoter annotation) are needed
- Data type is incompatible with ChIA-PET/HiChIP/PLAC-seq junction structures that the pipeline's linker filtering and tag mapping expect
- A custom, manually controlled per-module analysis is required, since ChIA-PIPE is designed as a fully automated pipeline

## Inputs

- ChIA-PET paired-end FASTQ reads (e.g., LDK0004-ds_*.fastq.gz CTCF MiSeq test data)
- ChIA-PIPE configuration shell script (e.g., example_config_file.sh)

## Outputs

- Called peak and loop callsets (loops with PET count >= 3 and peak support at both anchors)
- Nonredundant unique PETs (MAPQ >= 30) after CPU dedup
- .hic and .cool files for Juicebox/HiGlass 2D contact map visualization
- Per-dataset automated quality control assessment reports

## How to apply

Initialize a working directory, clone the TheJacksonLaboratory/ChIA-PIPE repository, and install bundled dependencies via local_install_chia_pipe_dependencies.sh into a dependency directory. Stage ChIA-PET reads (e.g., the LDK0004-ds_*.fastq.gz CTCF MiSeq test files) into a fastq/ directory, then launch the pipeline with 0.chia_pipe_shell.sh pointed at the example configuration file (or 0.chia_pipe_hpc.pbs via qsub on an HPC). The pipeline performs linker filtering, hybrid BWA-ALN/BWA-MEM mapping of the flanking genomic tags, retains unique alignments (MAPQ >= 30) and nonredundant PETs via the CPU 'dedup' module, calls peaks, and calls loops requiring PET count >= 3 with peak support at both anchors. It also generates .hic and .cool visualization files and automates quality control assessment for each dataset.

## Related tools

- **ChIA-PIPE** (Fully automated pipeline orchestrating linker filtering, BWA mapping, peak calling, loop calling, QC, and visualization file generation) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Modularized executables called within ChIA-PIPE (e.g., 'dedup' module retains unique MAPQ >= 30 nonredundant PETs)) — https://github.com/cheehongsg/CPU
- **Juicebox** (Renders the .hic contact maps produced by the pipeline)
- **HiGlass** (Renders the .cool contact maps produced by the pipeline)

## Examples

```
bash chia_pipe-master/0.chia_pipe_shell.sh -c chia_pipe-master/example_config_file.sh
```

## Evaluation signals

- Loop callset respects the thresholds: PET count >= 3 and peak support at both anchors
- Retained alignments have MAPQ >= 30 and PETs are nonredundant after the CPU 'dedup' step
- Pipeline completes all stages: linker filtering, read mapping, peak calling, loop calling, and QC assessment without manual intervention
- .hic and .cool visualization files are produced and render correctly in Juicebox/HiGlass
- Running on the LDK0004-ds test FASTQs reproduces the expected output structure of the example configuration

## Limitations

- The custom Python script used for E-P/P-P loop classification is not named, versioned, or hosted at a stated location, hindering reproducibility of that sub-step
- Reference genome version used for alignment is not specified in the article's discussion section
- No accession IDs are provided for the CTCF and RNAPII ChIA-PET sequencing data used in the allele-specific analysis; test data are distributed via a Dropbox link rather than an archive

## Evidence

- [discussion] The unique alignments [MAPQ (mapping quality) >= 30] and nonredundant PETs are retained by calling the CPU 'dedup' module.: [evidence span withheld — anchor status: not_found]
