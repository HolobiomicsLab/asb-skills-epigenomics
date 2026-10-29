---
name: hpc-cluster-parallel-scheduling
description: Use when running MEDIPIPE's Snakemake Snakefile over many cfMeDIP-seq
  FASTQ samples on a cluster (e.g. SLURM) or stand-alone machine, and per-sample Conda
  environments must be auto-deployed for parallel execution.
when_to_use_negative:
- The analysis is a single small sample set on a local machine where cluster scheduling
  adds overhead without benefit — MEDIPIPE also supports stand-alone machines
- The data are not (cf)MeDIP-seq FASTQ files; the workflow and its tools (MEDIPS,
  QSEA, MEDStrand) are designed for methylated DNA immunoprecipitation data
- Only post-processing of pre-computed methylome data is needed, in which case the
  prior pughlab/cfMeDIP-seq-analysis-pipeline may be more appropriate
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow engine that automatically deploys execution environments and parallelizes
    independent per-sample rules across cluster engines
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Deployment backend for Snakemake containerized execution environments; also
    used to create MEDIPIPE's core environment
  repo: https://github.com/pughlab/MEDIPIPE
- name: Mamba
  role: Recommended fast Conda reimplementation for installing the core MEDIPIPE environment
    from conda_env.yaml
  repo: https://github.com/mamba-org/mamba
- name: SLURM
  role: Cluster engine via which MEDIPIPE jobs are submitted (workflow/sbatch_Snakefile_template.sh
    pattern)
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:a620e7e5014e9f96fed330c3f8a0f272b1586b8f7073badd7f144a15525be088
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - BWA-MEM
  - Conda
  - MEDIPS
  - MEDStrand
  - Mamba
  - MultiQC
  - Picard
  - QSEA
  - SAMtools
  - SLURM
  - Snakemake
  - Trim Galore
  - UMI-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# hpc-cluster-parallel-scheduling

## Summary

Deploy and execute the MEDIPIPE (cf)MeDIP-seq QC/analysis workflow on an HPC cluster using Snakemake with automatically deployed Conda/Mamba execution environments, so that independent per-sample steps (trimming, alignment, methylation quantification) run in parallel across compute nodes.

## When to use

Use when running MEDIPIPE's Snakemake Snakefile over many cfMeDIP-seq FASTQ samples on a cluster (e.g. SLURM) or stand-alone machine, and per-sample Conda environments must be auto-deployed for parallel execution.

## When NOT to use

- The analysis is a single small sample set on a local machine where cluster scheduling adds overhead without benefit — MEDIPIPE also supports stand-alone machines
- The data are not (cf)MeDIP-seq FASTQ files; the workflow and its tools (MEDIPS, QSEA, MEDStrand) are designed for methylated DNA immunoprecipitation data
- Only post-processing of pre-computed methylome data is needed, in which case the prior pughlab/cfMeDIP-seq-analysis-pipeline may be more appropriate

## Inputs

- Raw cfMeDIP-seq FASTQ files (single-end or paired-end, with or without spike-in/UMI sequences)
- Sample sequencing data table
- YAML input configuration file (working environment paths, genome references, sample information, experimental settings)
- MEDIPIPE repository conda_env.yaml for the core environment

## Outputs

- Aggregated methylation quantification matrices as Tabix-indexed TAB-delimited bin-sample TXT files (original and filtered)
- MultiQC summarized interactive HTML QC report
- Per-sample QC outputs (FastQC, Picard insert-size/fragmentation metrics, MEDIPS saturation/coverage/enrichment QC)
- Snakemake and SLURM job logs from parallel rule execution

## How to apply

Clone the MEDIPIPE repository (https://github.com/pughlab/MEDIPIPE) and build its core environment with a Conda-based Python3 distribution using Mamba (recommended), e.g. creating the environment from conda_env.yaml; Snakemake then automatically deploys per-tool execution environments, with software versions pinned in the YAML files. Write the single YAML input configuration file specifying working environment paths, genome references (BWA index and ENCODE annotated regions), sample information, and experimental settings (single-end/paired-end, spike-in/UMI usage). Invoke the Snakefile with --use-conda and a --configfile, allocating cores explicitly for stand-alone execution. For cluster execution, submit via SLURM using the provided sbatch Snakefile template so that independent per-sample rules (Trim Galore, FastQC, UMI-tools, BWA-MEM, SAMtools, Picard, MEDIPS/QSEA/MEDStrand quantification, MultiQC aggregation) schedule in parallel. The rationale is reproducibility and portability: Snakemake containerized execution environments are automatically deployed via Conda, so identical software versions are reproduced regardless of cluster engine.

## Related tools

- **Snakemake** (Workflow engine that automatically deploys execution environments and parallelizes independent per-sample rules across cluster engines) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Deployment backend for Snakemake containerized execution environments; also used to create MEDIPIPE's core environment) — https://github.com/pughlab/MEDIPIPE
- **Mamba** (Recommended fast Conda reimplementation for installing the core MEDIPIPE environment from conda_env.yaml) — https://github.com/mamba-org/mamba
- **SLURM** (Cluster engine via which MEDIPIPE jobs are submitted (workflow/sbatch_Snakefile_template.sh pattern)) — https://github.com/pughlab/MEDIPIPE

## Examples

```
mamba env create --file conda_env.yaml; snakemake --use-conda --cores 16 --configfile config/config_template.yaml
```

## Evaluation signals

- All Conda environments resolve and are auto-deployed by Snakemake without version conflicts (software versions pinned in YAML files, e.g. Trim Galore v0.6.7, BWA-MEM v0.7.17, MEDIPS v1.46.0)
- Per-sample rules execute in parallel: job scheduling under --cores or SLURM sbatch processes all 163 samples concurrently rather than serially
- Final outputs exist for every sample: aggregated methylation matrices (before and after filtering, Tabix-indexed) and the MultiQC interactive HTML QC report
- Filtered aggregation excludes sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions as specified in the configuration
- Re-running the workflow on the same configuration reproduces the group/batch-level QC view demonstrated in Supplementary Fig. S2

## Limitations

- A Conda-based Python3 distribution is required before installation; Mamba installation is recommended but additional setup
- Genome references (BWA index and annotated regions) must be pre-downloaded via the provided shell scripts before execution
- The software versions bundled in the YAML environment files are fixed (e.g. Snakemake-deployed tool versions); users must edit YAML files to change pinned versions

## Evidence

- [readme] Portability via Snakemake auto-deployed environments: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments."
- [intro] Cluster-engine portability: "It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines"
- [readme] Version pinning in YAML files: "Users can also specify the version of softwares to be installed in YAML files."
- [intro] Conda prerequisite: "Make sure that you have a Conda-based Python3 distribution"
