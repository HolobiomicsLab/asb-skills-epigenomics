---
name: cwl-step-parameter-binding
description: Use when constructing a systemPipeR CL workflow step and you need to
  connect targets-file columns (e.g. FileName1, FileName2, SampleName) to CWL/yml
  parameters to render exact command-line calls.
when_to_use_negative:
- The step is pure R code with no external command-line software, so no CWL parameter
  connection is needed
- You only need to execute a pre-built, pre-configured workflow template without inspecting
  or customizing the CL calls
- Input sample metadata is not in a systemPipeR targets file format, so inputvars
  column-element matching cannot be established
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: systemPipeR
  role: Provides the targets/CWL/yml parameter-connection machinery, SYSargs containers,
    and the cmdlist accessor used to render exact CL calls
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming environment in which the SYSargs container is constructed and
    cmdlist calls are rendered
- name: Bioconductor
  role: Distribution channel through which systemPipeR and its extdata CWL/yml workflow
    files are provided
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:a1e55803066a991834a247fdcdd47de71a0bbb3a6953d89ee22073fe91cd7bc5
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
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cwl-step-parameter-binding

## Summary

Method for binding sample-level values from a systemPipeR targets file onto CWL-declared command-line parameters (via yml configs and inputvars) so that exact per-sample CL calls can be rendered with cmdlist before running an NGS workflow step.

## When to use

Use when constructing a systemPipeR CL workflow step and you need to connect targets-file columns (e.g. FileName1, FileName2, SampleName) to CWL/yml parameters to render exact command-line calls.

## When NOT to use

- The step is pure R code with no external command-line software, so no CWL parameter connection is needed
- You only need to execute a pre-built, pre-configured workflow template without inspecting or customizing the CL calls
- Input sample metadata is not in a systemPipeR targets file format, so inputvars column-element matching cannot be established

## Inputs

- CWL step-definition file (e.g. extdata/cwl/gunzip/gunzip.cwl)
- yml configuration file declaring the step's parameter values (e.g. gunzip.yml)
- targets file with sample-level columns (e.g. FileName1, FileName2, SampleName)
- inputvars bindings mapping targets columns to yml-declared parameters

## Outputs

- SYSargs/SYSargsList container for the step
- Rendered per-sample command lines via cmdlist (e.g. gunzip_cmdlist.txt) with all $(inputs.*) placeholders substituted by targets values and file paths

## How to apply

Load the step's CWL definition from the package's extdata/cwl directory (e.g. extdata/cwl/gunzip) along with its matching yml parameter file and the targets file. Establish the parameter connection by assigning inputvars such that each inputvars name matches a targets-file column (e.g. FileName1) and has corresponding element names in the yml file; this is the mechanism systemPipeR uses to pass sample-level values into CWL placeholders like $(inputs.gz). Construct the SYSargs container for the step, then render the exact command lines with the cmdlist accessor and write the record (e.g. gunzip_cmdlist.txt). Validate that the rendered cmdlist contains one command per targets row, every placeholder variable is substituted, and no unresolved inputvars tokens remain.

## Related tools

- **systemPipeR** (Provides the targets/CWL/yml parameter-connection machinery, SYSargs containers, and the cmdlist accessor used to render exact CL calls) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming environment in which the SYSargs container is constructed and cmdlist calls are rendered)
- **Bioconductor** (Distribution channel through which systemPipeR and its extdata CWL/yml workflow files are provided)

## Examples

```
args <- SYSargs2(wf_targets="targets.txt", wf_file="gunzip.cwl", input_file="gunzip.yml", dir_path="extdata/cwl/gunzip"); cmdlist(args)
```

## Evaluation signals

- Rendered cmdlist contains exactly one command per targets file row
- All placeholder variables (e.g. $(inputs.gz)) are substituted with targets values and file paths
- No unresolved inputvars tokens remain in the rendered command lines
- Each inputvars name matches both a targets-file column and element names in the yml file
- The cmdlist record (e.g. gunzip_cmdlist.txt) is written and inspectable before execution

## Limitations

- Parameter binding only succeeds when inputvars names have matching column names in the targets file and element names in the yml file; any mismatch silently breaks placeholder substitution
- systemPipeR unites R/Bioconductor with external command-line software, so bindings apply only to CL-defined steps, not pure-R steps
- Older version requirements (R >=3.2, Bioconductor >=3.2) may limit reproducibility with newer environments

## Evidence

- [abstract] Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [discussion] The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications.: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [readme] The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC)"
