---
name: sal-container-workflow-construction
description: Use when a multi-step analysis must be declared, loaded, and run as a
  reproducible systemPipeR SAL workflow where R steps and command-line steps chain
  via output-to-input targets (e.g., export -> gzip -> gunzip -> stats).
when_to_use_negative:
- The analysis is a single one-off script with no inter-step dependencies — a SAL
  container adds unnecessary structure
- Steps cannot be expressed as R code or command-line calls declared in an Rmd chunk
  via appendStep (e.g., interactive GUI tools)
- The workflow is not an NGS/data-analysis pipeline within the systemPipeR R/Bioconductor
  environment
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3489
tools:
- name: systemPipeR
  role: Provides importWF, SYSargsList, LineWise, and appendStep for declaring and
    loading workflow steps into a SAL container, and for executing them
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming language in which workflow steps and the SAL container are defined
    and executed
- name: Bioconductor
  role: Distribution channel for the systemPipeR package and its workflow templates
  repo: http://bioconductor.org/packages/devel/systemPipeR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:59b2b1d9c6411ae1f316af7e4142b68c23a4019a10f229096bf66989dac3f309
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
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sal-container-workflow-construction

## Summary

Builds a systemPipeR workflow container (SYSargsList, SAL) by importing an R Markdown template with importWF, which registers each Rmd chunk as a step declared via appendStep using SYSargsList (command-line) or LineWise (R code), then executes steps sequentially using outfiles-to-targets connectivity.

## When to use

Use when a multi-step analysis must be declared, loaded, and run as a reproducible systemPipeR SAL workflow where R steps and command-line steps chain via output-to-input targets (e.g., export -> gzip -> gunzip -> stats).

## When NOT to use

- The analysis is a single one-off script with no inter-step dependencies — a SAL container adds unnecessary structure
- Steps cannot be expressed as R code or command-line calls declared in an Rmd chunk via appendStep (e.g., interactive GUI tools)
- The workflow is not an NGS/data-analysis pipeline within the systemPipeR R/Bioconductor environment

## Inputs

- R Markdown workflow template (e.g., spr_simple_wf.Rmd) with steps ending in appendStep assignments
- systemPipeR package (R/Bioconductor) and its extdata templates
- Project working directory (SAL workflow container target)
- Input data files for the first step (e.g., iris data to export)

## Outputs

- SAL (SYSargsList) workflow container with registered R and command-line steps
- Executed step outputs under the project directory (e.g., exported iris files, gzipped/gunzipped files, statistics results)
- Workflow status showing all steps completed without error

## How to apply

Load systemPipeR in R and create/reset a project working directory (e.g., spr_project). Locate and copy the workflow Rmd template (e.g., spr_simple_wf.Rmd) from the installed package's extdata directory into the project. Author the Rmd so each step's last assignment is an appendStep call on the SAL object, using SYSargsList for command-line steps and LineWise for R code steps. Import the Rmd with importWF, which recognizes the declared steps and loads them into the SAL container. Run steps in order with runWF (or runStepWF), relying on outfiles-to-targets connectivity so each command-line step consumes the previous step's outputs; finish with the final R step and validate that all steps completed without error and outputs exist under the project directory.

## Related tools

- **systemPipeR** (Provides importWF, SYSargsList, LineWise, and appendStep for declaring and loading workflow steps into a SAL container, and for executing them) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming language in which workflow steps and the SAL container are defined and executed)
- **Bioconductor** (Distribution channel for the systemPipeR package and its workflow templates) — http://bioconductor.org/packages/devel/systemPipeR

## Examples

```
library(systemPipeR); sal <- SPRproject(); sal <- importWF(sal, file_path="spr_simple_wf.Rmd"); sal <- runWF(sal)
```

## Evaluation signals

- importWF registers every Rmd chunk ending in an appendStep assignment as a workflow step in the SAL container
- All workflow steps complete without error when executed via runWF
- Each command-line step consumes the prior step's outputs through outfiles-to-targets connectivity (e.g., gzip outputs feed the gunzip step)
- Generated outputs (exported iris files, gzipped/gunzipped files, final statistics) exist under the spr_project directory
- The final R statistics step runs successfully on the round-tripped files from the gunzip step

## Limitations

- Workflow steps must follow the appendStep convention (SYSargsList or LineWise as the last assignment) or importWF will not recognize them
- Command-line step chaining depends on correct outfiles-to-targets connectivity; misdeclared targets break downstream step consumption
- Version requirements (R >= 3.2, Bioconductor >= 3.2) may limit reproducibility with newer environments
- Templates for additional NGS applications may still be under development and not yet available

## Evidence

- [full_text] Rmd appendStep convention: [evidence span withheld — anchor status: not_found]
- [abstract] uniform workflow interface: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [discussion] package purpose: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [readme] README project scope: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC)"
