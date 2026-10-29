---
name: cfmedip-seq-read-qc
description: Use when starting from raw FASTQ files of cfMeDIP-seq (or (cf)MeDIP-seq)
  libraries and you need adapter trimming, conditional UMI barcode handling, and per-sample
  QC reports as the first module of an automated pipeline.
when_to_use_negative:
- Input data has already been adapter-trimmed or UMI-processed and only needs alignment
  or methylation quantification (this skill covers only module i, read trimming and
  QC)
- The data type is not cfMeDIP-seq / (cf)MeDIP-seq FASTQ (e.g. bulk WGBS or RNA-seq
  reads), since MEDIPIPE is a framework specifically designed for cfMeDIP-seq data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: UMI-tools
  role: Executed prior to trimming (v1.0.1) to extract UMI barcodes and filter out
    reads that do not contain the expected barcode sequence, using the regex method
    allowing variable barcode length
- name: Trim Galore
  role: Removes adapter sequences and low-quality bases from raw reads (v0.6.7), handling
    both single-end and paired-end input
- name: FastQC
  role: Generates per-sample QC reports on both the raw reads and the trimmed/processed
    reads (v0.11.9)
- name: Snakemake
  role: Workflow engine implementing the trimming/QC steps as rules with containerized
    execution environments automatically deployed
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Automatically deploys the Snakemake rule execution environments; users specify
    software versions in YAML files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:ca641803cead00061b6ce8ec3b9282a12b063291aa6750ab4a857cfb37cd8b97
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cfmedip-seq-read-qc

## Summary

Read-level trimming and quality-control module (module i) of the MEDIPIPE cfMeDIP-seq pipeline: conditional UMI barcode extraction with UMI-tools, adapter/quality trimming with Trim Galore, and FastQC reporting on raw and trimmed reads, orchestrated as Snakemake rules with Conda-deployed environments.

## When to use

Use when starting from raw FASTQ files of cfMeDIP-seq (or (cf)MeDIP-seq) libraries and you need adapter trimming, conditional UMI barcode handling, and per-sample QC reports as the first module of an automated pipeline.

## When NOT to use

- Input data has already been adapter-trimmed or UMI-processed and only needs alignment or methylation quantification (this skill covers only module i, read trimming and QC)
- The data type is not cfMeDIP-seq / (cf)MeDIP-seq FASTQ (e.g. bulk WGBS or RNA-seq reads), since MEDIPIPE is a framework specifically designed for cfMeDIP-seq data

## Inputs

- Raw FASTQ files (single-end or paired-end) from cfMeDIP-seq experiments
- YAML configuration file specifying single-end vs paired-end setting and presence/absence of UMI barcodes
- Optional UMI barcode sequences/regex specification

## Outputs

- Trimmed FASTQ files for every input sample
- FastQC QC reports for both the raw reads and the processed (trimmed) reads of each sample
- Extracted-UMI FASTQ files (when UMI barcodes are present)

## How to apply

Parse the pipeline's YAML configuration file to determine whether the library is single-end or paired-end and whether UMI barcodes are present. If UMI barcodes were used, run UMI-tools (v1.0.1) extract prior to trimming, applying the regex method that allows variable barcode length, and filter out reads lacking the expected barcode sequence; this is done before Trim Galore so downstream trimming and QC operate only on correctly barcoded reads. Then run Trim Galore (v0.6.7) to remove adapter sequences and low-quality bases from the raw reads, handling both single-end and paired-end input. Run FastQC (v0.11.9) on both the raw reads and the trimmed/processed reads to generate per-sample QC reports. Implement steps as Snakemake rules with Conda-deployed execution environments so the module is reproducible, runs on single samples in parallel, and can execute across cluster engines (e.g. SLURM) or stand-alone machines.

## Related tools

- **UMI-tools** (Executed prior to trimming (v1.0.1) to extract UMI barcodes and filter out reads that do not contain the expected barcode sequence, using the regex method allowing variable barcode length)
- **Trim Galore** (Removes adapter sequences and low-quality bases from raw reads (v0.6.7), handling both single-end and paired-end input)
- **FastQC** (Generates per-sample QC reports on both the raw reads and the trimmed/processed reads (v0.11.9))
- **Snakemake** (Workflow engine implementing the trimming/QC steps as rules with containerized execution environments automatically deployed) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Automatically deploys the Snakemake rule execution environments; users specify software versions in YAML files)

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- Trimmed FASTQ files are produced for every input sample
- FastQC reports exist for both the raw reads and the processed reads of each sample
- When UMI barcodes are configured, UMI-tools extract runs before Trim Galore and reads lacking the expected barcode sequence are filtered out
- The Snakemake pipeline resolves with Conda-deployed environments pinned to the configured software versions (UMI-tools v1.0.1, Trim Galore v0.6.7, FastQC v0.11.9)
- The module processes single-end and paired-end configurations consistently per the YAML configuration settings

## Limitations

- This module covers only read trimming and QC; methylation quantification, alignment, and sample aggregation are handled by other MEDIPIPE modules
- UMI handling requires the configuration to declare barcodes; if UMI presence is misconfigured, barcode extraction/filtering will be skipped or misapplied
- Execution requires a Conda-based Python3 distribution (Mamba installation recommended); running on clusters depends on supported cluster engines such as SLURM

## Evidence

- [abstract] One-stop cfMeDIP-seq QC: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [intro] Flexible read and UMI handling: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences. It can be applied to individual samples, as well as to aggregate multiple samples"
- [other] UMI-tools precedes Trim Galore: [evidence span withheld — anchor status: not_found]
- [readme] Snakemake-deployed environments: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments. Users can also specify the version of"
