---
name: adapter-trimming-quality-filtering
description: Use when raw FASTQ reads (single- or pair-ended NGS data, e.g. ChIP-seq/RNA-seq/ATAC-seq)
  must be cleaned of adapters, low-quality bases, and short or poorly mapping reads
  before alignment and downstream peak calling, differential expression, or nucleosome
  positioning.
when_to_use_negative:
- The workflow file must not mix single-ended and pair-ended data in the same CONFIG
  file — CIPHER does not handle that use-case (split them into separate runs).
- Only a single downstream tool (e.g. just a mapper or just a peak caller) is wanted
  — CIPHER currently only runs entire workflows, not subsets of tools.
- Input reads are already adapter-trimmed and quality-filtered by another tool; re-running
  trimming would be redundant.
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: FastQC
  role: 'Stage-1 raw sequence evaluation: quality and contamination control checks
    on raw FASTQ reads before trimming'
- name: BBDuk
  role: Adapter trimming and removal of low-quality/short reads (defaults exclude
    mapq < 20 and length < 10)
- name: BBMap
  role: Default read mapper applied after trimming/quality filtering (alternatives
    bwa, bowtie2, star, hisat2 via --aligner)
- name: MultiQC
  role: Aggregates per-sample QC outputs into a single PDF or HTML report
- name: Nextflow
  role: Workflow language in which the trimming/filtering/mapping stage is implemented
    (cipher.nf)
  repo: https://www.nextflow.io/
- name: CIPHER
  role: Workflow platform orchestrating the fixed pre-processing stage prior to mode-dependent
    downstream analysis
  repo: c-guzman/cipher-workflow-platform
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:ef85cf8095dca4fc5a4489f3dee1165016087acd830aba27a7a667c0a1b436d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBDuk
  - BBMap
  - CIPHER
  - Docker
  - FastQC
  - MACS2
  - MultiQC
  - Nextflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Adapter trimming and quality filtering

## Summary

The CIPHER pre-processing stage that evaluates raw reads (FastQC), trims adapters and removes low-quality or too-short reads (BBDuk, dropping reads with mapq < 20 or length < 10 during/after mapping with BBMap), and aggregates quality control (MultiQC) before any dataset-type-specific downstream analysis.

## When to use

Use when raw FASTQ reads (single- or pair-ended NGS data, e.g. ChIP-seq/RNA-seq/ATAC-seq) must be cleaned of adapters, low-quality bases, and short or poorly mapping reads before alignment and downstream peak calling, differential expression, or nucleosome positioning.

## When NOT to use

- The workflow file must not mix single-ended and pair-ended data in the same CONFIG file — CIPHER does not handle that use-case (split them into separate runs).
- Only a single downstream tool (e.g. just a mapper or just a peak caller) is wanted — CIPHER currently only runs entire workflows, not subsets of tools.
- Input reads are already adapter-trimmed and quality-filtered by another tool; re-running trimming would be redundant.

## Inputs

- Single-ended FASTQ files (.fastq.gz)
- Pair-ended FASTQ files (R1/R2 .fastq.gz)
- Tab-separated CONFIG file (5 columns single-ended, 6 columns pair-ended: MergeID, SampleID, FASTQ path(s), InputID, Mark)
- Reference genome FASTA (--fa)
- GTF annotation file (--gtf)

## Outputs

- Gzipped trimmed FASTQ files
- Sorted BAM files with alignment statistics
- Raw-read evaluation reports (FastQC)
- Aggregated quality control reports in PDF or HTML (MultiQC), written to './report' by default (override with --outdir)

## How to apply

Structure the step as the fixed stage-1 branch of the CIPHER Nextflow workflow (cipher.nf): first run FastQC on raw reads to evaluate quality and contamination, then trim adapter sequences and filter low-quality/short reads with BBDuk (the article lists trimming/adapter removal as pipeline stage 2 and read mapping and quality filtering as stage 3). Map filtered reads with the default BBMap aligner (alternatives bwa, bowtie2, star, hisat2 via the --aligner flag), enforcing the defaults mapq < 20 and length < 10 as exclusion thresholds. Input samples are described by a tab-separated config file (5 columns single-ended, 6 columns pair-ended: MergeID, SampleID, FASTQ paths, InputID, Mark) that drives which FASTQs are processed per sample/input pair. Emit gzipped trimmed FASTQs, sorted BAM files with alignment statistics, and PDF/HTML QC reports aggregated by MultiQC into the './report' directory (override with --outdir). The rationale is that trimming is a common pre-processing step that removes artifactual adapters and low-quality sequence so the computationally intensive mapping stage and downstream mode-dependent analysis are not biased by it.

## Related tools

- **FastQC** (Stage-1 raw sequence evaluation: quality and contamination control checks on raw FASTQ reads before trimming)
- **BBDuk** (Adapter trimming and removal of low-quality/short reads (defaults exclude mapq < 20 and length < 10))
- **BBMap** (Default read mapper applied after trimming/quality filtering (alternatives bwa, bowtie2, star, hisat2 via --aligner))
- **MultiQC** (Aggregates per-sample QC outputs into a single PDF or HTML report)
- **Nextflow** (Workflow language in which the trimming/filtering/mapping stage is implemented (cipher.nf)) — https://www.nextflow.io/
- **CIPHER** (Workflow platform orchestrating the fixed pre-processing stage prior to mode-dependent downstream analysis) — c-guzman/cipher-workflow-platform

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config config.txt --fa ref.fa --gtf ref.gtf --lib s --readLen 75
```

## Evaluation signals

- Run completes without error and './report' (or --outdir) contains gzipped trimmed FASTQ files for each input sample.
- Sorted BAM files exist per sample and carry alignment statistics consistent with the reference genome used.
- MultiQC report (PDF or HTML) is present and aggregates the per-sample QC results.
- No reads shorter than the length threshold (length < 10) and no alignments below mapq 20 remain in the filtered output.
- Each processed sample matches its CONFIG row (MergeID/SampleID/InputID pairing) with input files correctly excluded from the sample tracks.

## Limitations

- CIPHER does not handle mixing single-ended and pair-ended data in the same CONFIG file.
- Trimming/filtering cannot be run in isolation: CIPHER currently only runs entire workflows, not subsets of tools.
- When running without Singularity containers, all required software must be installed manually (tools used are listed inside cipher.nf).
- On a local machine the -qs <INT> flag must limit parallelized processes or the workflow can abruptly end due to memory exhaustion.

## Evidence

- [abstract] read trimming and adapter removal, (3) read mapping and quality filtering: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering"
- [methods] Trimming adapter sequences is a common pre-processing step during NGS data analysis: "Trimming adapter sequences is a common pre-processing step during NGS data analysis"
- [readme] CIPHER does not handle single/pair-ended mixing in configs: [evidence span withheld — anchor status: external_doc]
- [readme] CIPHER runs via nextflow run cipher.nf: [evidence span withheld — anchor status: external_doc]
