---
name: ngs-read-quality-control
description: Use when raw sequencing reads (FASTQ) enter a Watchdog RNA-seq workflow
  and their quality must be checked before mapping against the reference genome.
when_to_use_negative:
- The input is already a gene count table or DE result table — the QC module operates
  on raw input reads, not count matrices.
- You need quality scores, adapters, or trimming performed — the article describes
  QC checking as part of the workflow, not trimming/cleaning of reads.
- The analysis goal is a custom workflow built from scratch — the shipped QC step
  belongs to the example RNA-seq workflow requiring bwa and ContextMap2.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Watchdog
  role: workflow management system hosting the QC module that performs FastQC-style
    read quality checks as the first step of the shipped RNA-seq workflow
  repo: github.com/klugem/watchdog
- name: Watchdog modules
  role: community module repository supplying the QC module definition used in the
    workflow
  repo: https://github.com/watchdog-wms/watchdog-wms-modules
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:05e9fd01a108263cc44b4bfc399601c87fe7be9238f6982db303059c7c7d532a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Watchdog
  - Watchdog modules
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ngs-read-quality-control

## Summary

Run the Watchdog QC module (FastQC-style) as the first step of the shipped RNA-seq workflow to check the quality of raw input reads before mapping with ContextMap 2 and quantifying with featureCounts.

## When to use

Use when raw sequencing reads (FASTQ) enter a Watchdog RNA-seq workflow and their quality must be checked before mapping against the reference genome.

## When NOT to use

- The input is already a gene count table or DE result table — the QC module operates on raw input reads, not count matrices.
- You need quality scores, adapters, or trimming performed — the article describes QC checking as part of the workflow, not trimming/cleaning of reads.
- The analysis goal is a custom workflow built from scratch — the shipped QC step belongs to the example RNA-seq workflow requiring bwa and ContextMap2.

## Inputs

- raw RNA-seq input reads (FASTQ) from the shipped example dataset
- Watchdog workflow XML file (workflow2_differential_gene_expression.xml)

## Outputs

- FastQC-style read QC reports in the workflow output folder
- validated input reads passing QC before ContextMap 2 mapping

## How to apply

Configure the shipped example workflows with ./helper_scripts/configureExamples.sh and launch workflow2_differential_gene_expression.xml via ./watchdog.sh -x so the QC module runs as the first module. The Watchdog QC module performs FastQC-style read quality checks on the input reads, producing QC reports alongside the downstream steps (ContextMap 2 mapping, samtools BAM conversion, featureCounts quantification, RSeQC mapping QC). Treat the QC step as a gate: Watchdog's customizable error-detection system inspects module success, so a failing QC step halts the workflow before resources are spent on mapping and differential expression testing (limma, edgeR, DESeq, DESeq2 via the DETest module). Results are collected in the workflow output folder and can be summarized with reportGenerator.jar.

## Related tools

- **Watchdog** (workflow management system hosting the QC module that performs FastQC-style read quality checks as the first step of the shipped RNA-seq workflow) — github.com/klugem/watchdog
- **Watchdog modules** (community module repository supplying the QC module definition used in the workflow) — https://github.com/watchdog-wms/watchdog-wms-modules

## Examples

```
./helper_scripts/configureExamples.sh -i /path/to/install/folder/of/watchdog && ./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml
```

## Evaluation signals

- The QC module completes without triggering Watchdog's error-detection system, so downstream modules (ContextMap 2, samtools, featureCounts) start.
- A QC report file is present in the workflow output folder after execution.
- QC runs before read mapping — verify in the workflow XML/execution log that quality checks precede ContextMap 2 mapping and featureCounts quantification.
- The workflow finishes producing the gene count table and per-method DE result tables (limma, edgeR, DESeq, DESeq2), confirming the QC step did not block the pipeline.
- reportGenerator.jar can summarize the executed workflow steps including the QC module run.

## Limitations

- The shipped RNA-seq workflow requires bwa and ContextMap2 to be installed and configured separately; QC alone is not sufficient to run the workflow.
- Watchdog itself is implemented in Java (JDK 11 or higher per the README), so the QC workflow cannot run without the Java runtime and, for GUI tooling, the JavaFX SDK.
- No specific quality-score thresholds are stated in the article or README for the QC module; success/failure is governed by Watchdog's customizable error-detection system.

## Evidence

- [intro] e.g read quality control, mapping against the genome, counting of reads for gene features: "e.g read quality control, mapping against the genome, counting of reads for gene features"
- [abstract] comprehensive RNA-seq workflow provided with the software: "a comprehensive workflow and modules for the analysis of RNA-seq experiments were implemented and are provided with the software"
- [readme] workflow2 example requiring bwa and ContextMap2: "workflow2_differential_gene_expression.xml - workflow performing a differential gene expression analysis on an example data set (needs bwa and ContextMap2 to be installed and configured)"
- [intro] case study modules for read quality checks: "As a case study on how Watchdog can be applied, modules for read quality checks, read mapping, gene expression quantification and differential gene expression analysis were implemented"
