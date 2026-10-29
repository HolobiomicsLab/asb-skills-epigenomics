---
name: differential-gene-expression-analysis
description: Use when raw RNA-seq FASTQ files and sample metadata need end-to-end
  processing (QC, STAR/BBMap alignment, featureCounts quantification, RUVSeq normalization)
  followed by DEG testing, or when a two-factor CIPHER expInfo schema must be extended
  to multi-condition or time-series contrasts.
when_to_use_negative:
- Do not mix single-ended and pair-ended data into the same CONFIG file — the README
  states CIPHER does not handle this use case yet
- Do not use when only a subset of tools is needed — CIPHER currently only runs entire
  workflows, not subsets of tools
- Do not rely on CIPHER's default expInfo schema alone for designs with more than
  two factor levels or time-series contrasts
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CIPHER
  role: Nextflow-based workflow platform executing the RNA-seq pipeline end to end
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: DESeq2
  role: Fits multi-factor design matrices (e.g. ~ time + condition + time:condition)
    for contrasts beyond CIPHER's two-condition schema
- name: edgeR
  role: Parallel GLM-based differential expression testing as a second model design
- name: featureCounts
  role: Gene-level count quantification from STAR/BBMap alignments using the supplied
    GTF annotation
- name: RUVSeq
  role: Unwanted-variation assessment and normalization of the count matrix
- name: STAR
  role: Splice-aware read mapper selected via CIPHER's --aligner flag (BBMap default
    retained as fallback)
- name: MultiQC
  role: Aggregates FastQC and QoRTs QC results into a single HTML report
- name: Nextflow
  role: Workflow language driving cipher.nf execution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:d1eaedef09e18c39aa063c872503558316d39d9e0a8d859992abac7f96a2a17f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - DESeq2
  - MultiQC
  - Nextflow
  - QoRTs
  - RUVSeq
  - STAR
  - edgeR
  - featureCounts
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# differential-gene-expression-analysis

## Summary

CIPHER's RNA-seq workflow performs differential gene expression analysis as part of a Nextflow-based pipeline spanning raw sequence evaluation, read mapping, and QC. It is optimized for two-type experimental designs (e.g. WT versus KO), so multi-factor and time-series designs require constructing custom DESeq2/edgeR GLM design matrices outside CIPHER's default schema.

## When to use

Use when raw RNA-seq FASTQ files and sample metadata need end-to-end processing (QC, STAR/BBMap alignment, featureCounts quantification, RUVSeq normalization) followed by DEG testing, or when a two-factor CIPHER expInfo schema must be extended to multi-condition or time-series contrasts.

## When NOT to use

- Do not mix single-ended and pair-ended data into the same CONFIG file — the README states CIPHER does not handle this use case yet
- Do not use when only a subset of tools is needed — CIPHER currently only runs entire workflows, not subsets of tools
- Do not rely on CIPHER's default expInfo schema alone for designs with more than two factor levels or time-series contrasts

## Inputs

- Single-ended or pair-ended RNA-seq FASTQ files
- Tab-separated CONFIG file with columns MergeID, SampleID, Path (R1), Path 2 (R2, pair-ended only), InputID, Mark
- Experimental config file (sample, condition) matching CIPHER's expInfo schema
- Reference genome FASTA and GTF annotation (e.g. Ensembl/Gencode, downloadable via --download_data)

## Outputs

- Gene-level count matrix from featureCounts, RUVSeq-normalized
- Per-gene DEG results table with log2 fold-change, p-value, and FDR per contrast
- DEG table augmented with model-design metadata (design formula, factor levels, tested coefficient)
- MultiQC-aggregated HTML quality control report

## How to apply

Format a tab-separated CIPHER CONFIG file (MergeID, SampleID, FASTQ path, InputID, Mark) plus an experimental config mapping samples to conditions, then run the cipher.nf Nextflow workflow in a Singularity container with --gtf and --lib annotation arguments; use --aligner STAR for splice-aware mapping (BBMap is the default fallback). QC the runs with FastQC/QoRTs aggregated by MultiQC, then normalize the featureCounts matrix with RUVSeq to address unwanted variation. Because CIPHER's expInfo is optimized for two-type designs (e.g. WT versus KO), express multi-factor models by building a DESeq2 design such as ~ condition + batch or ~ time + condition + time:condition, and in parallel an edgeR GLM design, fitting contrasts CIPHER's default schema cannot express. Write a combined DEG table with per-gene log2 fold-change, p-value, and FDR per contrast, augmented with design-metadata columns (design formula, factor levels, tested coefficient).

## Related tools

- **CIPHER** (Nextflow-based workflow platform executing the RNA-seq pipeline end to end) — https://github.com/c-guzman/cipher-workflow-platform
- **DESeq2** (Fits multi-factor design matrices (e.g. ~ time + condition + time:condition) for contrasts beyond CIPHER's two-condition schema)
- **edgeR** (Parallel GLM-based differential expression testing as a second model design)
- **featureCounts** (Gene-level count quantification from STAR/BBMap alignments using the supplied GTF annotation)
- **RUVSeq** (Unwanted-variation assessment and normalization of the count matrix)
- **STAR** (Splice-aware read mapper selected via CIPHER's --aligner flag (BBMap default retained as fallback))
- **MultiQC** (Aggregates FastQC and QoRTs QC results into a single HTML report)
- **Nextflow** (Workflow language driving cipher.nf execution)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode <MODE> --config <CONFIG> --fa <FASTA> --gtf <GTF> --lib <LIB> --readLen <LENGTH>
```

## Evaluation signals

- Output DEG table exists and contains log2 fold-change, p-value, and FDR fields for all quantified genes
- Recorded design formula supports more than two factor levels, lifting CIPHER's two-condition expInfo restriction
- Per-contrast design metadata columns (design formula, factor levels, tested coefficient) are populated
- MultiQC HTML report covers QC from FastQC and QoRTs across all samples
- Count matrix passed RUVSeq unwanted-variation assessment before model fitting

## Limitations

- CIPHER's RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO); multi-design and time-series DEG analysis is missing by default
- CIPHER does not handle mixed single-ended and pair-ended data in the same CONFIG file
- CIPHER only runs entire workflows, not subsets of tools; users preferring a subset must run tools individually
- Running with too many parallelized processes can exhaust memory and abruptly end the workflow — set the -qs flag when not on a cluster

## Evidence

- [discussion] CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO): "CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO)"
- [abstract] differential gene expression analysis (RNA-seq): "differential gene expression analysis for RNA-seq"
- [readme] README config column definitions: [evidence span withheld — anchor status: not_found]
- [readme] README run command: [evidence span withheld — anchor status: external_doc]
