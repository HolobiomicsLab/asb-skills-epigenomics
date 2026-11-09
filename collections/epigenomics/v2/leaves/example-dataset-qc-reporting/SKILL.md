---
name: example-dataset-qc-reporting
description: Use when short-read sequencing data supplied as FASTQ files needs quality
  assessment — e.g., per-cycle quality scores, read-quality distributions, and QC
  summaries across many samples rendered in one combined multi-panel report figure.
when_to_use_negative:
- Input reads are already aligned (e.g., BAM) or provided in a non-FASTQ format, so
  no FASTQ quality computation applies
- The goal is read preprocessing (filtering/trimming) rather than generating a visual
  quality report
- Only one small FASTQ file needs a quick text-based QC summary rather than a combined
  multi-panel figure for many files
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0092
tools:
- name: systemPipeR
  role: Provides the seeFastq and seeFastqPlot functions that compute and plot FASTQ
    quality reports
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming language in which the quality-report workflow is implemented and
    rendered
- name: systemPipeRdata
  role: Example dataset package supplying sample FASTQ files for the quality-report
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:ed2060af0478fe5303fba58b38ef3a4ed382062776b1d92e35fa83c53160432d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# example-dataset-qc-reporting

## Summary

Generates and plots detailed per-file quality reports for FASTQ files using systemPipeR's seeFastq and seeFastqPlot functions, producing a single combined multi-panel figure that enables visual inspection of large numbers of FASTQ files.

## When to use

Use when short-read sequencing data supplied as FASTQ files needs quality assessment — e.g., per-cycle quality scores, read-quality distributions, and QC summaries across many samples rendered in one combined multi-panel report figure.

## When NOT to use

- Input reads are already aligned (e.g., BAM) or provided in a non-FASTQ format, so no FASTQ quality computation applies
- The goal is read preprocessing (filtering/trimming) rather than generating a visual quality report
- Only one small FASTQ file needs a quick text-based QC summary rather than a combined multi-panel figure for many files

## Inputs

- Short reads supplied as FASTQ files
- systemPipeRdata example dataset FASTQ files (for demonstration)

## Outputs

- Per-file FASTQ quality statistics (per-cycle quality scores, read-quality distributions, QC summaries)
- Combined multi-panel quality-report figure (e.g., seeFastq_quality_report.png, a valid PNG with one panel per FASTQ file)

## How to apply

Load the systemPipeR library and obtain sample FASTQ files from the systemPipeRdata example dataset. Compute per-file quality statistics with the seeFastq logic — per-cycle quality scores, read-quality distributions, and related QC summaries — across all FASTQ files. Then apply seeFastqPlot logic to arrange the per-file results into one multi-panel figure suitable for large numbers of files, mirroring the quality-report layout in the article. Render the combined figure (e.g., to seeFastq_quality_report.png) and verify it contains one quality panel per sampled FASTQ file.

## Related tools

- **systemPipeR** (Provides the seeFastq and seeFastqPlot functions that compute and plot FASTQ quality reports) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming language in which the quality-report workflow is implemented and rendered)
- **systemPipeRdata** (Example dataset package supplying sample FASTQ files for the quality-report workflow)

## Examples

```
library(systemPipeR); fqlist <- seeFastq(fastq=batch(fq), batchsize=10000, klength=8); seeFastqPlot(fqlist) # render combined panels to seeFastq_quality_report.png
```

## Evaluation signals

- Output file seeFastq_quality_report.png exists and opens as a valid PNG image
- Figure contains one quality panel per sampled FASTQ file
- Panels are arranged in a combined multi-panel layout matching the article's quality-report figure
- Per-cycle quality scores and read-quality distributions are present in each panel's computed statistics
- Report is easy to generate and supports visual inspection of large numbers of FASTQ files in a single report

## Limitations

- Version requirements (R >= 3.2, Bioconductor >= 3.2) may limit reproducibility with newer environments
- Quality-report utility is scoped to FASTQ short-read inputs; it is not a read trimming or filtering tool

## Evidence

- [full_text] seeFastq/seeFastqPlot quality-report utility: [evidence span withheld — anchor status: not_found]
- [full_text] report design intent: [evidence span withheld — anchor status: not_found]
- [abstract] package availability: "we have developed the R/Bioconductor package systemPipeR"
- [abstract] example dataset: ""package": "systemPipeRdata""
- [readme] project description: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows"
- [discussion] version constraints: "Other requirements: R version ≥3.2, Bioconductor version ≥3.2"
