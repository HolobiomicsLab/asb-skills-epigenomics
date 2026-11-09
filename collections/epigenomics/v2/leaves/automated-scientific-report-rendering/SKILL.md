---
name: automated-scientific-report-rendering
description: Use when a systemPipeR workflow has been initialized with SPRproject(),
  its steps imported with importWF(), and executed with runWF(), and you now need
  a reproducible HTML analysis report or a diagnostics log of the run.
when_to_use_negative:
- The input is an ad hoc R analysis script with no systemPipeR SPRproject workflow
  — rendering requires an executed workflow container with imported steps, not standalone
  R output
- The workflow has not yet been run (runWF() has not completed) — there are no executed
  step results for renderReport() to summarize
- A non-systemPipeR pipeline (e.g., Nextflow, Snakemake) was used for execution, since
  the reporting feature is built into the systemPipeR package's workflow management
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: systemPipeR
  role: Provides renderReport(), renderLogs(), SPRproject(), importWF(), and runWF()
    for executing and automatically rendering the scientific report of the NGS workflow
    run
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming language in which the workflow execution and report rendering
    commands are run
- name: Bioconductor
  role: Distribution channel from which systemPipeR is installed and run
- name: systemPipeRdata
  role: Supplies the pre-configured RNA-Seq working environment and systemPipeRNAseq.Rmd
    template used as the report source
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:3c5731d487e57a16cdf87336e199214973f8d8d3b20d16549db47f7e7c859ce1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# automated-scientific-report-rendering

## Summary

Renders publication-quality scientific analysis reports and run diagnostics (logs) from an executed systemPipeR workflow, turning completed workflow steps and their metadata into a reproducible HTML document. This is the reporting layer that makes systemPipeR NGS runs (e.g., the RNA-Seq template) interpretable and auditable.

## When to use

Use when a systemPipeR workflow has been initialized with SPRproject(), its steps imported with importWF(), and executed with runWF(), and you now need a reproducible HTML analysis report or a diagnostics log of the run.

## When NOT to use

- The input is an ad hoc R analysis script with no systemPipeR SPRproject workflow — rendering requires an executed workflow container with imported steps, not standalone R output
- The workflow has not yet been run (runWF() has not completed) — there are no executed step results for renderReport() to summarize
- A non-systemPipeR pipeline (e.g., Nextflow, Snakemake) was used for execution, since the reporting feature is built into the systemPipeR package's workflow management

## Inputs

- Executed SPRproject workflow container (initialized with SPRproject(), steps imported via importWF("systemPipeRNAseq.Rmd"))
- systemPipeR workflow R Markdown template (e.g., systemPipeRNAseq.Rmd) and its associated parameter files
- Completed runWF() execution with all imported workflow steps finished

## Outputs

- Rendered HTML scientific report (e.g., systemPipeRNAseq.html)
- Rendered workflow run logs and diagnostics (renderLogs output)

## How to apply

After runWF() completes all imported steps without errors, call renderReport() to render the scientific report for the workflow defined in the working environment (e.g., generated via genWorkenvir(work = "rnaseq") and parsed from systemPipeRNAseq.Rmd). Call renderLogs() to inspect run diagnostics recorded by the SPRproject container, which sets up project logging and environment. The report aggregates the executed workflow steps — spanning short read preprocessing, alignment, variant calling, peak calling, and differential expression/binding analysis of genomic regions — into an automated HTML document. Validate success by confirming runWF finished without errors and that the rendered HTML report file (systemPipeRNAseq.html) exists. The rationale is systemPipeR's built-in reporting feature, which improves reproducibility by uniting R/Bioconductor resources with external command-line software under a uniform workflow interface.

## Related tools

- **systemPipeR** (Provides renderReport(), renderLogs(), SPRproject(), importWF(), and runWF() for executing and automatically rendering the scientific report of the NGS workflow run) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming language in which the workflow execution and report rendering commands are run)
- **Bioconductor** (Distribution channel from which systemPipeR is installed and run)
- **systemPipeRdata** (Supplies the pre-configured RNA-Seq working environment and systemPipeRNAseq.Rmd template used as the report source)

## Examples

```
renderReport(); renderLogs()
```

## Evaluation signals

- runWF() completes all imported workflow steps without errors before any rendering attempt
- The HTML report file (e.g., systemPipeRNAseq.html) is rendered and exists on disk
- renderLogs() output shows the executed step-by-step diagnostics recorded by the SPRproject logging environment
- The report reflects the full set of imported R Markdown chunks from systemPipeRNAseq.Rmd, parsed into executable workflow steps
- Report content is reproducible: re-running importWF() and runWF() on the same environment regenerates consistent analysis results

## Limitations

- Reporting depends on the systemPipeR runtime environment; older versions constrained R >= 3.2 and Bioconductor >= 3.2, which may limit reproducibility with newer environments
- Pre-configured workflow/reporting templates were limited to RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq; templates for additional NGS applications were still under development
- No changelog was found for the discussion section, so version history of the reporting feature may be incomplete

## Evidence

- [abstract] unique features include automated report generation: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software"
- [discussion] reporting improves reproducibility: "Its functionalities reduce the complexity and time required to translate NGS data into interpretable research results, while a built-in reporting feature improves reproducibility."
- [readme] suite of R/Bioconductor packages with report generation: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems, while simultaneously generating"
- [abstract] reproducible analysis results from NGS: "systemPipeR accelerates the extraction of reproducible analysis results from NGS experiments."
