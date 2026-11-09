---
name: rna-seq-workflow-environment-generation
description: Use when a user needs to rapidly test or run a complete pre-configured
  RNA-Seq NGS workflow in R/Bioconductor, from environment generation through execution
  to HTML scientific reporting.
when_to_use_negative:
- The NGS application of interest has no pre-configured template, since 'Templates
  for additional NGS applications are under development and will be added to the package
  in the near future.'
- The user needs a single, ad hoc analysis step (e.g., one alignment or one peak call)
  rather than an end-to-end managed workflow with reporting
- The environment cannot satisfy version requirements (R version >=3.2, Bioconductor
  version >=3.2), which may limit reproducibility with newer environments
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: systemPipeR
  role: R/Bioconductor package providing genWorkenvir, SPRproject, importWF, runWF,
    renderReport, and renderLogs to generate, import, execute, and report the RNA-Seq
    workflow
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Companion data package supplying the pre-configured RNA-Seq template and parameter
    files used by genWorkenvir
- name: R
  role: Programming language in which the workflow commands are run
- name: Bioconductor
  role: Distribution channel from which systemPipeR is installed
  repo: http://bioconductor.org/packages/devel/systemPipeR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:e48eb292aeb0c2fe5715e0c450363d051dae2126adad5d6597c904ce1c65263c
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rna-seq-workflow-environment-generation

## Summary

Generates and executes a pre-configured end-to-end RNA-Seq analysis workflow using systemPipeR's workflow management toolkit (genWorkenvir, SPRproject, importWF, runWF, renderReport), producing reproducible, report-backed NGS results from the systemPipeRdata template.

## When to use

Use when a user needs to rapidly test or run a complete pre-configured RNA-Seq NGS workflow in R/Bioconductor, from environment generation through execution to HTML scientific reporting.

## When NOT to use

- The NGS application of interest has no pre-configured template, since 'Templates for additional NGS applications are under development and will be added to the package in the near future.'
- The user needs a single, ad hoc analysis step (e.g., one alignment or one peak call) rather than an end-to-end managed workflow with reporting
- The environment cannot satisfy version requirements (R version >=3.2, Bioconductor version >=3.2), which may limit reproducibility with newer environments

## Inputs

- RNA-Seq working environment template (systemPipeRNAseq.Rmd) supplied by systemPipeRdata
- Pre-populated parameter files in the generated spr_project directory
- R Markdown chunks defining workflow steps to be parsed by importWF

## Outputs

- spr_project directory containing the workflow environment, logs, and parameter files
- Executed workflow steps with run logging from SPRproject
- Rendered scientific report file systemPipeRNAseq.html
- Run diagnostics log from renderLogs()

## How to apply

Load systemPipeR in R and generate the working environment with genWorkenvir(work = "rnaseq"), which creates a spr_project directory pre-populated with parameter files and the systemPipeRNAseq.Rmd template. Initialize the workflow container with SPRproject() to set up project logging and the execution environment. Import the workflow steps by parsing the R Markdown chunks into executable steps with importWF("systemPipeRNAseq.Rmd"), then execute all imported steps with runWF(). Rationale: systemPipeR unites R/Bioconductor resources with external command-line software to standardize and automate NGS analyses, and the four-command cycle (genWorkenvir/SPRproject, importWF, runWF, renderReport/renderLogs) covers generation, import, execution, and scientific reporting of the RNA-Seq template.

## Related tools

- **systemPipeR** (R/Bioconductor package providing genWorkenvir, SPRproject, importWF, runWF, renderReport, and renderLogs to generate, import, execute, and report the RNA-Seq workflow) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Companion data package supplying the pre-configured RNA-Seq template and parameter files used by genWorkenvir)
- **R** (Programming language in which the workflow commands are run)
- **Bioconductor** (Distribution channel from which systemPipeR is installed) — http://bioconductor.org/packages/devel/systemPipeR

## Examples

```
library(systemPipeR); genWorkenvir(work = "rnaseq"); SPRproject(); importWF("systemPipeRNAseq.Rmd"); runWF(); renderReport(); renderLogs()
```

## Evaluation signals

- runWF() completes all imported steps without errors
- The HTML report file systemPipeRNAseq.html is rendered and exists on disk
- The spr_project directory is created by genWorkenvir(work = "rnaseq") and contains parameter files and systemPipeRNAseq.Rmd
- SPRproject() initializes a project log and the workflow environment records every imported step
- renderLogs() returns run diagnostics reflecting the executed steps

## Limitations

- Templates for additional NGS applications were still under development and not yet available at publication time
- Version constraints (R >=3.2, Bioconductor >=3.2) may limit reproducibility with newer environments
- The skill validates via workflow completion and report rendering; scientific correctness of individual analysis steps still depends on the underlying tools configured in the parameter files

## Evidence

- [abstract] Pre-configured templates: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [abstract] End-to-end environment: "It is an extensible environment for both building and running end-to-end analysis workflows with automated report generation for a wide range of NGS applications."
- [discussion] Unification of resources: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [readme] Project scope from README: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems"
