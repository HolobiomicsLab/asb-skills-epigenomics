---
name: r-commandline-mixed-step-execution
description: Use when a pipeline must interleave R code steps with external command-line
  steps (e.g. an R export step followed by gzip/gunzip utilities) and needs to run
  inside a single reproducible systemPipeR workflow container.
when_to_use_negative:
- The task involves only R code with no external command-line tools — plain R scripts
  avoid the SAL/SYSargsList overhead
- The pipeline steps are already defined in a workflow manager outside R/Bioconductor
  (e.g. a pure CWL or Snakemake setup) — mixing in systemPipeR adds no value
- Command-line tools cannot be expressed via SYSargsList target/outfiles connectivity,
  such as interactive or daemon-like software
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: systemPipeR
  role: Provides SAL (SYSargsList) containers, LineWise/SYSargsList step classes,
    importWF, and workflow execution for mixed R and command-line steps
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Language in which LineWise steps are authored and in which the workflow container
    is driven
- name: Bioconductor
  role: Distribution channel for systemPipeR and its pre-configured workflow templates
  repo: http://bioconductor.org/packages/devel/systemPipeR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:22baef627e65e59ebe28332d5e7c0ab207082eca151932b882d418a6390c93be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# r-commandline-mixed-step-execution

## Summary

Defines and executes a mixed R/command-line workflow in systemPipeR by authoring workflow steps in an R Markdown template, importing them into a SYSargsList (SAL) container with importWF, and running them in dependency order so each step consumes the prior step's outputs.

## When to use

Use when a pipeline must interleave R code steps with external command-line steps (e.g. an R export step followed by gzip/gunzip utilities) and needs to run inside a single reproducible systemPipeR workflow container.

## When NOT to use

- The task involves only R code with no external command-line tools — plain R scripts avoid the SAL/SYSargsList overhead
- The pipeline steps are already defined in a workflow manager outside R/Bioconductor (e.g. a pure CWL or Snakemake setup) — mixing in systemPipeR adds no value
- Command-line tools cannot be expressed via SYSargsList target/outfiles connectivity, such as interactive or daemon-like software

## Inputs

- R Markdown workflow template (e.g. spr_simple_wf.Rmd) containing LineWise and SYSargsList step declarations ending in appendStep(SAL)
- systemPipeR package installed in R/Bioconductor
- Input data consumed by the first step (e.g. iris data frame exported to files)
- Project working directory (e.g. spr_project)

## Outputs

- Loaded SYSargsList (SAL) workflow container with registered R and command-line steps
- Files produced per step (e.g. iris subset exports, gzip-compressed targets, gunzipped round-trip files)
- Final R step outputs (e.g. statistics computed on round-tripped files) written under the project directory

## How to apply

Author an R Markdown workflow template (e.g. spr_simple_wf.Rmd from the systemPipeR extdata directory) in which each step's last assignment is an appendStep call on a SAL container, using LineWise for pure R steps and SYSargsList for command-line steps. Load systemPipeR, create/reset the project working directory, copy the template into the project, and load it with importWF so all R and command-line steps are registered in the SAL container. Chain steps via outfiles-to-targets connectivity so that, for example, a gzip step compresses the files produced by the preceding R export step, and a gunzip step restores them for a final R statistics step. Execute steps in order (e.g. runWF or step-wise execution), then verify that every step completed without error and its generated outputs exist under the project directory.

## Related tools

- **systemPipeR** (Provides SAL (SYSargsList) containers, LineWise/SYSargsList step classes, importWF, and workflow execution for mixed R and command-line steps) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Language in which LineWise steps are authored and in which the workflow container is driven)
- **Bioconductor** (Distribution channel for systemPipeR and its pre-configured workflow templates) — http://bioconductor.org/packages/devel/systemPipeR

## Examples

```
library(systemPipeR); spr_project <- SPRproject(); SAL <- importWF(spr_project, file_path="spr_simple_wf.Rmd"); SAL <- runWF(spr_project, SAL)
```

## Evaluation signals

- Every workflow step in the SAL container reports completion without error after execution
- Each step's declared target files exist under the project directory (e.g. gzip outputs present after the gzip step)
- The gunzip step reproduces files matching the outputs of the export step (round-trip integrity)
- importWF registers the expected number of R (LineWise) and command-line (SYSargsList) steps from the Rmd template
- Each step's last assignment in the Rmd is an appendStep(SAL, ...) call — a template missing this fails to import as a workflow step

## Limitations

- Workflow templates for additional NGS applications were still under development and not yet available at publication time
- Version requirements (R >=3.2, Bioconductor >=3.2) may limit reproducibility with newer environments
- Steps must follow the appendStep convention; R code blocks without a terminating appendStep(SAL) assignment are not recognized as workflow steps by importWF

## Evidence

- [other] A workflow step in an Rmd file must have appendStep of SAL as its last assignment, using SYSargsList or LineWise for command-line or R code steps, respectively.: [evidence span withheld — anchor status: not_found]
- [abstract] support for running both R and command-line software on local computers: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [readme] designing, building, and executing end-to-end data analysis workflows on local machines and HPC systems: "designing, building, and executing end-to-end data analysis workflows on local
machines and high-performance computing (HPC) systems"
- [intro] systemPipeR integrates a wide array of command-line and R/Bioconductor software into workflows: "we designed systemPipeR as a Bioconductor package for building and running workflows for most NGS applications with support for integrating a wide array of command-line and R/Bioconductor software."
