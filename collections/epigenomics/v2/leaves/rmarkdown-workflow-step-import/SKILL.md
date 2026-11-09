---
name: rmarkdown-workflow-step-import
description: Use when a user wants to rapidly test or run an end-to-end NGS analysis
  (e.g. the RNA-Seq template) with systemPipeR's workflow toolkit starting from an
  R Markdown template that must be parsed into executable steps.
when_to_use_negative:
- The analysis is not an NGS application supported by systemPipeR's pre-configured
  templates (RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq) and no custom .Rmd workflow template
  exists
- The goal is to run only a single R/Bioconductor or command-line tool rather than
  building and executing a multi-step workflow with reporting
- The R environment does not meet systemPipeR's stated requirements (R version >=3.2,
  Bioconductor version >=3.2)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: systemPipeR
  role: Core toolkit providing genWorkenvir, SPRproject, importWF, runWF, renderReport
    and renderLogs to import, execute and report the .Rmd workflow steps
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Data package supplying the pre-configured RNA-Seq working environment and
    systemPipeRNAseq.Rmd template used as the import target
- name: R
  role: Language environment in which the systemPipeR workflow commands are run
- name: Bioconductor
  role: Distribution channel and hosting environment for systemPipeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:ff5df587efa9be82ba672c1770c64437f8269eaaf8d0b56c7688d99e3fffe46e
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rmarkdown-workflow-step-import

## Summary

Run an end-to-end systemPipeR NGS workflow by importing the R Markdown template's chunks as executable workflow steps: generate the working environment, import the .Rmd steps, execute, and render the scientific report, as in the pre-configured RNA-Seq template.

## When to use

Use when a user wants to rapidly test or run an end-to-end NGS analysis (e.g. the RNA-Seq template) with systemPipeR's workflow toolkit starting from an R Markdown template that must be parsed into executable steps.

## When NOT to use

- The analysis is not an NGS application supported by systemPipeR's pre-configured templates (RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq) and no custom .Rmd workflow template exists
- The goal is to run only a single R/Bioconductor or command-line tool rather than building and executing a multi-step workflow with reporting
- The R environment does not meet systemPipeR's stated requirements (R version >=3.2, Bioconductor version >=3.2)

## Inputs

- R Markdown workflow template (e.g. systemPipeRNAseq.Rmd)
- Parameter files (e.g. targets/CWL-style parameter files) generated in the spr_project directory
- NGS input data referenced by the template (e.g. RNA-Seq FASTQ reads)

## Outputs

- Executed workflow steps recorded in the SPR project logging/environment
- Rendered HTML scientific report (systemPipeRNAseq.html)
- Workflow logs/diagnostics from renderLogs()

## How to apply

Load systemPipeR in R and generate the working environment with genWorkenvir(work = "rnaseq"), which creates a spr_project directory pre-populated with parameter files and the systemPipeRNAseq.Rmd template. Initialize the workflow container with SPRproject() to set up project logging and the environment, then parse the R Markdown chunks into executable workflow steps with importWF("systemPipeRNAseq.Rmd"). Execute the full workflow with runWF(), which runs all imported steps, then generate the scientific report with renderReport() and inspect diagnostics with renderLogs(). The rationale is that systemPipeR provides a uniform workflow interface across NGS applications (RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq) with automated report generation, so the .Rmd template serves both as documentation and as the executable step definition.

## Related tools

- **systemPipeR** (Core toolkit providing genWorkenvir, SPRproject, importWF, runWF, renderReport and renderLogs to import, execute and report the .Rmd workflow steps) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Data package supplying the pre-configured RNA-Seq working environment and systemPipeRNAseq.Rmd template used as the import target)
- **R** (Language environment in which the systemPipeR workflow commands are run)
- **Bioconductor** (Distribution channel and hosting environment for systemPipeR)

## Examples

```
library(systemPipeR); genWorkenvir(work = "rnaseq"); SPRproject(); importWF("systemPipeRNAseq.Rmd"); runWF(); renderReport(); renderLogs()
```

## Evaluation signals

- runWF completes all imported steps without errors (workflow execution finishes cleanly)
- The HTML report file systemPipeRNAseq.html is rendered and exists on disk
- importWF successfully parsed the R Markdown chunks into executable workflow steps (step list populated in the SPRproject container)
- SPRproject logging records the run diagnostics, and renderLogs() produces the log report
- The spr_project directory contains the expected parameter files and the systemPipeRNAseq.Rmd template from genWorkenvir

## Limitations

- Workflow templates for additional NGS applications are still under development and not yet available
- Version requirement constraints (R >=3.2, Bioconductor >=3.2) may limit reproducibility with newer environments
- No changelog found for tracking package version differences

## Evidence

- [abstract] systemPipeR supports importing .Rmd workflow steps, executing them, and generating scientific reports: "It is an extensible environment for both building and running end-to-end analysis workflows with automated report generation for a wide range of NGS applications."
- [abstract] The package provides pre-configured RNA-Seq workflow and reporting templates: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [discussion] systemPipeR unites R/Bioconductor and command-line software for standardized NGS workflow execution: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [readme] README describes the project's purpose of designing and executing end-to-end workflows: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems"
