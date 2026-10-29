---
name: snakemake-conda-workflow-configuration
description: Use when running an end-to-end (cf)MeDIP-seq workflow from raw FASTQ
  files to QC, alignment, methylation quantification and aggregation, and you need
  portable Snakemake/Conda environment deployment.
when_to_use_negative:
- Input data is not (cf)MeDIP-seq — e.g. whole-genome bisulfite sequencing or methylation
  arrays — since MEDIPIPE is designed specifically for MeDIP-seq data types
- You only need to run a single tool interactively (e.g. one FastQC or BWA run) rather
  than a reproducible end-to-end multi-sample workflow
- No Conda-based Python3 distribution or cluster engine is available and the Snakemake
  --use-conda environment deployment cannot be used
edam_operation: http://edamontology.org/operation_3764
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: Snakemake
  role: Workflow engine that automatically deploys execution environments and orchestrates
    per-sample parallel steps
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Underlying package system via which Snakemake containerized execution environments
    are automatically deployed
- name: Mamba
  role: Recommended fast Conda reimplementation used to create the core MEDIPIPE environment
    from conda_env.yaml
  repo: https://github.com/mamba-org/mamba
- name: SLURM
  role: Cluster engine on which the pipeline can be executed via the sbatch_Snakefile_template.sh
    submission script
- name: MEDIPIPE
  role: The end-to-end (cf)MeDIP-seq QC and analysis pipeline being configured and
    executed
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:1c560b1b713987e81e5a5e0b9a4910fc61cbd6cef6620df1eaf83ffa3e451a3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - BWA-MEM
  - Conda
  - MEDIPIPE
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
  edam_operation: http://edamontology.org/operation_3764
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# snakemake-conda-workflow-configuration

## Summary

Configure and run MEDIPIPE, a Snakemake-based (cf)MeDIP-seq QC and analysis pipeline whose per-tool execution environments are automatically deployed via Conda, from a single YAML input configuration file covering FASTQ inputs, genome references, and experimental settings.

## When to use

Use when running an end-to-end (cf)MeDIP-seq workflow from raw FASTQ files to QC, alignment, methylation quantification and aggregation, and you need portable Snakemake/Conda environment deployment.

## When NOT to use

- Input data is not (cf)MeDIP-seq — e.g. whole-genome bisulfite sequencing or methylation arrays — since MEDIPIPE is designed specifically for MeDIP-seq data types
- You only need to run a single tool interactively (e.g. one FastQC or BWA run) rather than a reproducible end-to-end multi-sample workflow
- No Conda-based Python3 distribution or cluster engine is available and the Snakemake --use-conda environment deployment cannot be used

## Inputs

- Raw FASTQ files (single-end or paired-end, with or without spike-in/UMI sequences)
- Sample sequencing data table
- Single YAML input configuration file (working environment paths, genome references, sample information, experimental settings)
- ENCODE pre-built genome references (BWA index and annotated regions)
- conda_env.yaml environment specification

## Outputs

- Aggregated methylation quantification matrices (TAB-delimited bin-sample TXT files, original and filtered, Tabix-indexed)
- MultiQC (v1.0.dev0) summarized interactive HTML QC report
- Per-sample QC outputs (FastQC, insert-size/fragmentation metrics, MEDIPS saturation/coverage/enrichment QC)
- Per-sample aligned/filtered BAM outputs from BWA-MEM and SAMtools

## How to apply

Clone the MEDIPIPE repository (https://github.com/pughlab/MEDIPIPE) and build its core environment with Conda/Mamba using 'mamba env create --file conda_env.yaml'; Snakemake then automatically deploys per-tool execution environments, and users can pin software versions in YAML files. Obtain raw FASTQ files (e.g. the Nassiri et al. 2020 dataset of 163 samples from 6 brain cancer subtypes), prepare the sample sequencing data table, and download ENCODE pre-built genome references (BWA index and annotated regions) via the provided shell scripts. Write a single YAML input configuration file specifying working environment paths, genome references, sample information, and experimental settings (single-end vs paired-end, spike-in and/or UMI usage). Execute the Snakefile with --use-conda (e.g. 'snakemake --use-conda --cores N --configfile test/config_template.yaml'), or submit to a SLURM cluster via workflow/sbatch_Snakefile_template.sh so independent per-sample steps (Trim Galore, FastQC, UMI-tools, BWA-MEM, SAMtools, Picard, MEDIPS/QSEA/MEDStrand quantification, MultiQC aggregation, Tabix indexing) run in parallel.

## Related tools

- **Snakemake** (Workflow engine that automatically deploys execution environments and orchestrates per-sample parallel steps) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Underlying package system via which Snakemake containerized execution environments are automatically deployed)
- **Mamba** (Recommended fast Conda reimplementation used to create the core MEDIPIPE environment from conda_env.yaml) — https://github.com/mamba-org/mamba
- **SLURM** (Cluster engine on which the pipeline can be executed via the sbatch_Snakefile_template.sh submission script)
- **MEDIPIPE** (The end-to-end (cf)MeDIP-seq QC and analysis pipeline being configured and executed) — https://github.com/pughlab/MEDIPIPE

## Examples

```
mamba env create --file conda_env.yaml; snakemake --use-conda --cores 16 --configfile test/config_template.yaml
```

## Evaluation signals

- Snakemake completes without errors and all per-tool Conda environments are created automatically via --use-conda
- For the Nassiri et al. 2020 dataset, aggregated methylation matrices (before and after filtering), Tabix indexes (.tbi), the MultiQC report, and the summarized HTML QC report are produced for all 163 samples
- Filtered aggregated files correctly exclude sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions
- Group/batch-level QC view reproduces what is demonstrated in Supplementary Fig. S2
- Pinned tool versions (e.g. Trim Galore v0.6.7, BWA-MEM v0.7.17, SAMtools v1.17, MEDIPS v1.46.0, QSEA v1.20.0) are installed in the deployed environments as specified in the YAML files

## Limitations

- Requires a Conda-based Python3 distribution; Mamba installation is recommended but not assumed
- Software versions must be specified manually in YAML files if reproducibility of specific versions is needed
- Cluster execution requires adapting the provided SLURM sbatch template to the local cluster engine; other engines may need their own submission configuration

## Evidence

- [abstract] Portability via Snakemake/Conda: "ease of implementation and reproducibility with Snakemake containerized execution environments that will be automatically deployed via Conda"
- [intro] MEDIPIPE developed with Snakemake across cluster engines: "It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines"
- [intro] End-to-end scope of the pipeline: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation"
- [intro] Flexibility of input configurations: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
- [readme] Repository README installation and portability description: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments. Users can also specify the version of"
