---
name: interactive-html-qc-report-generation
description: Use when a cfMeDIP-seq (or MeDIP-seq) pipeline has produced per-sample
  QC outputs (FASTQC, SAMtools, Picard, MEDIPS reports) and they must be aggregated
  into one interactive HTML summary across samples, groups, or batches.
when_to_use_negative:
- Input is raw FASTQ only and no upstream QC (FASTQC/SAMtools/Picard/MEDIPS) outputs
  exist yet — run the upstream QC pipeline stages first
- The goal is methylation quantification aggregation (bin-sample TXT matrices) rather
  than QC metric summarization — use the aggregation/filtering skill instead
- A single-sample run where a per-tool native QC report already suffices and no cross-sample
  or cross-batch comparison is needed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: MultiQC
  role: Aggregates per-sample FASTQC, SAMtools, Picard, and MEDIPS QC reports into
    a single summarized HTML report (v1.0.dev0), used with an embedded script
  repo: https://github.com/pughlab/MEDIPIPE
- name: Snakemake
  role: Orchestrates the QC-report aggregation rule with containerized Conda execution
    environments and pinned tool versions in YAML files
  repo: https://github.com/pughlab/MEDIPIPE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:d26ac6d1e32d08bdfb45047cc1f761da634b54e69cc9ece22bdfc14e31be3b18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - MultiQC
  - Snakemake
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Interactive HTML QC report generation

## Summary

Aggregates per-sample QC metrics from cfMeDIP-seq processing (FASTQC, SAMtools, Picard, MEDIPS) with MultiQC plus an embedded script into a summarized HTML report that allows interactive examination of sample QC metrics across different groups or batches.

## When to use

Use when a cfMeDIP-seq (or MeDIP-seq) pipeline has produced per-sample QC outputs (FASTQC, SAMtools, Picard, MEDIPS reports) and they must be aggregated into one interactive HTML summary across samples, groups, or batches.

## When NOT to use

- Input is raw FASTQ only and no upstream QC (FASTQC/SAMtools/Picard/MEDIPS) outputs exist yet — run the upstream QC pipeline stages first
- The goal is methylation quantification aggregation (bin-sample TXT matrices) rather than QC metric summarization — use the aggregation/filtering skill instead
- A single-sample run where a per-tool native QC report already suffices and no cross-sample or cross-batch comparison is needed

## Inputs

- Per-sample FASTQC reports from raw reads
- Per-sample SAMtools alignment QC outputs
- Per-sample Picard metrics files
- Per-sample MEDIPS QC reports
- Sample sheet / metadata defining groups or batches

## Outputs

- Summarized interactive HTML QC report with selected QC metrics for all aggregated samples

## How to apply

After methylation quantification and aggregation steps, collect the per-sample QC reports generated during FASTQC, SAMtools, Picard, and MEDIPS execution. Run MultiQC (v1.0.dev0) together with an embedded custom script to merge these tool-specific reports, since no single tool parses all four outputs natively. Configure the report to expose selected QC metrics so users can interactively compare samples across different groups or batches (e.g. brain cancer subtypes). The pipeline itself is orchestrated by Snakemake, which deploys execution environments automatically via Conda, so the MultiQC step should be defined as a rule with pinned versions in YAML files. Validate that the final HTML report renders the selected QC metrics for every aggregated sample (e.g. the 163-sample demonstration set spanning 6 brain cancer subtypes).

## Related tools

- **MultiQC** (Aggregates per-sample FASTQC, SAMtools, Picard, and MEDIPS QC reports into a single summarized HTML report (v1.0.dev0), used with an embedded script) — https://github.com/pughlab/MEDIPIPE
- **Snakemake** (Orchestrates the QC-report aggregation rule with containerized Conda execution environments and pinned tool versions in YAML files) — https://github.com/pughlab/MEDIPIPE

## Examples

```
git clone https://github.com/pughlab/MEDIPIPE
```

## Evaluation signals

- The HTML QC report renders selected QC metrics for all aggregated samples (e.g. the 163-sample, 6-subtype brain cancer demonstration set)
- QC reports from all four tools (FASTQC, SAMtools, Picard, MEDIPS) are represented in the aggregated output, not a subset
- Metrics can be examined interactively per sample across defined groups or batches
- The MultiQC version is pinned (v1.0.dev0) and the rule runs in the Snakemake-deployed Conda environment

## Limitations

- The report summarizes only selected QC metrics; full per-tool reports must be consulted for deeper inspection
- Aggregation scale is bounded by what MultiQC can render in a single HTML file for very large cohorts
- The pipeline was developed and tested for cfMeDIP-seq data (single-end or paired-end, with or without spike-in/UMI sequences); other data types may require configuration changes

## Evidence

- [abstract] MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation: "MEDIPIPE, which provides a one-stop solution for cfMeDIP-seq data quality control, methylation quantification, and sample aggregation"
- [intro] MEDIPIPE was developed with [Snakemake], which will automatically deploy the execution environments: "MEDIPIPE was developed with [Snakemake](https://snakemake.readthedocs.io/en/stable/index.html), which will automatically deploy the execution environments"
- [readme] Users can also specify the version of softwares to be installed in YAML files. It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines.: "Users can also specify the version of softwares to be installed in YAML files. It can also be performed across different cluster engines (e.g. SLURM) or stand-alone machines."
- [intro] It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling.: "It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling."
