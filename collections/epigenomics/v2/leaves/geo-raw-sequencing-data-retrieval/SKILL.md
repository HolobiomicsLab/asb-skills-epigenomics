---
name: geo-raw-sequencing-data-retrieval
description: Use when starting from a list of GEO accession datasets (e.g., ChIP-seq
  or DNase-seq studies) and raw FASTQ files for treatment and control samples are
  needed as pipeline input.
when_to_use_negative:
- Input is already processed data (BAM alignment files or called peaks) rather than
  raw FASTQ from GEO
- The data source is not GEO-deposited public-domain data and no accession list exists
  to drive retrieval
- Only summary-level QC metrics are needed and the goal is to query an existing quality
  atlas rather than recompute metrics from raw reads
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ChiLin
  role: Consumes the retrieved FASTQ files; per-dataset configs are generated with
    'chilin gen' listing treatment/control FASTQ paths
  repo: https://github.com/cfce/chilin
- name: bwa
  role: Default mapper applied to the retrieved raw reads in the subsequent QC step,
    making single-end vs paired-end distinction important at retrieval time
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:ccb3e9186af25fd91c1441bdbd2a116f6c8f321442fe3753d9c5dd4fe0bf3c4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - Cistrome
  - R
  - bedtools
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# geo-raw-sequencing-data-retrieval

## Summary

Retrieve raw single-end or paired-end ChIP-seq/DNase-seq FASTQ files (treatment and control) for GEO-deposited datasets so they can be fed into downstream quality-control pipelines such as ChiLin. This step grounds the skill in the provenance of the public-domain data (e.g., the 23,677 samples from 11,265 GEO datasets cited in the ChiLin atlas).

## When to use

Use when starting from a list of GEO accession datasets (e.g., ChIP-seq or DNase-seq studies) and raw FASTQ files for treatment and control samples are needed as pipeline input.

## When NOT to use

- Input is already processed data (BAM alignment files or called peaks) rather than raw FASTQ from GEO
- The data source is not GEO-deposited public-domain data and no accession list exists to drive retrieval
- Only summary-level QC metrics are needed and the goal is to query an existing quality atlas rather than recompute metrics from raw reads

## Inputs

- List of GEO dataset accessions (GSE/GSM identifiers) for ChIP-seq or DNase-seq studies
- Species specification (hg19, hg38, mm9, mm10) used to select matching reference genome data

## Outputs

- Raw single-end or paired-end ChIP-seq/DNase-seq FASTQ files for treatment samples
- Raw FASTQ files for matched control samples
- Per-dataset file organization ready for ChiLin config generation ('chilin gen')

## How to apply

Identify the GEO-deposited datasets of interest and obtain the raw FASTQ files for both treatment and control samples, distinguishing single-end from paired-end reads since this affects later mapping with bwa. Alongside the sequencing data, confirm which species' reference data are required (e.g., hg19/hg38 or mm9/mm10 genome index, chrom_len, DHS/blacklist regions, geneTable) because ChiLin's configuration requires these per-species dependencies. Organize the per-dataset FASTQ paths so they can be written into a per-dataset ChiLin configuration generated with 'chilin gen', then listed in a batch.conf for 'chilin batch' processing. The rationale is that the historical public-domain data obtained via GEO (11,265 datasets in the ChiLin atlas) form the raw material whose quality metrics populate the compiled database.

## Related tools

- **ChiLin** (Consumes the retrieved FASTQ files; per-dataset configs are generated with 'chilin gen' listing treatment/control FASTQ paths) — https://github.com/cfce/chilin
- **bwa** (Default mapper applied to the retrieved raw reads in the subsequent QC step, making single-end vs paired-end distinction important at retrieval time) — https://github.com/lh3/bwa

## Evaluation signals

- Each GEO accession in the input list yields both treatment and control FASTQ files (where a control exists for the dataset)
- Read type (single-end vs paired-end) is recorded per dataset because it determines bwa mapping mode downstream
- Retrieved FASTQ files correspond to the correct species so the matching reference data (hg19/hg38/mm9/mm10 genome index, chrom_len, DHS/blacklist, geneTable) can be paired with them
- FASTQ paths resolve on disk and can be written into the [basics] section of a per-dataset 'chilin gen' configuration

## Limitations

- ChiLin's downstream 13-step workflow expects FASTQ input; pre-aligned or pre-processed GEO submissions would not fit the 'chilin gen' [basics] configuration directly
- Public GEO submissions vary in whether matched controls are deposited, which affects which QC steps (e.g., peak calling with controls) can run
- The compiled atlas covers eight literature-based classified categories of ChIP-seq and DNase-seq data; assay types outside this scope may not have comparable historical reference metrics

## Evidence

- [other] Obtain the raw single-end or paired-end ChIP-seq/DNase-seq FASTQ files (treatment and control) for the GEO-deposited datasets to be processed.: [evidence span withheld — anchor status: not_found]
- [intro] applied to 11,265 datasets from a wide variety of studies in the public domain obtained via GEO: "applied to 11,265 datasets from a wide variety of studies in the public domain obtained via GEO"
- [other] Generate a per-dataset configuration with 'chilin gen', filling the [basics] section (species, treatment/control FASTQ paths, narrow/broad peak type): [evidence span withheld — anchor status: not_found]
