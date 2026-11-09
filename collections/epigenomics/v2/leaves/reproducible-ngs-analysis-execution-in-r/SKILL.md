---
name: reproducible-ngs-analysis-execution-in-r
description: Use when the analysis goal is to rapidly run or test a complete NGS workflow
  in R/Bioconductor — e.g. reproducing a template RNA-Seq analysis from FASTQ inputs
  through reporting — with logged, reproducible execution.
when_to_use_negative:
- The task is a single specialized analysis step outside the pre-configured NGS templates
  (RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq) and does not need workflow management or
  reporting
- The analysis cannot be expressed as R or command-line steps orchestrated by an R/Bioconductor
  package, or the environment cannot meet version requirements (R >= 3.2, Bioconductor
  >= 3.2)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: systemPipeR
  role: Core R/Bioconductor package providing genWorkenvir/SPRproject, importWF, runWF,
    renderReport, and renderLogs for workflow generation, execution, and reporting
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Data package supplying pre-configured workflow templates (e.g. the RNA-Seq
    template) for rapid end-to-end runs
  repo: https://github.com/tgirke/systemPipeR
- name: R
  role: Programming language in which workflows are built and executed
- name: Bioconductor
  role: Distribution environment for systemPipeR and affiliated NGS tools
  repo: http://bioconductor.org/packages/devel/systemPipeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:b488cfc6d2ebfb973908d3696a5d110c1d5906d07194d146cf8d6c5ac0bcef1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reproducible-ngs-analysis-execution-in-r

## Summary

Skill for executing reproducible, end-to-end NGS analysis workflows (e.g. pre-configured RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq templates) with systemPipeR in R/Bioconductor, including automated scientific report generation.

## When to use

Use when the analysis goal is to rapidly run or test a complete NGS workflow in R/Bioconductor — e.g. reproducing a template RNA-Seq analysis from FASTQ inputs through reporting — with logged, reproducible execution.

## When NOT to use

- The task is a single specialized analysis step outside the pre-configured NGS templates (RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq) and does not need workflow management or reporting
- The analysis cannot be expressed as R or command-line steps orchestrated by an R/Bioconductor package, or the environment cannot meet version requirements (R >= 3.2, Bioconductor >= 3.2)

## Inputs

- Workflow template R Markdown file (e.g. systemPipeRNAseq.Rmd)
- Parameter files (e.g. targets/sample annotation files) pre-populated in the spr_project directory
- Raw sequencing data (e.g. FASTQ reads for RNA-Seq)
- R/Bioconductor session with systemPipeR (and systemPipeRdata for templates) installed

## Outputs

- Executed workflow steps with project logging (SPRproject log/environment)
- Rendered scientific report (systemPipeRNAseq.html)
- Workflow run diagnostics/logs (renderLogs output)

## How to apply

Generate the working environment with genWorkenvir(work = "rnaseq"), which creates a spr_project directory pre-populated with parameter files and the systemPipeRNAseq.Rmd template. Initialize the workflow container with SPRproject() to set up project logging, then import the R Markdown chunks as executable steps via importWF("systemPipeRNAseq.Rmd"). Execute the full workflow with runWF(), which can run both R functions and external command-line software on local machines or HPC clusters. Finally, renderReport() produces the HTML scientific report and renderLogs() exposes run diagnostics; the rationale is that this four-command interface (genWorkenvir/SPRproject, importWF, runWF, renderReport/renderLogs) standardizes generation, step import, execution, and reporting of NGS analyses.

## Related tools

- **systemPipeR** (Core R/Bioconductor package providing genWorkenvir/SPRproject, importWF, runWF, renderReport, and renderLogs for workflow generation, execution, and reporting) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Data package supplying pre-configured workflow templates (e.g. the RNA-Seq template) for rapid end-to-end runs) — https://github.com/tgirke/systemPipeR
- **R** (Programming language in which workflows are built and executed)
- **Bioconductor** (Distribution environment for systemPipeR and affiliated NGS tools) — http://bioconductor.org/packages/devel/systemPipeR

## Examples

```
library(systemPipeR); genWorkenvir(work = "rnaseq"); SPRproject(); importWF("systemPipeRNAseq.Rmd"); runWF(); renderReport(); renderLogs()
```

## Evaluation signals

- runWF() completes all imported workflow steps without errors
- The HTML report file systemPipeRNAseq.html is rendered and exists on disk
- renderLogs() produces run diagnostics showing each step executed in order without failures
- The spr_project directory contains the expected logging/environment artifacts from SPRproject()
- The R Markdown chunks from systemPipeRNAseq.Rmd were parsed into the expected number of executable workflow steps by importWF()

## Limitations

- Version requirements (R >= 3.2, Bioconductor >= 3.2) may limit reproducibility with newer environments
- Workflow templates for additional NGS applications were still under development and not yet available
- No changelog was found for tracking version-specific changes

## Evidence

- [abstract] End-to-end workflow execution with reporting: "It is an extensible environment for both building and running end-to-end analysis workflows with automated report generation for a wide range of NGS applications."
- [abstract] Pre-configured templates: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [intro] Design goal: "we designed systemPipeR as a Bioconductor package for building and running workflows for most NGS applications"
- [discussion] Unification of R and CLI resources: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [readme] README project scope: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems"
