---
name: count-matrix-normalization
description: Use when RNA-seq FASTQ files plus a (MergeID, ID, Path) config and expInfo-style
  condition table must be converted into normalized gene counts and DEG tables, especially
  for multi-factor or time-series designs.
when_to_use_negative:
- The experiment is a simple two-type design (e.g. WT versus KO) fully expressible
  in CIPHER's current expInfo schema and DEG output
- The input is single-end and paired-end RNA-seq data mixed in the same CONFIG file
  — CIPHER explicitly does not handle this use case
- The goal is de novo transcriptome assembly or GWAS, which CIPHER's discussion notes
  are planned but not implemented
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: CIPHER
  role: Workflow platform orchestrating the RNA-seq pre-processing pipeline (trimming,
    mapping, QC) invoked via nextflow run cipher.nf
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: STAR
  role: Splice-aware read aligner selected via CIPHER's --aligner flag; BBMap remains
    the default fallback
- name: featureCounts
  role: Subread-suite tool used to quantify gene-level counts against the supplied
    GTF annotation
- name: RUVSeq
  role: Unwanted-variation assessment and normalization of the featureCounts count
    matrix
- name: DESeq2
  role: Fits multi-factor design matrices (e.g. ~ condition + batch, ~ time + condition
    + time:condition) for differential expression contrasts
- name: edgeR
  role: Parallel GLM design fitted alongside DESeq2 for robustness of contrast testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:fa134aab88381457e4dbc4f06edc7decb64332a8f459de190033b59bb38c3839
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - DESeq2
  - QoRTs
  - RUVSeq
  - STAR
  - edgeR
  - featureCounts
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# count-matrix-normalization

## Summary

Turn aligned RNA-seq reads into a gene-level count matrix, apply unwanted-variation normalization (RUVSeq), and fit differential expression models (DESeq2/edgeR GLM) with design matrices that go beyond CIPHER's built-in two-condition schema.

## When to use

Use when RNA-seq FASTQ files plus a (MergeID, ID, Path) config and expInfo-style condition table must be converted into normalized gene counts and DEG tables, especially for multi-factor or time-series designs.

## When NOT to use

- The experiment is a simple two-type design (e.g. WT versus KO) fully expressible in CIPHER's current expInfo schema and DEG output
- The input is single-end and paired-end RNA-seq data mixed in the same CONFIG file — CIPHER explicitly does not handle this use case
- The goal is de novo transcriptome assembly or GWAS, which CIPHER's discussion notes are planned but not implemented

## Inputs

- Raw RNA-seq FASTQ files (single- or pair-ended) with sample metadata from a repository such as GEO/SRA
- Tab-separated CONFIG file with 5 columns (single-end) or 6 columns (paired-end): MergeID, SampleID, Path 1, optional Path 2, InputID, Mark
- Experimental config file (sample, condition) in CIPHER's expInfo schema
- Reference genome FASTA and GTF annotation (e.g. Ensembl/Gencode, fetched via CIPHER's --download_data flag)

## Outputs

- Gene-level raw count matrix (featureCounts output on the supplied GTF)
- RUVSeq-normalized count matrix with unwanted-variation assessment
- Combined DEG results table with per-gene log2 fold-change, p-value, and FDR for each contrast, plus design-formula metadata columns (design formula, factor levels, tested coefficient)

## How to apply

Build a tab-separated CONFIG file (MergeID, SampleID, FASTQ path, plus R2 path for paired-end) and an experimental config (sample, condition) matching CIPHER's expInfo schema; run CIPHER's RNA-seq workflow via `nextflow run cipher.nf` with `--gtf` and `--lib` set, choosing the splice-aware aligner STAR through the `--aligner` flag (BBMap is the default fallback). Quantify gene-level counts with featureCounts from the Subread suite against the supplied GTF, then apply RUVSeq to the count matrix for unwanted-variation assessment and normalization. For differential testing, construct a DESeq2 design matrix (e.g. `~ condition + batch` or `~ time + condition + time:condition`) and in parallel an edgeR GLM design, then test contrasts CIPHER's native two-type schema cannot express. Emit a combined DEG table with per-gene log2 fold-change, p-value, and FDR per contrast, augmented with design-metadata columns (design formula, factor levels, tested coefficient).

## Related tools

- **CIPHER** (Workflow platform orchestrating the RNA-seq pre-processing pipeline (trimming, mapping, QC) invoked via nextflow run cipher.nf) — https://github.com/c-guzman/cipher-workflow-platform
- **STAR** (Splice-aware read aligner selected via CIPHER's --aligner flag; BBMap remains the default fallback)
- **featureCounts** (Subread-suite tool used to quantify gene-level counts against the supplied GTF annotation)
- **RUVSeq** (Unwanted-variation assessment and normalization of the featureCounts count matrix)
- **DESeq2** (Fits multi-factor design matrices (e.g. ~ condition + batch, ~ time + condition + time:condition) for differential expression contrasts)
- **edgeR** (Parallel GLM design fitted alongside DESeq2 for robustness of contrast testing)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode RNA-seq --config config.tsv --fa genome.fa --gtf genes.gtf --lib LIB --readLen 100 --aligner STAR
```

## Evaluation signals

- The output DEG table exists and contains log2 fold-change, p-value, and FDR columns for all quantified genes
- The design-formula metadata column supports more than two factor levels, confirming CIPHER's two-condition restriction is lifted
- Each contrast row records its tested coefficient and factor levels, so any FDR value is traceable to a specific model design
- MultiQC report (aggregating FastQC and QoRTs) shows raw-read QC passed before mapping with STAR/BBMap and counting with featureCounts
- The count matrix has been RUVSeq-normalized, so fold-changes and p-values are not driven by unwanted variation

## Limitations

- CIPHER's current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO); multi-design and time-series DEG analysis requires extending the pipeline with external DESeq2/edgeR designs
- CIPHER does not handle mixing single-end and pair-ended data in the same CONFIG file
- CIPHER only runs entire workflows, not subsets of tools, and cannot be run without Singularity containers unless all pipeline software is installed manually

## Evidence

- [discussion] CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO): "CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO)"
- [abstract] differential gene expression analysis for RNA-seq: "differential gene expression analysis for RNA-seq"
- [readme] Config files are tab separated text files with 5 columns for single-ended data and 6 columns for pair ended data.: [evidence span withheld — anchor status: external_doc]
- [readme] DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE. CIPHER DOES NOT HANDLE THIS USE-CASE YET.: [evidence span withheld — anchor status: external_doc]
- [methods] CIPHER incorporates a number of quality control tools to identify potential biases, contaminations, and errors in NGS datasets: "CIPHER incorporates a number of quality control tools to identify potential biases, contaminations, and errors in NGS datasets"
