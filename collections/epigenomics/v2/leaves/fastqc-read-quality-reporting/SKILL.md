---
name: fastqc-read-quality-reporting
description: Use when raw cfMeDIP-seq FASTQ files (single-end or paired-end, with
  or without UMI/spike-in sequences) need adapter and low-quality base trimming plus
  per-sample QC reports, and UMI-containing libraries must be filtered before trimming.
when_to_use_negative:
- Input is not raw FASTQ (e.g. already-aligned BAM or post-QC reads from an upstream
  pipeline)
- The library uses custom barcode sequences requiring ConsensusCruncher-style consensus
  workflows rather than UMI-tools regex extraction
- The goal is downstream methylation quantification or sample aggregation rather than
  read-level trimming and QC
edam_operation: http://edamontology.org/operation_0579
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: UMI-tools
  role: Extracts UMI barcodes via the regex method (v1.0.1) prior to trimming and
    filters out reads lacking the expected barcode sequence
- name: Trim Galore
  role: Removes adapter sequences and low-quality bases from raw single-end or paired-end
    reads (v0.6.7)
- name: FastQC
  role: Generates per-sample QC reports on both raw and trimmed/processed reads (v0.11.9)
- name: Snakemake
  role: Implements the trimming/QC rules and automatically deploys execution environments
    for reproducible, parallel per-sample runs
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Automatically deploys the Conda execution environments specified for each
    Snakemake rule
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:00d08801b7c84732edc36ccdcefe13bcc72e1e9f24d23f918986348f92af2b04
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
  edam_operation: http://edamontology.org/operation_0579
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fastqc-read-quality-reporting

## Summary

Read trimming and QC module of the MEDIPIPE (cf)MeDIP-seq pipeline: conditional UMI barcode extraction (UMI-tools) followed by Trim Galore adapter/quality trimming and FastQC reporting on both raw and processed reads, implemented as parallel Snakemake rules with Conda-deployed environments.

## When to use

Use when raw cfMeDIP-seq FASTQ files (single-end or paired-end, with or without UMI/spike-in sequences) need adapter and low-quality base trimming plus per-sample QC reports, and UMI-containing libraries must be filtered before trimming.

## When NOT to use

- Input is not raw FASTQ (e.g. already-aligned BAM or post-QC reads from an upstream pipeline)
- The library uses custom barcode sequences requiring ConsensusCruncher-style consensus workflows rather than UMI-tools regex extraction
- The goal is downstream methylation quantification or sample aggregation rather than read-level trimming and QC

## Inputs

- Raw FASTQ files (single-end or paired-end) per sample
- YAML configuration file specifying single-end vs paired-end mode and presence/absence of UMI barcodes

## Outputs

- Trimmed FASTQ files for every input sample
- FastQC QC reports (HTML) for both the raw and the trimmed/processed reads of each sample
- UMI-extracted/filtered FASTQ files when UMI barcodes are present

## How to apply

Parse the pipeline's YAML configuration file to determine whether reads are single-end or paired-end and whether UMI barcodes are present. If UMIs are present, run UMI-tools (v1.0.1) extract with the regex method — which permits variable barcode length — prior to trimming, discarding reads that do not carry the expected barcode sequence, so that sequence error suppression happens before adapter removal. Then run Trim Galore (v0.6.7) to strip adapter sequences and low-quality bases from the raw reads, handling both single-end and paired-end input. Run FastQC (v0.11.9) on both the raw reads and the trimmed/processed reads to generate per-sample QC reports. Implement steps as Snakemake rules with Conda-deployed execution environments so each single sample runs in parallel and software versions are pinned in YAML files.

## Related tools

- **UMI-tools** (Extracts UMI barcodes via the regex method (v1.0.1) prior to trimming and filters out reads lacking the expected barcode sequence)
- **Trim Galore** (Removes adapter sequences and low-quality bases from raw single-end or paired-end reads (v0.6.7))
- **FastQC** (Generates per-sample QC reports on both raw and trimmed/processed reads (v0.11.9))
- **Snakemake** (Implements the trimming/QC rules and automatically deploys execution environments for reproducible, parallel per-sample runs) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Automatically deploys the Conda execution environments specified for each Snakemake rule)

## Examples

```
umi_tools extract --extract-method=regex --stdin sample_R1.fastq.gz --stdout sample_R1.umi.fastq.gz && trim_galore sample_R1.umi.fastq.gz && fastqc sample_R1.fastq.gz sample_R1_trimmed.fq.gz
```

## Evaluation signals

- Trimmed FASTQ files exist for every input sample
- FastQC reports exist for both the raw and the processed/trimmed reads of each sample
- For UMI libraries, reads lacking the expected barcode sequence have been filtered out before trimming (regex-based UMI-tools extract step)
- Software versions match the pinned YAML files (UMI-tools v1.0.1, Trim Galore v0.6.7, FastQC v0.11.9)
- Snakemake rules execute in Conda-deployed environments and complete for single samples in parallel

## Limitations

- The UMI regex method allows variable barcode length, so barcode extraction tolerance depends on the configured regex and expected barcode sequence
- Flexibility is limited to single-end or paired-end reads with or without spike-in/UMI sequences; custom barcodes may need the separate cfMeDIP-seq-analysis-pipeline/ConsensusCruncher route
- Users must have a Conda-based Python3 distribution (Mamba recommended) for environment deployment

## Evidence

- [other] UMI execution precedes trimming: [evidence span withheld — anchor status: not_found]
- [intro] Article intro on read-type flexibility: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
- [readme] README on Snakemake environment deployment: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments. Users can also specify the version of"
- [readme] README on scope: "It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling."
