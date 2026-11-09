---
name: umi-barcode-extraction-regex-matching
description: Use when input FASTQ libraries (single-end or paired-end cfMeDIP-seq)
  carry UMI barcodes for sequence error suppression and barcodes must be extracted
  and reads lacking the expected barcode filtered before adapter trimming.
when_to_use_negative:
- The library was prepared without UMI barcodes (or the YAML config does not specify
  UMI presence) — extraction would be skipped and trimming should proceed directly
  on raw reads
- The input is already trimmed or already barcode-extracted FASTQ, not raw reads from
  the sequencer
- The goal is downstream post-alignment deduplication rather than pre-trimming barcode
  extraction from raw FASTQ
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3673
tools:
- name: UMI-tools
  role: Runs `extract` (v1.0.1) with the regex method prior to trimming to pull out
    UMI barcodes and filter out reads lacking the expected barcode sequence
- name: Snakemake
  role: Implements the extraction/trimming/QC steps as Snakemake rules with automatically
    deployed execution environments so the module is reproducible and parallel per
    sample
  repo: https://github.com/pughlab/MEDIPIPE
- name: Conda
  role: Deploys the pinned execution environment (software versions specified in YAML
    files) for the UMI extraction rule
- name: Trim Galore
  role: Executed after UMI extraction (v0.6.7) to remove adapter sequences and low-quality
    bases; UMI-tools must run prior to it
- name: FastQC
  role: Generates per-sample QC reports on raw and processed reads downstream of extraction
    (v0.11.9)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:162fdd8e982943584119a0b9b66a531b5071354701b908de58af3ed688999836
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# umi-barcode-extraction-regex-matching

## Summary

Conditional UMI barcode extraction step from MEDIPIPE module (i) (cfMeDIP-seq QC): when libraries incorporate UMI barcodes, UMI-tools extract (v1.0.1) with a regex method is run prior to Trim Galore to pull barcodes out of reads and remove reads that lack the expected barcode sequence.

## When to use

Use when input FASTQ libraries (single-end or paired-end cfMeDIP-seq) carry UMI barcodes for sequence error suppression and barcodes must be extracted and reads lacking the expected barcode filtered before adapter trimming.

## When NOT to use

- The library was prepared without UMI barcodes (or the YAML config does not specify UMI presence) — extraction would be skipped and trimming should proceed directly on raw reads
- The input is already trimmed or already barcode-extracted FASTQ, not raw reads from the sequencer
- The goal is downstream post-alignment deduplication rather than pre-trimming barcode extraction from raw FASTQ

## Inputs

- Raw FASTQ files (single-end or paired-end) from cfMeDIP-seq libraries containing UMI barcode sequences
- YAML configuration file specifying single-end vs paired-end mode and whether UMI barcodes are present
- Expected UMI barcode sequence/regex pattern for the library

## Outputs

- Barcode-extracted FASTQ reads with UMI sequences removed from the read and encoded in the read name (UMI-tools extract output)
- Filtered read set in which reads lacking the expected barcode sequence have been removed
- Intermediate FASTQ passed on to Trim Galore trimming

## How to apply

Parse the pipeline's YAML configuration to determine whether the experiment is single-end or paired-end and whether UMI barcodes are present; this is a conditional decision point — if no UMI barcodes are configured, skip extraction and proceed directly to trimming. If UMI barcodes were used, run UMI-tools (v1.0.1) extract on the raw FASTQ reads before any trimming, using the regex method so that variable-length barcode sequences are tolerated. The regex-based extraction both isolates the UMI barcode sequence and acts as a filter step: reads that do not contain the expected barcode sequence are discarded. Only after extraction/filtering should Trim Galore (v0.6.7) be run for adapter and low-quality base trimming, followed by FastQC (v0.11.9) QC reporting. Implement the step as a Snakemake rule with a Conda-deployed environment (software versions pinned in YAML files) so extraction is reproducible and can run per-sample in parallel.

## Related tools

- **UMI-tools** (Runs `extract` (v1.0.1) with the regex method prior to trimming to pull out UMI barcodes and filter out reads lacking the expected barcode sequence)
- **Snakemake** (Implements the extraction/trimming/QC steps as Snakemake rules with automatically deployed execution environments so the module is reproducible and parallel per sample) — https://github.com/pughlab/MEDIPIPE
- **Conda** (Deploys the pinned execution environment (software versions specified in YAML files) for the UMI extraction rule)
- **Trim Galore** (Executed after UMI extraction (v0.6.7) to remove adapter sequences and low-quality bases; UMI-tools must run prior to it)
- **FastQC** (Generates per-sample QC reports on raw and processed reads downstream of extraction (v0.11.9))

## Examples

```
umi_tools extract --stdin sample_R1.fastq.gz --bc-pattern=NNNNCCCC... --log=sample.log > sample_extracted.fastq.gz
```

## Evaluation signals

- Extracted FASTQ files exist for every input sample that was configured with UMI barcodes
- Reads lacking the expected UMI barcode sequence are absent from the extracted output (filter applied)
- UMI-tools runs before Trim Galore in the rule dependency graph, and the regex method is used (supporting variable barcode length)
- Software versions are pinned in the Conda/YAML environment files (e.g. UMI-tools v1.0.1) for reproducibility
- FastQC reports exist for both the raw reads and the trimmed/processed reads of each sample, confirming the module completed

## Limitations

- Extraction is conditional: it only applies to libraries prepared with UMI/spike-in sequences; MEDIPIPE otherwise handles reads 'with or without spike-in/UMI sequences'
- The regex method allows variable barcode length, so custom barcode sequences must be encoded correctly in the regex/ configuration to avoid incorrect filtering
- Portability depends on Snakemake's Conda environment deployment; runs require a Conda-based Python3 distribution (Mamba recommended) and may target cluster engines such as SLURM or stand-alone machines

## Evidence

- [other] UMI-tools runs before Trim Galore to extract barcodes and filter reads lacking expected barcode via regex: [evidence span withheld — anchor status: not_found]
- [other] Regex method allows variable barcode length (UMI-tools v1.0.1): [evidence span withheld — anchor status: not_found]
- [readme] MEDIPIPE flexibility: single-end or paired-end reads with or without UMI sequences: "MEDIPIPE can deal with single-end or paired-end reads, which comes along with or without spike-in/UMI sequences."
- [readme] Snakemake auto-deploys execution environments; software versions pinned in YAML files: [evidence span withheld — anchor status: not_found]
