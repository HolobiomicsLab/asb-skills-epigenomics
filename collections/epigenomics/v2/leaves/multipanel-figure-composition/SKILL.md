---
name: multipanel-figure-composition
description: Use when many per-sample QC summaries (e.g., FASTQ per-cycle quality
  and read-quality distributions) must be arranged into one combined multi-panel plot/report
  for visual inspection of large sample sets.
when_to_use_negative:
- Input is a single sample with no need for a combined multi-file layout — a single-panel
  QC plot suffices
- The data are not short reads in FASTQ format and no per-file QC summaries (per-cycle
  qualities, read-quality distributions) exist to arrange into panels
- The goal is interactive exploration rather than a reproducible, report-embedded
  figure artifact
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: systemPipeR
  role: R/Bioconductor package whose seeFastq and seeFastqPlot functions generate
    and plot the combined multi-panel FASTQ quality report
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming environment in which the per-file QC computation and panel composition
    are implemented
- name: systemPipeRdata
  role: Example dataset supplying the sample FASTQ files used to render the combined
    quality report
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:2ad571fb65a078fc0c61f14519c541168acbcff25dac736d2e9f6c988a3c0512
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multipanel-figure-composition

## Summary

Compose per-sample quality results (e.g., FASTQ QC statistics) into a single combined multi-panel figure, as systemPipeR's seeFastq/seeFastqPlot do for FASTQ quality reports, so large numbers of files can be visually inspected in one report.

## When to use

Use when many per-sample QC summaries (e.g., FASTQ per-cycle quality and read-quality distributions) must be arranged into one combined multi-panel plot/report for visual inspection of large sample sets.

## When NOT to use

- Input is a single sample with no need for a combined multi-file layout — a single-panel QC plot suffices
- The data are not short reads in FASTQ format and no per-file QC summaries (per-cycle qualities, read-quality distributions) exist to arrange into panels
- The goal is interactive exploration rather than a reproducible, report-embedded figure artifact

## Inputs

- FASTQ files (short-read sequencing reads), e.g., the example FASTQ set from the systemPipeRdata package
- Per-file quality statistics computed per FASTQ file (per-cycle quality scores, read-quality distributions, QC summaries)

## Outputs

- A combined multi-panel PNG figure (e.g., seeFastq_quality_report.png) with one quality panel per FASTQ file
- A single visual quality report covering all input FASTQ files for inspection

## How to apply

First compute per-file quality statistics for each input FASTQ file — per-cycle quality scores, read-quality distributions, and related QC summaries — as seeFastq does over the sample FASTQ files from systemPipeRdata. Then implement the seeFastqPlot-style arrangement: one quality panel per file, laid out in a grid so all files fit in a single multi-panel figure suitable for large numbers of FASTQ files. Render the combined layout to a raster output (e.g., seeFastq_quality_report.png) so the report can be embedded in the automated analysis report systemPipeR generates. The rationale is that systemPipeR's reporting feature 'improves reproducibility' and the quality reports are 'designed to facilitate visual inspection of large numbers of FASTQ files in a single report.'

## Related tools

- **systemPipeR** (R/Bioconductor package whose seeFastq and seeFastqPlot functions generate and plot the combined multi-panel FASTQ quality report) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming environment in which the per-file QC computation and panel composition are implemented)
- **systemPipeRdata** (Example dataset supplying the sample FASTQ files used to render the combined quality report)

## Examples

```
Rscript -e 'library(systemPipeR); fq <- seeFastq(fastq=sample_fastq_files, batchsize=10000, klength=8); png("seeFastq_quality_report.png"); seeFastqPlot(fq); dev.off()'
```

## Evaluation signals

- The output file seeFastq_quality_report.png exists and opens as a valid PNG image
- The figure contains exactly one quality panel per sampled FASTQ file
- Panels are arranged in a combined multi-panel grid layout, not as separate figures
- Each panel reproduces the report layout of the article's quality-report figure (per-cycle quality scores and read-quality distributions)
- The figure is suitable for visually inspecting large numbers of FASTQ files in a single report

## Limitations

- The skill targets visual inspection/report composition; it does not replace numerical QC thresholds or filtering decisions
- systemPipeR documentation notes version requirements (R >=3.2, Bioconductor >=3.2) that may limit reproducibility in newer environments

## Evidence

- [results] seeFastq and seeFastqPlot generate and plot detailed quality reports for FASTQ files; the reports are easy to generate and designed to facilitate visual inspection of large numbers of FASTQ files in a single report: [evidence span withheld — anchor status: not_found]
- [discussion] Its functionalities reduce the complexity and time required to translate NGS data into interpretable research results, while a built-in reporting feature improves reproducibility.: "Its functionalities reduce the complexity and time required to translate NGS data into interpretable research results, while a built-in reporting feature improves reproducibility."
- [readme] The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows"
