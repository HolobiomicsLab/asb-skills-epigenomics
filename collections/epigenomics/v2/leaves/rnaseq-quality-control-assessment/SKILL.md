---
name: rnaseq-quality-control-assessment
description: Use when raw RNA-seq FASTQ files plus a tab-separated CONFIG file (MergeID,
  SampleID, Path, InputID, Mark) are the input and QC/bias/contamination assessment
  is needed before DEG analysis.
when_to_use_negative:
- The input is already a count matrix or processed expression table rather than raw
  FASTQ reads — the QC stage operates on raw sequence data
- The sample sheet mixes single-ended and pair-ended FASTQ files in one CONFIG — CIPHER
  does not handle this use-case yet
- The goal is ChIP-seq, DNase-seq, MNase-seq, ATAC-seq, or GRO-seq QC rather than
  RNA-seq; use the corresponding CIPHER mode instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CIPHER
  role: Workflow platform orchestrating the RNA-seq QC stages (raw sequence evaluation,
    trimming, mapping and quality filtering, QC assessment)
  repo: c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language in which CIPHER's QC pipeline is implemented and executed
  repo: https://www.nextflow.io/
- name: Singularity
  role: Container technology used to package QC tool dependencies for reproducible
    QC runs
  repo: http://singularity.lbl.gov/index.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:0f68ce8fa05df8964af83f4c102d559096fc317f8980856792b3a3127e2317b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - DESeq2
  - Nextflow
  - QoRTs
  - RUVSeq
  - STAR
  - Singularity
  - edgeR
  - featureCounts
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rnaseq-quality-control-assessment

## Summary

Performs extensive quality and contamination control on RNA-seq datasets using CIPHER's QC stage (raw sequence evaluation, trimming, mapping quality filtering, and QC reporting) so that only trustworthy reads feed downstream differential expression analysis.

## When to use

Use when raw RNA-seq FASTQ files plus a tab-separated CONFIG file (MergeID, SampleID, Path, InputID, Mark) are the input and QC/bias/contamination assessment is needed before DEG analysis.

## When NOT to use

- The input is already a count matrix or processed expression table rather than raw FASTQ reads — the QC stage operates on raw sequence data
- The sample sheet mixes single-ended and pair-ended FASTQ files in one CONFIG — CIPHER does not handle this use-case yet
- The goal is ChIP-seq, DNase-seq, MNase-seq, ATAC-seq, or GRO-seq QC rather than RNA-seq; use the corresponding CIPHER mode instead

## Inputs

- Raw single-ended RNA-seq FASTQ files (.fastq.gz)
- Raw pair-ended RNA-seq FASTQ R1/R2 files (.fastq.gz)
- Tab-separated CONFIG file (MergeID, SampleID, Path1[, Path2], InputID, Mark)
- Reference genome FASTA file (--fa)
- GTF annotation file (--gtf)

## Outputs

- Raw sequence evaluation (FastQC-style) reports per sample
- Trimmed/adapter-removed reads
- Mapping and quality-filtering statistics
- Extensive quality control assessment report covering biases, contaminations, and errors

## How to apply

Build a tab-separated CONFIG file with 5 columns for single-ended data (MergeID, SampleID, FASTQ path, InputID set to '-', Mark as MergeID) or 6 columns for pair-ended data, and never mix single- and pair-ended reads in one CONFIG. Run CIPHER via Nextflow in RNA-seq mode with the FASTA and GTF reference files, using Singularity containers for reproducibility; the pipeline executes the five canonical stages: raw sequence evaluation, read trimming and adapter removal, read mapping and quality filtering, visualization track generation, and extensive quality control assessment. The QC stage aggregates tools to identify potential biases, contaminations, and errors before downstream differential gene expression analysis. When running on a non-cluster machine, set the -qs flag to limit parallelized processes, as excessive parallelism exhausts memory and aborts the workflow. Note that CIPHER's RNA-seq pipeline assumes two-type experimental designs (e.g. WT versus KO), so multi-factor or time-series DEG designs require downstream tools like DESeq2/edgeR outside the QC scope.

## Related tools

- **CIPHER** (Workflow platform orchestrating the RNA-seq QC stages (raw sequence evaluation, trimming, mapping and quality filtering, QC assessment)) — c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language in which CIPHER's QC pipeline is implemented and executed) — https://www.nextflow.io/
- **Singularity** (Container technology used to package QC tool dependencies for reproducible QC runs) — http://singularity.lbl.gov/index.html

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode RNA-seq --config CONFIG --fa genome.fa --gtf annotation.gtf --lib LIB --readLen LENGTH -qs 4
```

## Evaluation signals

- The run uses a valid tab-separated CONFIG with the correct column count (5 single-ended, 6 pair-ended) and InputID set to '-' for RNA-seq samples
- All five pipeline stages complete: raw sequence evaluation, trimming/adapter removal, mapping and quality filtering, visualization track generation, and QC assessment
- QC reports flag no unresolved biases, contaminations, or adapter content before downstream differential expression analysis
- On a local machine the -qs parallelization flag was set so the workflow does not abruptly end from memory exhaustion
- All QC steps executed inside the Singularity container (cipher.img), preserving reproducibility

## Limitations

- CIPHER does not handle mixing single-ended and pair-ended data in the same CONFIG file
- Running without limiting parallelism (-qs) on a local machine can exhaust memory and abruptly terminate the workflow
- The RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO), so downstream DEG analysis is restricted; this QC skill alone does not lift that limitation

## Evidence

- [methods] CIPHER incorporates a number of quality control tools to identify potential biases, contaminations, and errors in NGS datasets: "CIPHER incorporates a number of quality control tools to identify potential biases, contaminations, and errors in NGS datasets"
- [abstract] Five canonical pipeline stages: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
- [readme] CONFIG file constraints: [evidence span withheld — anchor status: external_doc]
- [readme] Memory parallelization caveat: [evidence span withheld — anchor status: external_doc]
- [discussion] Two-design restriction on downstream DEG: "CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO)"
