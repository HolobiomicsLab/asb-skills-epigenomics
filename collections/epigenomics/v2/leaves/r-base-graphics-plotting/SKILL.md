---
name: r-base-graphics-plotting
description: Use when the task requires summarizing quality and diversity of short
  reads in FASTQ files as plots — e.g. per-cycle quality scores or read-quality distributions
  across many samples in one report figure.
when_to_use_negative:
- Input is already aligned BAM or a post-alignment QC artifact (e.g. coverage plots)
  rather than raw FASTQ short reads
- Only a single FASTQ file is involved and a simple per-file quality summary suffices
  — the multi-panel combined-report layout is designed for large numbers of FASTQ
  files
- The task requires computing alignment or variant statistics rather than read-level
  quality/diversity summaries
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: systemPipeR
  role: Provides the seeFastq function to compute FASTQ quality statistics and seeFastqPlot
    to render the combined multi-panel quality report figure
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming language in which the quality statistics are computed and the
    graphics are rendered
- name: Bioconductor
  role: Distribution channel for the systemPipeR package used in this workflow
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Example data package supplying the sample FASTQ files used as test inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:49207d3ac68418be04a54800c8543fd2d14c85d6f6fe094d31c970ece368184b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# r-base-graphics-plotting

## Summary

Generates multi-panel R graphics for NGS quality reporting, exemplified by systemPipeR's seeFastq/seeFastqPlot functions, which compute per-file FASTQ quality statistics and arrange them into a single combined figure for visual inspection of many FASTQ files.

## When to use

Use when the task requires summarizing quality and diversity of short reads in FASTQ files as plots — e.g. per-cycle quality scores or read-quality distributions across many samples in one report figure.

## When NOT to use

- Input is already aligned BAM or a post-alignment QC artifact (e.g. coverage plots) rather than raw FASTQ short reads
- Only a single FASTQ file is involved and a simple per-file quality summary suffices — the multi-panel combined-report layout is designed for large numbers of FASTQ files
- The task requires computing alignment or variant statistics rather than read-level quality/diversity summaries

## Inputs

- FASTQ files (short-read sequence data)
- systemPipeRdata example FASTQ dataset (for testing/demonstration)
- Vector of FASTQ file paths (one or many samples)

## Outputs

- Per-file FASTQ quality statistics (per-cycle quality scores, read-quality distributions, base composition/diversity summaries)
- Combined multi-panel quality-report figure (e.g. seeFastq_quality_report.png, a valid PNG image with one panel per FASTQ file)

## How to apply

Load the R/Bioconductor package systemPipeR (or reimplement equivalent logic in R base graphics) and obtain input FASTQ files, e.g. the sample FASTQ set distributed with the systemPipeRdata example dataset. For each FASTQ file, compute per-cycle base-call quality summaries, per-base/cycle nucleotide distributions, and read-level quality distributions (seeFastq-style statistics), batching reads so very large files remain tractable. Then assemble the per-file results into a single multi-panel figure (seeFastqPlot-style layout) with one quality panel per file, so that dozens of FASTQ files can be inspected in one report. Render the combined figure to a file such as seeFastq_quality_report.png using R's graphics device. Validation rests on the figure containing one panel per input FASTQ file and each panel exposing the per-cycle quality information.

## Related tools

- **systemPipeR** (Provides the seeFastq function to compute FASTQ quality statistics and seeFastqPlot to render the combined multi-panel quality report figure) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming language in which the quality statistics are computed and the graphics are rendered)
- **Bioconductor** (Distribution channel for the systemPipeR package used in this workflow) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Example data package supplying the sample FASTQ files used as test inputs)

## Examples

```
library(systemPipeR); fq <- seeFastq(fastq=c("sample1.fastq","sample2.fastq")); png("seeFastq_quality_report.png"); seeFastqPlot(fq); dev.off()
```

## Evaluation signals

- Output file seeFastq_quality_report.png exists and opens as a valid PNG image
- The figure is a combined multi-panel layout containing one quality panel per sampled FASTQ file
- Each panel displays per-cycle quality scores and read-quality distribution summaries for its FASTQ file
- Statistics are computed identically per file (same batching/summaries), so panels are comparable across samples
- Input FASTQ set matches the render — every input file appears in the report and no extra panels appear

## Limitations

- systemPipeR at publication time required R >=3.2 and Bioconductor >=3.2, which may limit reproducibility with newer environments
- Templates for additional NGS applications were still under development; functionality beyond the documented workflows may vary
- The multi-panel design targets visual inspection of large numbers of FASTQ files; per-file deep-dive QC tools may be better for single-sample diagnosis

## Evidence

- [full_text] seeFastq quality-report utility: [evidence span withheld — anchor status: not_found]
- [full_text] design of the report layout: [evidence span withheld — anchor status: not_found]
- [intro] R/Bioconductor environment: "R and the affiliated Bioconductor environment provide a substantial number of widely used tools with a large user base in this area"
- [abstract] example data: "systemPipeR is freely available for all common operating systems from Bioconductor ( http://bioconductor.org/packages/devel/systemPipeR )"
- [readme] README project scope: "executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems, while simultaneously generating publication-quality analysis reports"
