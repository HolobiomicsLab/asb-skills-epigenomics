---
name: snakemake-workflow-module-implementation
description: Use when implementing or reconstructing a reproducible bioinformatics
  module (e.g., FASTQ trimming/QC) from a published pipeline, where steps must be
  encoded as Snakemake rules with Conda-deployed environments and configurable behavior.
when_to_use_negative:
- Input data is not raw FASTQ from (cf)MeDIP-seq experiments — e.g., already-aligned
  BAM files or feature-level tables — since this module starts from raw FASTQ files
- A one-off interactive analysis is needed where workflow reproducibility, environment
  deployment, and cluster parallelism add no value
- The protocol requires consensus generation from custom barcode lists via a different
  framework (e.g., ConsensusCruncher with a --blist barcode file) rather than UMI-tools
  extraction
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Snakemake
  role: 'Workflow engine: encodes each trimming/QC step as a rule and automatically
    deploys execution environments'
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Deploys the containerized/Conda execution environments for each rule
- name: UMI-tools
  role: Extracts UMI barcodes (v1.0.1, regex method for variable barcode length) and
    filters reads lacking the expected barcode, prior to trimming
- name: Trim Galore
  role: Removes adapter sequences and low-quality bases from raw reads (v0.6.7), handling
    single-end and paired-end input
- name: FastQC
  role: Generates per-sample QC reports on both raw and trimmed/processed reads (v0.11.9)
- name: Mamba
  role: Recommended installer for the Conda-based Python3 distribution used to deploy
    environments
  repo: https://github.com/mamba-org/mamba
- name: SLURM
  role: Cluster engine on which the Snakemake module can be executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:b9a74e236ddff59ca1687ffd7a03bbccd0df84870db4f7d90b9835c59eaf6a03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - Conda
  - FastQC
  - Mamba
  - SLURM
  - Snakemake
  - Trim Galore
  - UMI-tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# snakemake-workflow-module-implementation

## Summary

Implement a pipeline module (e.g., MEDIPIPE's read trimming and QC stage) as Snakemake rules whose execution environments are automatically deployed via Conda, so each step (UMI-tools extract, Trim Galore, FastQC) is reproducible, versioned in YAML files, and runs on individual samples in parallel.

## When to use

Use when implementing or reconstructing a reproducible bioinformatics module (e.g., FASTQ trimming/QC) from a published pipeline, where steps must be encoded as Snakemake rules with Conda-deployed environments and configurable behavior.

## When NOT to use

- Input data is not raw FASTQ from (cf)MeDIP-seq experiments — e.g., already-aligned BAM files or feature-level tables — since this module starts from raw FASTQ files
- A one-off interactive analysis is needed where workflow reproducibility, environment deployment, and cluster parallelism add no value
- The protocol requires consensus generation from custom barcode lists via a different framework (e.g., ConsensusCruncher with a --blist barcode file) rather than UMI-tools extraction

## Inputs

- Raw FASTQ files (single-end or paired-end) per sample
- YAML configuration file specifying single-end/paired-end mode, presence of UMI/spike-in sequences, and software versions
- Conda environment YAML files pinning tool versions (UMI-tools v1.0.1, Trim Galore v0.6.7, FastQC v0.11.9)

## Outputs

- Trimmed/processed FASTQ files for every input sample
- FastQC QC reports (HTML) for both raw reads and trimmed/processed reads, per sample
- Snakemake rule definitions with Conda-deployed execution environments

## How to apply

Parse the pipeline's YAML configuration file to determine experimental settings such as single-end vs paired-end reads and whether UMI barcodes are present, then branch the workflow accordingly. Encode each processing step as a Snakemake rule: if UMIs are present, run UMI-tools (v1.0.1) extract with the regex method (allowing variable barcode length) prior to trimming and filter out reads lacking the expected barcode sequence; then run Trim Galore (v0.6.7) to remove adapter sequences and low-quality bases from raw reads, handling both single-end and paired-end input; finally run FastQC (v0.11.9) on both the raw reads and the trimmed/processed reads. Pin software versions in the YAML environment files so Snakemake automatically deploys the execution environments via Conda (Mamba recommended), which also makes the module portable across cluster engines such as SLURM or stand-alone machines. Structure rules so single samples run in parallel. Validation: the module is accepted when trimmed FASTQ files are produced for every input sample and FastQC reports exist for both the raw and the processed reads of each sample.

## Related tools

- **Snakemake** (Workflow engine: encodes each trimming/QC step as a rule and automatically deploys execution environments) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Deploys the containerized/Conda execution environments for each rule)
- **UMI-tools** (Extracts UMI barcodes (v1.0.1, regex method for variable barcode length) and filters reads lacking the expected barcode, prior to trimming)
- **Trim Galore** (Removes adapter sequences and low-quality bases from raw reads (v0.6.7), handling single-end and paired-end input)
- **FastQC** (Generates per-sample QC reports on both raw and trimmed/processed reads (v0.11.9))
- **Mamba** (Recommended installer for the Conda-based Python3 distribution used to deploy environments) — https://github.com/mamba-org/mamba
- **SLURM** (Cluster engine on which the Snakemake module can be executed)

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- Trimmed FASTQ files are produced for every input sample
- FastQC reports exist for both the raw reads and the trimmed/processed reads of each sample
- UMI-containing libraries show UMI-tools extract ran before Trim Galore, with reads lacking the expected barcode sequence removed
- Each rule's Conda environment resolves with the pinned versions (UMI-tools v1.0.1, Trim Galore v0.6.7, FastQC v0.11.9) rather than drift
- The workflow handles both single-end and paired-end configurations as set in the YAML config, and samples run in parallel without path collisions

## Limitations

- Behavior is configuration-dependent: the module must correctly branch on single-end vs paired-end and on presence/absence of UMI sequences; a misconfigured YAML produces wrong or missing outputs
- The regex-based UMI extraction allows variable barcode length, so it tolerates but does not strictly enforce fixed-length barcodes; custom barcode protocols may need a different approach (cf. ConsensusCruncher --blist)
- Portability relies on Conda environment deployment; environments must be deployable on the target cluster engine (e.g., SLURM) or stand-alone machine

## Evidence

- [intro] MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments: [evidence span withheld — anchor status: not_found]
- [abstract] Snakemake containerized execution environments: "ease of implementation and reproducibility with Snakemake containerized execution environments that will be automatically deployed via Conda"
- [other] When UMI barcodes are present, UMI-tools is executed prior to Trim Galore: [evidence span withheld — anchor status: not_found]
- [other] Run FastQC (v0.11.9) on both the raw reads and the trimmed/processed reads: [evidence span withheld — anchor status: not_found]
- [readme] MEDIPIPE can deal with single-end or paired-end reads: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences. It can be applied to individual samples, as well as to aggregate multiple samples"
- [readme] Users can also specify the version of softwares to be installed in YAML files: "Users can also specify the version of softwares to be installed in YAML files. It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines"
