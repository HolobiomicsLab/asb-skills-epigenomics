---
name: per-base-quality-profiling
description: Use when raw short-read data is supplied as FASTQ files and you need
  per-cycle quality summaries before or after preprocessing, especially for many samples
  requiring one combined visual QC report.
when_to_use_negative:
- Input reads are already aligned (BAM) or a QC report has already been generated
  for the FASTQ files
- Quality profiling of non-FASTQ data such as feature tables or expression matrices
  is required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3572
tools:
- name: systemPipeR
  role: Provides the seeFastq and seeFastqPlot functions that compute and plot FASTQ
    quality reports
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Supplies the sample/example FASTQ files used to exercise the quality-report
    workflow
- name: R
  role: Programming environment in which the seeFastq/seeFastqPlot workflow is run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:2af6ed308948b06806fff03cef83e7b522423ae1b28730a1565e18b4e0a0d849
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-base-quality-profiling

## Summary

Generates and plots per-file FASTQ quality reports using systemPipeR's seeFastq/seeFastqPlot functions, summarizing per-cycle quality scores and read-quality distributions so many FASTQ files can be visually inspected in a single multi-panel report.

## When to use

Use when raw short-read data is supplied as FASTQ files and you need per-cycle quality summaries before or after preprocessing, especially for many samples requiring one combined visual QC report.

## When NOT to use

- Input reads are already aligned (BAM) or a QC report has already been generated for the FASTQ files
- Quality profiling of non-FASTQ data such as feature tables or expression matrices is required

## Inputs

- One or more FASTQ files (e.g. sample FASTQ files from the systemPipeRdata example dataset)
- R/Bioconductor environment with systemPipeR installed

## Outputs

- Per-file FASTQ quality statistics (per-cycle quality scores, read-quality distributions, QC summaries)
- A single multi-panel PNG quality-report figure (e.g. seeFastq_quality_report.png) with one panel per FASTQ file

## How to apply

Load systemPipeR and point seeFastq at the FASTQ files (e.g. the systemPipeRdata example set); the function computes per-file quality statistics including per-cycle quality scores and read-quality distributions. Pass the returned list to seeFastqPlot, which arranges one quality panel per FASTQ file into a single combined multi-panel figure designed for visual inspection of large numbers of files. Render the figure to a PNG (e.g. seeFastq_quality_report.png) so the report can be checked for one panel per input file. Rationale: the article positions these functions as an easy-to-generate quality-report utility within systemPipeR's NGS workflow and report generation environment, supporting the short-read preprocessing stage before alignment.

## Related tools

- **systemPipeR** (Provides the seeFastq and seeFastqPlot functions that compute and plot FASTQ quality reports) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Supplies the sample/example FASTQ files used to exercise the quality-report workflow)
- **R** (Programming environment in which the seeFastq/seeFastqPlot workflow is run)

## Examples

```
library(systemPipeR); fq <- seeFastq(fastq=c(paths_to_fastq_files), batchsize=10000); png("seeFastq_quality_report.png", height=30, width=150, units="in", res=72); seeFastqPlot(fq); dev.off()
```

## Evaluation signals

- Output file seeFastq_quality_report.png exists and opens as a valid PNG image
- The figure contains exactly one quality panel per sampled FASTQ file in a combined multi-panel layout
- Panels include per-cycle quality score summaries and read-quality distributions for each file
- Statistics are computed for all supplied FASTQ files, matching the sample set count

## Limitations

- Report layout is designed for inspecting large numbers of FASTQ files in a single report; extremely large inputs may still affect rendering time
- Version constraints (R >= 3.2, Bioconductor >= 3.2) may limit reproducibility with newer environments

## Evidence

- [results] systemPipeR's seeFastq and seeFastqPlot functions generate and plot detailed quality reports for FASTQ files; the reports are easy to generate and designed to facilitate visual inspection of large numbers of FASTQ files in a single report.: [evidence span withheld — anchor status: not_found]
- [intro] These include short read pre-processors, aligners, variant and peak callers: "These include short read pre-processors, aligners, variant and peak callers"
- [readme] The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems, while simultaneously generating publication-quality analysis reports.: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows"
