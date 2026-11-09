---
name: rna-seq-short-read-alignment
description: Use when aligning RNA-Seq FASTQ files inside a systemPipeR workflow and
  you need to choose or benchmark an aligner (Bowtie2/TopHat2 vs HISAT2/Rsubread)
  or verify near-linear parallel speedup via clusterRun.
when_to_use_negative:
- The analysis goal is a downstream step (variant calling, peak calling, or differential
  expression analysis) rather than the alignment stage itself.
- Inputs are already aligned (BAM) or the task is read pre-processing/trimming (the
  'short read pre-processors' filter step), not alignment.
- The workflow is not runnable via systemPipeR (e.g. no R/Bioconductor >=3.2 environment,
  per the paper's stated R >=3.2, Bioconductor >=3.2 requirement).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: systemPipeR
  role: Bioconductor workflow environment that unites R with external command-line
    aligners, pre-configured RNA-Seq templates, and clusterRun for sample-level parallelization
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Programming language in which the workflow is loaded and the runtime/speedup
    tables are assembled
- name: Bioconductor
  role: Distribution channel for systemPipeR; requires Bioconductor >=3.2
- name: HISAT2
  role: Faster RNA-Seq aligner swapped into the alignment step to cut runtime below
    3 h for 100 FASTQ files
- name: Bowtie2
  role: Default aligner in the workflow template; its configuration defines the 6-8
    h baseline runtime
- name: TopHat2
  role: Default RNA-Seq aligner configuration whose runtime (6-8 h for 100 FASTQ files)
    serves as the comparison baseline
- name: systemPipeRdata
  role: Companion package providing the sample FASTQ files and pre-configured RNA-Seq
    workflow template
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:8faaaab9b55ed25e587d938d573751a01ea8ff0c0f16aa1a4a904f795360ccba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Bowtie2
  - HISAT2
  - R
  - TopHat2
  - systemPipeR
  - systemPipeRdata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# rna-seq-short-read-alignment

## Summary

Run short-read alignment for RNA-Seq within systemPipeR workflows, swapping the alignment step between Bowtie2/TopHat2 and faster aligners such as HISAT2 or Rsubread, and benchmarking runtime scaling across CPU-core counts on FASTQ sample sets.

## When to use

Use when aligning RNA-Seq FASTQ files inside a systemPipeR workflow and you need to choose or benchmark an aligner (Bowtie2/TopHat2 vs HISAT2/Rsubread) or verify near-linear parallel speedup via clusterRun.

## When NOT to use

- The analysis goal is a downstream step (variant calling, peak calling, or differential expression analysis) rather than the alignment stage itself.
- Inputs are already aligned (BAM) or the task is read pre-processing/trimming (the 'short read pre-processors' filter step), not alignment.
- The workflow is not runnable via systemPipeR (e.g. no R/Bioconductor >=3.2 environment, per the paper's stated R >=3.2, Bioconductor >=3.2 requirement).

## Inputs

- FASTQ files (sample short reads, e.g. the small sample FASTQ reads shipped with systemPipeRdata)
- systemPipeR targets file listing samples
- systemPipeR workflow parameter files (pre-configured RNA-Seq template, alignment step cwl/param configuration)
- CPU-core count specification for clusterRun

## Outputs

- Aligned reads per sample (short-read alignments, e.g. BAM output from HISAT2/Rsubread)
- runtime_alignment_comparison.csv (two-row table: Bowtie2/TopHat2 baseline 6-8 h vs HISAT2/Rsubread <3 h for 100 FASTQ files)
- speedup_table.csv (runtimes for 1-core and N-core conditions plus computed speedup value)

## How to apply

Instantiate the pre-configured RNA-Seq workflow template from systemPipeRdata in R, which ships sample FASTQ files and the targets file. Edit the workflow's alignment step parameter file to invoke HISAT2 (or Rsubread) in place of the default Bowtie2/TopHat2 configuration. Execute the workflow on the sample FASTQ reads, recording wall-clock runtime of the alignment step; since alignment accounts for most of the workflow's compute time, the faster aligners reduce compute from a 6-8 h baseline (100 mammalian FASTQ files on a cluster) to under 3 h. For parallelization, submit alignment jobs per sample via clusterRun first with 1 core, then with N cores, because every workflow step can be parallelized per sample, yielding near-linear runtime scaling with sample-file count. Assemble runtimes and computed speedup (single-core runtime / multi-core runtime) into a comparison table (e.g. runtime_alignment_comparison.csv or speedup_table.csv).

## Related tools

- **systemPipeR** (Bioconductor workflow environment that unites R with external command-line aligners, pre-configured RNA-Seq templates, and clusterRun for sample-level parallelization) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Programming language in which the workflow is loaded and the runtime/speedup tables are assembled)
- **Bioconductor** (Distribution channel for systemPipeR; requires Bioconductor >=3.2)
- **HISAT2** (Faster RNA-Seq aligner swapped into the alignment step to cut runtime below 3 h for 100 FASTQ files)
- **Bowtie2** (Default aligner in the workflow template; its configuration defines the 6-8 h baseline runtime)
- **TopHat2** (Default RNA-Seq aligner configuration whose runtime (6-8 h for 100 FASTQ files) serves as the comparison baseline)
- **systemPipeRdata** (Companion package providing the sample FASTQ files and pre-configured RNA-Seq workflow template)

## Examples

```
library(systemPipeRdata); genWorkenvir(workflow="rnaseq"); runWF(args=args)  # then edit the alignment step param to invoke HISAT2 and time it vs the Bowtie2/TopHat2 baseline
```

## Evaluation signals

- The runtime comparison table reproduces the reported speedup pattern: HISAT2/Rsubread configuration substantially lower than the Bowtie2/TopHat2 baseline (<3 h vs 6-8 h for 100 FASTQ files).
- The speedup table contains runtime entries for both the 1-core and N-core clusterRun conditions, and the computed speedup (single-core runtime / multi-core runtime) approaches the number of cores used.
- The workflow executes end-to-end on the systemPipeRdata sample FASTQ files with the edited alignment-step parameter file, producing alignment output for every sample in the targets file.
- Wall-clock runtimes are recorded per alignment step for each condition, not estimated, so the CSV tables are traceable to actual executions.

## Limitations

- The 6-8 h vs <3 h runtimes were reported for 100 mammalian FASTQ files on a cluster; timings on the small sample FASTQ files shipped with systemPipeRdata validate the speedup pattern rather than reproduce the absolute hours.
- Version constraints limit reproducibility: R version >=3.2 and Bioconductor version >=3.2 are required, which may not hold in newer environments.
- Workflow templates for additional NGS applications are under development and not yet available, so this skill is grounded in the RNA-Seq (and similar pre-configured) templates only.

## Evidence

- [abstract] alignment step dominates runtime: "It is an extensible environment for both building and running end-to-end analysis workflows with automated report generation for a wide range of NGS applications."
- [abstract] pre-configured RNA-Seq template: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [abstract] support for clusters and command-line tools: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [intro] aligners are workflow steps: "These include short read pre-processors, aligners, variant and peak callers"
- [discussion] systemPipeR unites R with external command-line software: "The systemPipeR package unites R/Bioconductor resources with external command-line software to standardize and automate the analysis of a wide range of NGS applications."
- [discussion] version requirements limit reproducibility: "Other requirements: R version ≥3.2, Bioconductor version ≥3.2"
- [readme] README project description: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems"
