---
name: nextflow-workflow-execution
description: Use when raw single- or pair-ended FASTQ datasets for one or more NGS
  assays must be processed reproducibly end-to-end via Nextflow, or when comparing
  pipeline runtime/user-input burden against alternatives.
when_to_use_negative:
- You need to run only a subset of tools rather than an entire workflow — the article
  states 'CIPHER currently only runs entire workflows, but we are aware that individuals
  may prefer to use only a subset of tools'.
- You need to mix single-ended and pair-ended data in one run — the README warns 'DO
  NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE. CIPHER DOES
  NOT HANDLE THIS USE-CASE YET.'
- You need multi-factor or time-series RNA-seq differential designs — CIPHER's RNA-seq
  pipeline is largely optimized for two-type experimental designs (e.g. WT versus
  KO).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0080
tools:
- name: CIPHER
  role: The Nextflow workflow platform being executed; runs entire NGS pipelines from
    raw FASTQ to QC and downstream analysis
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language/engine that launches cipher.nf, manages task parallelization,
    and supports cluster executors (SGE, SLURM, LSF, PBS/Torque)
  repo: https://www.nextflow.io/
- name: Docker
  role: Container-based virtualization used to encapsulate dependencies for reproducible
    pipeline execution (paper); README also supports Singularity
- name: MultiQC
  role: Aggregates the extensive per-dataset quality control assessments produced
    by the pipeline
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:90bdb0cb74f2d8ad237c6eb3b4e2fc7528da7c95d7af6eafef05da1760e685bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - MAP-RSeq
  - MultiQC
  - Nextflow
  - bcbio-nextgen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nextflow-workflow-execution

## Summary

Execute CIPHER's Nextflow-based NGS workflow platform (ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq, GRO-seq) with containerization, running entire pipelines from raw FASTQ evaluation through QC, peak calling, or differential expression with parallel dataset processing.

## When to use

Use when raw single- or pair-ended FASTQ datasets for one or more NGS assays must be processed reproducibly end-to-end via Nextflow, or when comparing pipeline runtime/user-input burden against alternatives.

## When NOT to use

- You need to run only a subset of tools rather than an entire workflow — the article states 'CIPHER currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools'.
- You need to mix single-ended and pair-ended data in one run — the README warns 'DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE. CIPHER DOES NOT HANDLE THIS USE-CASE YET.'
- You need multi-factor or time-series RNA-seq differential designs — CIPHER's RNA-seq pipeline is largely optimized for two-type experimental designs (e.g. WT versus KO).

## Inputs

- single-ended or pair-ended FASTQ files (.fastq.gz) with optional input/control files
- tab-separated CONFIG file (5 columns single-end, 6 columns pair-end: MergeID, SampleID, Path 1, Path 2, InputID, Mark)
- reference genome FASTA (--fa/--fasta)
- Ensembl/Gencode GTF annotation (--gtf)
- library type (--lib) and read length (--readLen)

## Outputs

- QC reports
- aligned BAM files
- bigWig visualization tracks
- peak calls (narrow/broad) with peak annotation and motif identification (ChIP-seq)
- differential gene expression results (RNA-seq)

## How to apply

Install Nextflow plus a container runtime (Docker per the paper; the README also supports Singularity), then clone the c-guzman/cipher-workflow-platform repository. Build a tab-separated CONFIG file with 5 columns for single-ended data (MergeID, SampleID, fastq path, InputID, Mark) or 6 columns for pair-ended data, never mixing single- and pair-ended samples in one file. Invoke the workflow as 'nextflow run cipher.nf' with the required flags --mode (e.g. chip or rna), --config, --fasta/--fa, --gtf, --lib, and --readLen, optionally adding --download_data to auto-fetch Ensembl/Gencode references; on a local machine set -qs <INT> to limit parallel processes and avoid out-of-memory failures. CIPHER then runs raw sequence evaluation, read trimming/adapter removal, read mapping and quality filtering, visualization track (bigWig) generation, and extensive QC, followed by assay-specific analysis (e.g. narrow/broad peak calling and motif identification for ChIP-seq, differential gene expression for RNA-seq). Rationale: CIPHER parallelizes across datasets — six ChIP-seq sets (~30 million reads each) completed in 7 h 23 min versus ~30 h for HiChIP — while needing only about six user flags; for cluster execution edit nextflow.config to set the executor (SGE, SLURM, LSF, PBS/Torque).

## Related tools

- **CIPHER** (The Nextflow workflow platform being executed; runs entire NGS pipelines from raw FASTQ to QC and downstream analysis) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language/engine that launches cipher.nf, manages task parallelization, and supports cluster executors (SGE, SLURM, LSF, PBS/Torque)) — https://www.nextflow.io/
- **Docker** (Container-based virtualization used to encapsulate dependencies for reproducible pipeline execution (paper); README also supports Singularity)
- **MultiQC** (Aggregates the extensive per-dataset quality control assessments produced by the pipeline)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config config.txt --fa hg38.fa --gtf genes.gtf --lib lib.txt --readLen 50
```

## Evaluation signals

- The run completes all pipeline stages for every sample in the CONFIG: raw sequence evaluation, trimming/adapter removal, mapping and quality filtering, visualization track generation, and QC assessment.
- Runtime is consistent with the paper's benchmark: six ChIP-seq datasets (~30 million reads each) finishing in 7 h 23 min on parallel execution.
- Only the six required flags (--mode, --config, --fasta/--fa, --gtf, --lib, --readLen) were needed, matching the ease-of-use comparison against MAP-RSeq/HiChIP.
- Outputs include the expected artifacts per assay: BAM files, bigWig tracks, peak/DGE results, and QC reports.
- On a local (non-cluster) machine, the -qs flag was set so the workflow does not abort from memory exhaustion during parallelization.

## Limitations

- CIPHER does not handle mixed single- and pair-ended data within one CONFIG file.
- CIPHER only runs entire workflows; individual tools cannot be selected as a subset.
- Running too many parallel processes locally without setting -qs causes the workflow to end abruptly because it runs out of memory.
- The current RNA-seq pipeline is largely optimized for two-type experimental designs (e.g. WT versus KO); no GWAS or de novo transcriptome assembly pipelines exist.
- Creating Singularity containers requires sudo access.

## Evidence

- [intro] By combining Nextflow - a powerful workflow language based on the Unix pipe concept: "By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept"
- [intro] Docker - a container-based virtualization technology: "Docker [13] - a container-based virtualization technology"
- [abstract] (1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
- [results] CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
- [readme] DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE. CIPHER DOES NOT HANDLE THIS USE-CASE YET.: [evidence span withheld — anchor status: external_doc]
- [readme] nextflow run cipher.nf -with-singularity <cipher.img> --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH>: [evidence span withheld — anchor status: external_doc]
- [readme] If not running on a cluster please set the -qs <INT> flag in order to control the number of processes that CIPHER parallelizes.: [evidence span withheld — anchor status: external_doc]
