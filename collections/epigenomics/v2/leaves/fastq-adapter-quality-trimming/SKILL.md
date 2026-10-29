---
name: fastq-adapter-quality-trimming
description: Use when raw cfMeDIP-seq (or MeDIP-seq) FASTQ files enter a pipeline
  and need adapter/quality trimming plus QC, especially when the library may or may
  not carry UMI or spike-in sequences configured via a YAML file.
when_to_use_negative:
- Input FASTQ files are already adapter-trimmed and QC-passed, so re-running Trim
  Galore would only degrade reads
- The downstream goal is not trimming/QC — e.g. alignment, methylation quantification,
  or sample aggregation steps of the (cf)MeDIP-seq workflow
- Non-MeDIP-seq data types (e.g. methylation arrays or bisulfite workflows) whose
  adapter/QC requirements differ from cfMeDIP-seq FASTQ processing
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Trim Galore
  role: Removes adapter sequences and low-quality bases from raw single-end or paired-end
    FASTQ reads (v0.6.7)
- name: UMI-tools
  role: Conditionally run before trimming to extract UMI barcodes via the regex method
    (variable barcode length) and filter out reads lacking the expected barcode (v1.0.1)
- name: FastQC
  role: Generates per-sample QC reports on both raw reads and trimmed/processed reads
    (v0.11.9)
- name: Snakemake
  role: Implements the trimming/QC steps as rules with containerized execution environments
    so software versions are reproducible and samples run in parallel
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Automatically deploys the execution environments for each Snakemake rule,
    with tool versions pinned in YAML files
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:87c538241fb098bc621ebba14813146bebe9a3d6a136855264b44192b9aa7659
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - Conda
  - FastQC
  - Snakemake
  - Trim Galore
  - UMI-tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fastq-adapter-quality-trimming

## Summary

Module (i) of the MEDIPIPE cfMeDIP-seq pipeline: read trimming and QC that removes adapters and low-quality bases from raw FASTQ reads with Trim Galore, conditionally extracts UMI barcodes with UMI-tools, and generates FastQC reports on raw and processed reads.

## When to use

Use when raw cfMeDIP-seq (or MeDIP-seq) FASTQ files enter a pipeline and need adapter/quality trimming plus QC, especially when the library may or may not carry UMI or spike-in sequences configured via a YAML file.

## When NOT to use

- Input FASTQ files are already adapter-trimmed and QC-passed, so re-running Trim Galore would only degrade reads
- The downstream goal is not trimming/QC — e.g. alignment, methylation quantification, or sample aggregation steps of the (cf)MeDIP-seq workflow
- Non-MeDIP-seq data types (e.g. methylation arrays or bisulfite workflows) whose adapter/QC requirements differ from cfMeDIP-seq FASTQ processing

## Inputs

- Raw FASTQ files (single-end or paired-end) from a cfMeDIP-seq / (cf)MeDIP-seq experiment
- YAML configuration file specifying single-end vs paired-end setting and presence of UMI/spike-in sequences
- Sample list or per-sample FASTQ paths consumed by Snakemake rules

## Outputs

- Trimmed FASTQ files for every input sample
- UMI-extracted/filtered FASTQ files when UMI barcodes are configured
- FastQC per-sample QC reports for both raw and trimmed/processed reads

## How to apply

Parse the experiment YAML configuration to decide single-end vs paired-end handling and whether UMI barcodes are present. If UMI barcodes were used, run UMI-tools (v1.0.1) extract with the regex method — which permits variable barcode length — prior to trimming, so that reads lacking the expected barcode sequence are filtered out before adapter removal. Then run Trim Galore (v0.6.7) to strip adapter sequences and low-quality bases, handling both single-end and paired-end input. Run FastQC (v0.11.9) on both the raw FASTQ files and the trimmed/processed reads to produce per-sample QC reports. Implement the conditional UMI extraction, trimming, and QC as Snakemake rules with Conda-deployed execution environments (YAML-pinned tool versions) so the module is reproducible, runs per sample in parallel, and can execute across cluster engines such as SLURM or stand-alone machines.

## Related tools

- **Trim Galore** (Removes adapter sequences and low-quality bases from raw single-end or paired-end FASTQ reads (v0.6.7))
- **UMI-tools** (Conditionally run before trimming to extract UMI barcodes via the regex method (variable barcode length) and filter out reads lacking the expected barcode (v1.0.1))
- **FastQC** (Generates per-sample QC reports on both raw reads and trimmed/processed reads (v0.11.9))
- **Snakemake** (Implements the trimming/QC steps as rules with containerized execution environments so software versions are reproducible and samples run in parallel) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Automatically deploys the execution environments for each Snakemake rule, with tool versions pinned in YAML files) — https://github.com/pughlab/MEDIPIPE

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE && cd MEDIPIPE && snakemake --use-conda --cores 1
```

## Evaluation signals

- Trimmed FASTQ files are produced for every input sample (module acceptance criterion)
- FastQC reports exist for both the raw reads and the trimmed/processed reads of each sample
- When UMI barcodes are configured, UMI-tools extraction occurs before Trim Galore and reads lacking the expected barcode sequence are absent from the output FASTQ
- Single-end vs paired-end handling in the output matches the YAML configuration (paired trimming yields coordinated R1/R2 files)
- Each step runs inside its Conda-deployed environment with tool versions matching those pinned in the YAML files (UMI-tools 1.0.1, Trim Galore 0.6.7, FastQC 0.11.9)

## Limitations

- UMI barcode handling assumes barcodes match the configured regex; reads not matching the expected barcode sequence are discarded rather than rescued
- Tool versions are fixed in YAML files (e.g. Trim Galore v0.6.7, UMI-tools v1.0.1, FastQC v0.11.9); users must edit YAML to change versions
- The module processes single samples in parallel and does not itself perform sample aggregation or methylation quantification

## Evidence

- [abstract] MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [readme] MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences.: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences. It can be applied to individual samples"
- [intro] MEDIPIPE was developed with [Snakemake], which will automatically deploy the execution environments: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments"
- [readme] It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines: "It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines"
