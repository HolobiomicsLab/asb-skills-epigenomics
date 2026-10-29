---
name: rnaseq-read-quantification
description: Use when processing bulk RNA-seq FASTQ files (single- or pair-ended)
  through alignment, gene-level counting, and normalization to produce count matrices
  and DEG tables, or when assessing whether a design beyond two conditions fits CIPHER's
  expInfo schema.
when_to_use_negative:
- The experiment requires multi-factor designs or time-series differential expression
  beyond two conditions — CIPHER's current RNA-seq pipeline is optimized for typical
  two-type designs (e.g. WT versus KO), so the DEG step must be replaced/extended
  manually
- Input is already a gene count matrix rather than raw FASTQ — the alignment and quantification
  stages are unnecessary; go directly to RUVSeq/DESeq2
- You want to run only a subset of tools (e.g. only trimming or only mapping) — CIPHER
  currently only runs entire workflows, not subsets of tools
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CIPHER
  role: Workflow platform orchestrating the RNA-seq processing pipeline from config
    file to DEG analysis
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language backing CIPHER; run entry point is `nextflow run cipher.nf`
- name: STAR
  role: Splice-aware aligner selectable via CIPHER's --aligner flag for RNA-seq read
    mapping
- name: BBMap
  role: Default DNA mapper retained as fallback for alignment
- name: featureCounts
  role: Subread-suite tool for gene-level read counting against the supplied GTF annotation
- name: RUVSeq
  role: Unwanted-variation assessment and normalization of the gene count matrix before
    DEG testing
- name: DESeq2
  role: Differential expression modeling; design matrix (e.g. ~ condition + batch
    or ~ time + condition + time:condition) for contrasts CIPHER's two-condition schema
    cannot express
- name: edgeR
  role: GLM design used in parallel to DESeq2 for multi-factor contrasts
- name: MultiQC
  role: Aggregates FastQC/QoRTs QC results into a single HTML report
- name: Singularity
  role: Containerization for reproducible execution (cipher.img created via `sudo
    singularity create` and bootstrapped)
  repo: http://singularity.lbl.gov/index.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:d4722abd99ea0eddf329ace7fee5eb580b82f74aacfbc13d1546c849348eaf1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - BBMap
  - CIPHER
  - DESeq2
  - MultiQC
  - Nextflow
  - QoRTs
  - RUVSeq
  - STAR
  - Singularity
  - edgeR
  - featureCounts
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# rnaseq-read-quantification

## Summary

CIPHER's RNA-seq workflow quantifies gene-level expression from FASTQ files through splice-aware alignment and counting, followed by unwanted-variation correction, to support differential gene expression analysis. The current implementation is optimized for two-type experimental designs (e.g. WT versus KO), so multi-factor or time-series designs require manual extension with DESeq2/edgeR GLM models.

## When to use

Use when processing bulk RNA-seq FASTQ files (single- or pair-ended) through alignment, gene-level counting, and normalization to produce count matrices and DEG tables, or when assessing whether a design beyond two conditions fits CIPHER's expInfo schema.

## When NOT to use

- The experiment requires multi-factor designs or time-series differential expression beyond two conditions — CIPHER's current RNA-seq pipeline is optimized for typical two-type designs (e.g. WT versus KO), so the DEG step must be replaced/extended manually
- Input is already a gene count matrix rather than raw FASTQ — the alignment and quantification stages are unnecessary; go directly to RUVSeq/DESeq2
- You want to run only a subset of tools (e.g. only trimming or only mapping) — CIPHER currently only runs entire workflows, not subsets of tools

## Inputs

- Single-ended FASTQ files (FASTQ R1) listed in a 5-column tab-separated config file (MergeID, SampleID, Path, InputID, Mark)
- Pair-ended FASTQ files (R1/R2) listed in a 6-column tab-separated config file
- Experimental config file (sample, condition) matching CIPHER's expInfo schema
- Reference genome FASTA (via --fa)
- Gene annotation GTF (via --gtf)
- Library type (--lib) and read length (--readLen)

## Outputs

- Per-sample gene-level count matrix from featureCounts
- MultiQC-aggregated HTML QC report covering raw-read evaluation, trimming, mapping, and quality filtering
- RUVSeq-normalized count matrix
- Combined DEG results table with per-gene log2 fold-change, p-value, and FDR per contrast (optionally augmented with design-formula metadata columns)

## How to apply

Format a tab-separated CIPHER config file (5 columns for single-ended, 6 for pair-ended data: MergeID, SampleID, Path1, optional Path2, InputID, Mark; use '-' for InputID in RNA-seq). Run the workflow with --mode for RNA-seq, supplying --fa FASTA, --gtf GTF annotation, --lib library type, and --readLen; optionally override the default BBMap mapper with the splice-aware aligner STAR via --aligner. CIPHER performs raw sequence evaluation, trimming/adapter removal, read mapping and quality filtering, visualization track generation, and extensive quality control (FastQC/QoRTs aggregated by MultiQC), then counts gene-level reads with featureCounts (Subread). Apply RUVSeq for unwanted-variation assessment and normalization of the count matrix before DEG testing in DESeq2. For designs with more than two factor levels (e.g. ~ time + condition + time:condition), construct the design matrix manually in DESeq2 or an edgeR GLM, since CIPHER's expInfo schema only expresses two-type designs.

## Related tools

- **CIPHER** (Workflow platform orchestrating the RNA-seq processing pipeline from config file to DEG analysis) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language backing CIPHER; run entry point is `nextflow run cipher.nf`)
- **STAR** (Splice-aware aligner selectable via CIPHER's --aligner flag for RNA-seq read mapping)
- **BBMap** (Default DNA mapper retained as fallback for alignment)
- **featureCounts** (Subread-suite tool for gene-level read counting against the supplied GTF annotation)
- **RUVSeq** (Unwanted-variation assessment and normalization of the gene count matrix before DEG testing)
- **DESeq2** (Differential expression modeling; design matrix (e.g. ~ condition + batch or ~ time + condition + time:condition) for contrasts CIPHER's two-condition schema cannot express)
- **edgeR** (GLM design used in parallel to DESeq2 for multi-factor contrasts)
- **MultiQC** (Aggregates FastQC/QoRTs QC results into a single HTML report)
- **Singularity** (Containerization for reproducible execution (cipher.img created via `sudo singularity create` and bootstrapped)) — http://singularity.lbl.gov/index.html

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode RNA-seq --config config.tsv --fa hg38.fa --gtf gencode.gtf --lib unstranded --readLen 100
```

## Evaluation signals

- Output DEG table exists and contains log2 fold-change, p-value, and FDR fields for all quantified genes
- If extended to multi-factor designs, the recorded design formula supports more than two factor levels, lifting the two-condition restriction of CIPHER's expInfo
- Design-metadata columns (design formula, factor levels, tested coefficient) are present in the combined results table
- Config file follows CIPHER's schema exactly: 5 columns single-ended / 6 columns pair-ended, with no mixing of single and pair-ended data in one file (CIPHER does not handle this use case)
- MultiQC HTML report is produced covering raw evaluation, trimming, mapping, and quality-filtering stages

## Limitations

- CIPHER's RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO); multi-design and time-series DEG analysis capability is missing
- Single-ended and pair-ended data must not be mixed in the same config file — CIPHER does not handle this use case yet
- CIPHER currently only runs entire workflows; it does not support running subsets of tools
- No pipelines for GWAS or de novo transcriptome assembly are available
- When running without Singularity, all pipeline software must be installed manually (tools are listed inside cipher.nf); on a local machine, -qs <INT> must limit parallel processes or the workflow may end abruptly from memory exhaustion

## Evidence

- [discussion] CIPHER's current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO): [evidence span withheld — anchor status: not_found]
- [abstract] differential gene expression analysis for RNA-seq: [evidence span withheld — anchor status: not_found]
- [abstract] (1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
- [readme] DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE. CIPHER DOES NOT HANDLE THIS USE-CASE YET.: [evidence span withheld — anchor status: external_doc]
- [readme] nextflow run cipher.nf -with-singularity <cipher.img> --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH> [options]: [evidence span withheld — anchor status: external_doc]
- [discussion] CIPHER currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools: "CIPHER currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools to complete certain tasks"
