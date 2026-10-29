---
name: fastq-quality-metrics-computation
description: Use when short-read sequencing data are supplied as FASTQ files and per-cycle/per-read
  quality summaries must be computed and visually inspected, especially across many
  files in one report.
when_to_use_negative:
- Input is not FASTQ (e.g. BAM alignments, feature counts) — seeFastq operates on
  raw read files
- Only a single trivially small FASTQ needs a one-line read (e.g. summary from ShortRead's
  qa) and no combined multi-file report is needed
- Quality control is already summarized in an existing QC report and only downstream
  alignment/variant steps are required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3572
tools:
- name: systemPipeR
  role: Provides seeFastq (compute per-file FASTQ quality statistics) and seeFastqPlot
    (render combined multi-panel quality report)
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Supplies the example/sample FASTQ files used to exercise the quality-report
    workflow
- name: R
  role: Execution environment in which the seeFastq/seeFastqPlot calls and PNG rendering
    are scripted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:3d98f59e81e003ba6577735d2fd9cbab8d86ff5b0cecc9f5b3f1ddd4ac3d7968
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# fastq-quality-metrics-computation

## Summary

Computes and visualizes per-file FASTQ quality reports using systemPipeR's seeFastq and seeFastqPlot functions, producing per-cycle quality scores, read-quality distributions, and a combined multi-panel QC figure suitable for inspecting large numbers of FASTQ files.

## When to use

Use when short-read sequencing data are supplied as FASTQ files and per-cycle/per-read quality summaries must be computed and visually inspected, especially across many files in one report.

## When NOT to use

- Input is not FASTQ (e.g. BAM alignments, feature counts) — seeFastq operates on raw read files
- Only a single trivially small FASTQ needs a one-line read (e.g. summary from ShortRead's qa) and no combined multi-file report is needed
- Quality control is already summarized in an existing QC report and only downstream alignment/variant steps are required

## Inputs

- One or more FASTQ files (short-read sequencing reads)
- systemPipeRdata example FASTQ files for testing/reference

## Outputs

- Per-file FASTQ quality statistics object (per-cycle quality scores, read-quality distributions)
- Multi-panel PNG quality report with one panel per FASTQ file (e.g. seeFastq_quality_report.png)

## How to apply

Load the systemPipeR library and point seeFastq at the FASTQ files (e.g. those shipped with the systemPipeRdata example dataset), computing per-cycle quality scores and read-quality distributions per file; seeFastq supports batched processing (batchsize) to bound memory on large libraries. Then pass the resulting per-file statistics object to seeFastqPlot, which arranges one quality panel per file into a single multi-panel figure mirroring the quality-report layout in the systemPipeR article. Render the combined figure to a file such as seeFastq_quality_report.png via a graphics device. Rationale: the package was designed as an R/Bioconductor workflow environment whose reporting utilities make QC 'easy to generate' and enable 'visual inspection of large numbers of FASTQ files in a single report.'

## Related tools

- **systemPipeR** (Provides seeFastq (compute per-file FASTQ quality statistics) and seeFastqPlot (render combined multi-panel quality report)) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Supplies the example/sample FASTQ files used to exercise the quality-report workflow)
- **R** (Execution environment in which the seeFastq/seeFastqPlot calls and PNG rendering are scripted)

## Examples

```
library(systemPipeR); fq <- seeFastq(fastq=c("reads1.fastq","reads2.fastq"), batchsize=1e5); png("seeFastq_quality_report.png"); seeFastqPlot(fq); dev.off()
```

## Evaluation signals

- Output file seeFastq_quality_report.png exists and opens as a valid PNG image
- The figure contains one quality panel per sampled FASTQ file in a combined multi-panel layout
- Per-cycle quality score and read-quality distribution summaries are present for every input file
- Statistics are computed over all supplied FASTQ files, not a subset (batchsize does not silently drop files)
- Reimplementation reproduces the report layout shown in the systemPipeR article's quality-report figure

## Limitations

- Requires R >= 3.2 and Bioconductor >= 3.2, which may limit reproducibility with newer environments
- Large FASTQ collections require batched computation (batchsize) to stay within memory; per-cycle statistics over full files can be expensive
- Templates/utilities beyond those documented may still be under development (the article notes additional workflow templates were not yet available)

## Evidence

- [abstract] Bioconductor availability of the package: "systemPipeR is freely available for all common operating systems from Bioconductor"
- [discussion] Package unites R/Bioconductor resources: "The systemPipeR package unites R/Bioconductor resources with external command-line software"
- [readme] README describes the suite of R/Bioconductor packages for workflows and reports: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows"
