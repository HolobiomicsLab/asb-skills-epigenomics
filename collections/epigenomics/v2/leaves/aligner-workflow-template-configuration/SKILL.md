---
name: aligner-workflow-template-configuration
description: Use when an existing systemPipeR RNA-Seq workflow is too slow because
  its alignment step dominates compute time, and the goal is to substitute HISAT2
  or Rsubread for Bowtie2/TopHat2 and quantify the runtime speedup.
when_to_use_negative:
- The analysis goal is not speed benchmarking but changing biological output, e.g.
  switching aligners to improve sensitivity rather than measuring wall-clock runtime
  against the 6-8 h baseline
- The input is not short-read RNA-Seq FASTQ data or the workflow is not the systemPipeR
  RNA-Seq template (e.g. ChIP-Seq, VAR-Seq, or Ribo-Seq templates, which have different
  alignment configurations)
- The environment cannot meet the paper's version constraints (R >= 3.2, Bioconductor
  >= 3.2), so template parameter files may not load reproducibly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: systemPipeR
  role: R/Bioconductor package that initializes the spr_project, loads the pre-configured
    RNA-Seq workflow template, executes the modified workflow, and automates report
    generation
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: HISAT2
  role: Faster short-read RNA-Seq aligner substituted into the alignment step to reduce
    runtime below 3 hours for 100 FASTQ files
- name: Rsubread
  role: Alternative faster RNA-Seq aligner that can replace Bowtie2/TopHat2 for the
    same speedup goal
- name: Bowtie2
  role: Default baseline aligner in the systemPipeR RNA-Seq template; its 6-8 hour
    runtime on 100 FASTQ files is the benchmark to beat
- name: TopHat2
  role: Splice-aware aligner paired with Bowtie2 in the original template configuration
    being replaced
- name: systemPipeRdata
  role: Companion package supplying the sample FASTQ reads used to execute and benchmark
    the modified workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:b33c7b8ea5752ef254c256911c9323ad3a0ce6bcd7fa1b31ec73494d0b52786f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie2
  - HISAT2
  - R
  - Rsubread
  - TopHat2
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# aligner-workflow-template-configuration

## Summary

Configures a systemPipeR RNA-Seq workflow template by swapping the default Bowtie2/TopHat2 alignment step for a faster aligner such as HISAT2 or Rsubread, then benchmarking wall-clock runtime against the paper's 6-8 hour baseline to reduce compute time for 100 mammalian FASTQ files below 3 hours.

## When to use

Use when an existing systemPipeR RNA-Seq workflow is too slow because its alignment step dominates compute time, and the goal is to substitute HISAT2 or Rsubread for Bowtie2/TopHat2 and quantify the runtime speedup.

## When NOT to use

- The analysis goal is not speed benchmarking but changing biological output, e.g. switching aligners to improve sensitivity rather than measuring wall-clock runtime against the 6-8 h baseline
- The input is not short-read RNA-Seq FASTQ data or the workflow is not the systemPipeR RNA-Seq template (e.g. ChIP-Seq, VAR-Seq, or Ribo-Seq templates, which have different alignment configurations)
- The environment cannot meet the paper's version constraints (R >= 3.2, Bioconductor >= 3.2), so template parameter files may not load reproducibly

## Inputs

- Mammalian FASTQ read files (sample reads from the systemPipeRdata companion package)
- systemPipeR RNA-Seq workflow template parameter files (default Bowtie2/TopHat2 configuration)
- Baseline runtime figure from the paper: 6-8 hours for Bowtie2/TopHat2 on 100 FASTQ files on a cluster

## Outputs

- Modified workflow parameter file invoking HISAT2/Rsubread in place of Bowtie2/TopHat2
- Recorded wall-clock runtime of the HISAT2/Rsubread alignment step
- Two-row runtime comparison table written to runtime_alignment_comparison.csv

## How to apply

Initialize a fresh spr_project in R from systemPipeR's pre-configured RNA-Seq template, which ships with parameter files specifying the default Bowtie2/TopHat2 alignment configuration. Edit the alignment step in the workflow parameter file to invoke HISAT2 (or Rsubread) in place of Bowtie2/TopHat2, keeping all downstream steps (short read preprocessing, alignment, quantification, differential expression) unchanged so runtime differences are attributable to the aligner alone. Execute the modified workflow with systemPipeR on the sample FASTQ reads from the systemPipeRdata companion package, recording the wall-clock runtime of the alignment step. Compare against the paper's baseline of 6-8 hours on a cluster for 100 FASTQ files with Bowtie2/TopHat2 versus under 3 hours for HISAT2/Rsubread, assembling both timings into a two-row comparison table. The rationale is that alignment accounts for most of the workflow's total compute time, so swapping the aligner yields the largest end-to-end speedup without altering other analysis logic.

## Related tools

- **systemPipeR** (R/Bioconductor package that initializes the spr_project, loads the pre-configured RNA-Seq workflow template, executes the modified workflow, and automates report generation) — http://bioconductor.org/packages/devel/systemPipeR
- **HISAT2** (Faster short-read RNA-Seq aligner substituted into the alignment step to reduce runtime below 3 hours for 100 FASTQ files)
- **Rsubread** (Alternative faster RNA-Seq aligner that can replace Bowtie2/TopHat2 for the same speedup goal)
- **Bowtie2** (Default baseline aligner in the systemPipeR RNA-Seq template; its 6-8 hour runtime on 100 FASTQ files is the benchmark to beat)
- **TopHat2** (Splice-aware aligner paired with Bowtie2 in the original template configuration being replaced)
- **systemPipeRdata** (Companion package supplying the sample FASTQ reads used to execute and benchmark the modified workflow)

## Examples

```
Rscript -e 'library(systemPipeR); spr_project(template="RNA-Seq"); ## edit alignment step: HISAT2 in place of Bowtie2/TopHat2; runWorkflow()'
```

## Evaluation signals

- The comparison table (runtime_alignment_comparison.csv) reproduces the reported speedup pattern: HISAT2/Rsubread < 3 h vs Bowtie2/TopHat2 6-8 h for 100 FASTQ files
- The modified parameter file invokes HISAT2 in the alignment step while all non-alignment steps (preprocessing, quantification, differential expression) remain identical to the template
- The workflow executes end-to-end on systemPipeRdata sample FASTQ reads without alignment-step errors
- Recorded wall-clock runtimes are present for both configurations in the two-row table
- The workflow runs within systemPipeR's supported environment (R >= 3.2, Bioconductor >= 3.2)

## Limitations

- The 6-8 h baseline and <3 h speedup figures were reported on a cluster for 100 FASTQ files; local runs on small sample reads will not reproduce absolute timings, only the relative speedup pattern
- No changelog is available for the package, complicating tracking of template parameter-file changes across versions
- Version requirements (R >= 3.2, Bioconductor >= 3.2) may limit reproducibility with newer environments
- Workflow templates for additional NGS applications were still under development at publication time

## Evidence

- [results] Because the Bowtie2/TopHat2 alignment step accounts for most of the workflow's compute time, using faster RNA-Seq aligners such as Rsubread or HISAT2 can reduce the compute time to less than 3 hours.: [evidence span withheld — anchor status: not_found]
- [abstract] The package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq.: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [discussion] systemPipeR unites R/Bioconductor resources with external command-line software for workflow execution.: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [discussion] Version requirement constraints that may limit reproducibility with newer environments.: "Other requirements: R version ≥3.2, Bioconductor version ≥3.2"
