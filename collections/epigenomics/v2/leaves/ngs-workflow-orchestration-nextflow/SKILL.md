---
name: ngs-workflow-orchestration-nextflow
description: Use when raw single- or pair-ended FASTQ datasets for common NGS assays
  must be batch-processed into trimmed reads, mapped BAMs, QC reports, and mode-specific
  outputs (e.g. MACS2 peaks or DE genes) in one reproducible pipeline.
when_to_use_negative:
- The task needs only a subset of tools or steps — the article notes CIPHER 'currently
  only runs entire workflows, but we are aware that individuals may prefer to use
  only a subset of tools'
- The RNA-seq design is multi-factor or time-series — CIPHER's RNA-seq pipeline 'is
  largely optimized for typical two-type experimental designs (e.g. WT versus KO)'
- The goal is GWAS or de novo transcriptome assembly, which CIPHER does not provide
  pipelines for
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0080
tools:
- name: CIPHER
  role: The workflow platform being orchestrated; defines the two-stage pipeline structure
    and mode dispatch
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language (Unix pipe concept) used to implement and parallelize the
    pipeline stages
  repo: https://www.nextflow.io/
- name: Docker
  role: Container technology pairing with Nextflow in the paper to make pipelines
    reproducible and ease installation
- name: Singularity
  role: 'Container runtime used in the README: create cipher.img, bootstrap from Singularity
    file, run with -with-singularity'
  repo: http://singularity.lbl.gov/index.html
- name: BBDuk
  role: Stage-1 adapter trimming and filtering of low-quality/short reads (defaults
    mapq < 20, length < 10)
- name: BBMap
  role: Default read mapper in stage 1; alternatives bwa, bowtie2, star, hisat2 selectable
    via --aligner
- name: FastQC
  role: Raw sequence evaluation at the start of stage 1
- name: MultiQC
  role: Aggregates quality control reports into pdf/html output
- name: MACS2
  role: Stage-2 ChIP-seq narrow peak calling with --qvalue 0.01 and --keep-dup all
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:62edd1595aeb6ee7c59c9f83b5984ad71d45af2a520a81fefe3cdd34aa47bf77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - BBDuk
  - BBMap
  - CIPHER
  - Docker
  - FastQC
  - MACS2
  - MultiQC
  - Nextflow
  - Singularity
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# ngs-workflow-orchestration-nextflow

## Summary

Orchestrate reproducible NGS pre-processing and dataset-type-specific analysis (ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq, GRO-seq) as a two-stage Nextflow workflow, following CIPHER's design of a fixed evaluation/trimming/mapping/QC stage followed by mode-dispatched downstream analysis.

## When to use

Use when raw single- or pair-ended FASTQ datasets for common NGS assays must be batch-processed into trimmed reads, mapped BAMs, QC reports, and mode-specific outputs (e.g. MACS2 peaks or DE genes) in one reproducible pipeline.

## When NOT to use

- The task needs only a subset of tools or steps — the article notes CIPHER 'currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools'
- The RNA-seq design is multi-factor or time-series — CIPHER's RNA-seq pipeline 'is largely optimized for typical two-type experimental designs (e.g. WT versus KO)'
- The goal is GWAS or de novo transcriptome assembly, which CIPHER does not provide pipelines for

## Inputs

- Single-ended or pair-ended FASTQ files (.fastq.gz) referenced by a tab-separated config file
- Config file with columns MergeID, SampleID, FASTQ path(s), InputID, Mark
- Reference genome FASTA (--fa) and GTF annotation (--gtf)
- Read length (--readLen) and library type (--lib, e.g. 's' for single-end)

## Outputs

- Gzipped trimmed FASTQ files
- Sorted BAM files with alignment statistics
- Visualization tracks (e.g. bigWig)
- Quality control reports (pdf/html) aggregated by MultiQC
- Mode-specific outputs such as MACS2 narrow peaks (--qvalue 0.01, --keep-dup all) and HOMER annotations for ChIP-seq, or differential expression tables for RNA-seq
- All artifacts written to './report' by default (override with --outdir)

## How to apply

Build a Nextflow entry script (e.g. main.nf/cipher.nf) whose required flags (--mode, --config, --fa, --gtf, --lib, --readLen) dispatch to one of the mode pipelines (chip, rna, gro, mnase, dnase, atac). Implement stage 1 as the fixed pre-processing chain: FastQC on raw reads, adapter trimming and filtering of low-quality/short reads with BBDuk (defaults mapq < 20, length < 10), then mapping with the default BBMap aligner (alternatives bwa, bowtie2, star, hisat2 via --aligner), followed by visualization track generation and QC. Implement stage 2 as mode-dependent analysis, e.g. MACS2 narrow peak calling with --qvalue 0.01 and --keep-dup all plus HOMER annotation for ChIP-seq, or differential gene expression for RNA-seq. Provide sample metadata via a tab-separated config file (5 columns single-end, 6 pair-end: MergeID, SampleID, path1, [path2,] InputID, Mark) and run inside containers (the paper pairs Nextflow with Docker; the README uses Singularity images built from a Singularity bootstrap file). Emit gzipped trimmed FASTQs, sorted BAMs with alignment statistics, and MultiQC-aggregated pdf/html reports into the default './report' output directory (override with --outdir).

## Related tools

- **CIPHER** (The workflow platform being orchestrated; defines the two-stage pipeline structure and mode dispatch) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language (Unix pipe concept) used to implement and parallelize the pipeline stages) — https://www.nextflow.io/
- **Docker** (Container technology pairing with Nextflow in the paper to make pipelines reproducible and ease installation)
- **Singularity** (Container runtime used in the README: create cipher.img, bootstrap from Singularity file, run with -with-singularity) — http://singularity.lbl.gov/index.html
- **BBDuk** (Stage-1 adapter trimming and filtering of low-quality/short reads (defaults mapq < 20, length < 10))
- **BBMap** (Default read mapper in stage 1; alternatives bwa, bowtie2, star, hisat2 selectable via --aligner)
- **FastQC** (Raw sequence evaluation at the start of stage 1)
- **MultiQC** (Aggregates quality control reports into pdf/html output)
- **MACS2** (Stage-2 ChIP-seq narrow peak calling with --qvalue 0.01 and --keep-dup all)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode chip --config config.txt --fa ref.fa --gtf ref.gtf --lib s --readLen 75
```

## Evaluation signals

- The run completes and './report' (or --outdir) contains gzipped trimmed FASTQs, sorted BAMs, and QC reports in pdf/html format
- QC reports are aggregated by MultiQC covering raw evaluation, trimming, mapping, and contamination checks
- For ChIP-seq mode, MACS2 narrow peaks are called with the documented thresholds (--qvalue 0.01, --keep-dup all) and annotated via HOMER
- Sample/control pairing is respected: InputID pairs sample and input files, and input files are marked with the keyword 'input' in the config
- Performance is plausible relative to the paper's benchmark: six ChIP-seq datasets (~30 million reads each) processed in 7 h 23 min

## Limitations

- CIPHER does not handle mixed single- and pair-ended data in the same config file
- CIPHER only runs entire workflows, not subsets of individual tools
- The RNA-seq pipeline is largely optimized for two-type designs (WT versus KO), lacking multi-design DEG and time-series analysis
- No pipelines for GWAS or de novo transcriptome assembly
- Running without containers requires manual installation of all pipeline software; on non-cluster runs, the -qs flag must limit parallel processes or the workflow can exhaust memory

## Evidence

- [other] CIPHER's pipelines proceed in two layers: a fixed pre-processing stage (evaluation, trimming, mapping, visualization, QC) followed by downstream analysis dispatched per dataset type: [evidence span withheld — anchor status: not_found]
- [abstract] (1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
- [intro] By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept: "By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept"
- [readme] nextflow run cipher.nf -with-singularity <cipher.img> --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH> [options]: [evidence span withheld — anchor status: external_doc]
- [results] CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min: "CIPHER processed all ChIP-seq datasets (~30 million reads per dataset) in 7 h and 23 min"
