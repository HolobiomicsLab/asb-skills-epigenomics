---
name: two-stage-pipeline-dispatch
description: Use when building or reproducing a Nextflow-based NGS pipeline that must
  share one fixed pre-processing stage (FastQC → trimming → alignment → QC) across
  dataset types and then dispatch mode-specific downstream analyses.
when_to_use_negative:
- You need to run only a subset of tools rather than an entire workflow — the article
  notes 'CIPHER currently only runs entire workflows, but we are aware that individuals
  may prefer to use only a subset of tools'
- Your config file mixes single- and pair-ended samples — the README warns 'DO NOT
  MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE'
- You need multi-design or time-series RNA-seq differential analysis — the RNA-seq
  pipeline is largely optimized for typical two-type designs (e.g. WT versus KO)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow language implementing the entry point and per-mode dispatch of stage-1
    and stage-2 branches
  repo: https://www.nextflow.io/
- name: FastQC
  role: Raw sequence evaluation at the start of the fixed pre-processing stage
- name: BBDuk
  role: Adapter trimming and filtering of low-quality/short reads (mapq < 20, length
    < 10 defaults)
- name: BBMap
  role: Default read aligner in the mapping/quality-filtering step; alternatives selectable
    via --aligner
- name: MultiQC
  role: Aggregates per-sample QC reports into a single PDF/HTML output
- name: MACS2
  role: Mode-dependent ChIP-seq narrow peak calling (qvalue 0.01, keep-dup all)
- name: Singularity
  role: Container virtualization used to make the dispatched workflow reproducible
  repo: http://singularity.lbl.gov/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:9447db3436fdcb70a36039692bdad5cc294048bcf7f16a18436284acb664b4af
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
  - Singularity
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# two-stage-pipeline-dispatch

## Summary

Reconstructs CIPHER's two-layer NGS workflow structure: a fixed pre-processing stage (raw sequence evaluation, trimming/adapter removal, mapping/quality filtering, visualization tracks, QC) followed by a mode-dependent downstream analysis dispatched per dataset type (ChIP-seq, RNA-seq, MNase-seq, DNase-seq, GRO-seq, ATAC-seq).

## When to use

Use when building or reproducing a Nextflow-based NGS pipeline that must share one fixed pre-processing stage (FastQC → trimming → alignment → QC) across dataset types and then dispatch mode-specific downstream analyses.

## When NOT to use

- You need to run only a subset of tools rather than an entire workflow — the article notes 'CIPHER currently only runs entire workflows, but we are aware that individuals may prefer to use only a subset of tools'
- Your config file mixes single- and pair-ended samples — the README warns 'DO NOT MIX AND MATCH SINGLE AND PAIR ENDED DATA INTO THE SAME CONFIG FILE'
- You need multi-design or time-series RNA-seq differential analysis — the RNA-seq pipeline is largely optimized for typical two-type designs (e.g. WT versus KO)

## Inputs

- Tab-separated CONFIG file (5 columns single-end / 6 columns pair-end: MergeID, SampleID, FASTQ path 1, [FASTQ path 2], InputID, Mark)
- Single- or pair-ended gzipped FASTQ files
- Reference genome FASTA (--fa/--fasta)
- Gene annotation GTF (--gtf)
- Library type flag (--lib s|p) and read length (--readLen)

## Outputs

- Gzipped trimmed FASTQ files
- Sorted BAM files with alignment statistics
- Visualization tracks
- QC reports (PDF/HTML) aggregated by MultiQC
- Mode-specific downstream results (e.g. MACS2 narrow peaks and HOMER motif annotations for ChIP-seq; differential gene expression table for RNA-seq)
- All artifacts in './report' output directory (override with --outdir)

## How to apply

Create a Nextflow entry point (main.nf / cipher.nf) that parses flags (--mode, --config, --fasta/--fa, --gtf, --lib, --readLen) and dispatches to one of six pipelines (chip, rna, gro, mnase, dnase, atac). Implement the stage-1 branch identically for all modes: FastQC on raw reads, BBDuk trimming/adapter removal and filtering (defaults mapq < 20, length < 10), then BBMap alignment (alternatives bwa, bowtie2, star, hisat2 via --aligner). Implement the stage-2 branch as the mode-dependent analysis, e.g. MACS2 narrow peak calling with --qvalue 0.01 and --keep-dup all plus HOMER annotation for ChIP-seq, or differential gene expression for RNA-seq. Emit gzipped trimmed FASTQs, sorted BAMs with alignment statistics, and MultiQC-aggregated PDF/HTML QC reports into './report' (override with --outdir). The config file is tab-separated with 5 columns (single-end) or 6 columns (pair-end): MergeID, SampleID, FASTQ path(s), InputID, and Mark ('input' for control files); never mix single- and pair-ended rows in one config.

## Related tools

- **Nextflow** (Workflow language implementing the entry point and per-mode dispatch of stage-1 and stage-2 branches) — https://www.nextflow.io/
- **FastQC** (Raw sequence evaluation at the start of the fixed pre-processing stage)
- **BBDuk** (Adapter trimming and filtering of low-quality/short reads (mapq < 20, length < 10 defaults))
- **BBMap** (Default read aligner in the mapping/quality-filtering step; alternatives selectable via --aligner)
- **MultiQC** (Aggregates per-sample QC reports into a single PDF/HTML output)
- **MACS2** (Mode-dependent ChIP-seq narrow peak calling (qvalue 0.01, keep-dup all))
- **Singularity** (Container virtualization used to make the dispatched workflow reproducible) — http://singularity.lbl.gov/index.html

## Examples

```
nextflow run main.nf --mode chip --config config.txt --fasta ref.fa --gtf ref.gtf --lib s --readLen 75
```

## Evaluation signals

- The run completes and './report' contains gzipped trimmed FASTQs, sorted BAMs, and QC reports in the expected formats
- QC artifacts are aggregated by MultiQC into a single PDF/HTML report
- Stage-1 output is identical in structure across modes (same trim/filter/map steps); only stage-2 differs by --mode
- Peak calling output for ChIP-seq uses MACS2 --qvalue 0.01 and --keep-dup all as specified
- Config file column count matches library type (5 columns single-end, 6 columns pair-end) with no mixed rows

## Limitations

- CIPHER currently only runs entire workflows, not subsets of tools
- The RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO), lacking multi-design DEG and time-series analysis
- CIPHER does not handle mixed single-/pair-ended data within one config file
- When not running on a cluster, the -qs <INT> flag must control parallelism or the workflow can run out of memory and end abruptly

## Evidence

- [other] CIPHER's pipelines proceed in two layers: "(1) raw sequence evaluation, (2) read trimming and adapter removal, (3) read mapping and quality filtering, (4) visualization track generation, and (5) extensive quality control assessment"
- [abstract] Stage-2 dispatch per dataset type: "narrow and broad peak calling, peak annotation, and motif identification for ChIP-seq"
- [readme] CLI entry point and flags: [evidence span withheld — anchor status: external_doc]
- [readme] Config file constraint: [evidence span withheld — anchor status: external_doc]
- [abstract] Downstream analysis scope: "CIPHER contains an "analysis" mode that completes complex bioinformatics tasks such as enhancer discovery and provides functions to integrate various datasets together."
