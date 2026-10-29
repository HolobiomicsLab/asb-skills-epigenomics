---
name: r-bioconductor-workflow-execution
description: Use when many NGS sample files (e.g., ~100 FASTQ files) must be processed
  through multi-step workflows (alignment, variant/peak calling) in R, or when per-sample
  parallelization on an HPC cluster is needed to cut runtime.
when_to_use_negative:
- The analysis is a single-tool, single-sample task with no multi-step workflow or
  parallelization need — running the tool directly is simpler
- Templates for the required NGS application are not yet available in the package
  (templates for additional applications were still under development)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: systemPipeR
  role: Bioconductor package that builds, runs, and reports end-to-end NGS workflows;
    provides clusterRun for parallel job submission and pre-configured RNA-Seq templates
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Companion data package supplying sample FASTQ files and pre-configured workflow
    templates for instantiation
- name: HISAT2
  role: Faster alternative aligner that can replace the default Bowtie2/TopHat2 alignment
    step to cut RNA-Seq workflow runtime below 3 hours for 100 FASTQ files
- name: Rsubread
  role: Alternative fast RNA-Seq aligner for the alignment step
- name: Bowtie2/TopHat2
  role: Default alignment configuration in the RNA-Seq template; serves as the 6-8
    hour baseline for runtime comparison on 100 FASTQ files
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:55c7739a6e1ccea9e7b9dcdd299c7fb29ae24b7d5bd43c9895b7d87625b78e4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Bowtie2
  - Bowtie2/TopHat2
  - HISAT2
  - R
  - Rsubread
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# r-bioconductor-workflow-execution

## Summary

Build, run, and scale end-to-end NGS workflows (e.g., RNA-Seq short-read alignment on FASTQ samples) inside R/Bioconductor using systemPipeR, which unifies command-line tools and R functions under one workflow interface with automated report generation and cluster-based parallel execution.

## When to use

Use when many NGS sample files (e.g., ~100 FASTQ files) must be processed through multi-step workflows (alignment, variant/peak calling) in R, or when per-sample parallelization on an HPC cluster is needed to cut runtime.

## When NOT to use

- The analysis is a single-tool, single-sample task with no multi-step workflow or parallelization need — running the tool directly is simpler
- Templates for the required NGS application are not yet available in the package (templates for additional applications were still under development)

## Inputs

- sample FASTQ read files (from the systemPipeRdata companion package or user data)
- pre-configured workflow template and parameter files from systemPipeR
- targets file mapping sample FASTQ files to workflow steps

## Outputs

- completed workflow outputs (e.g., BAM alignment files from HISAT2/Rsubread/Bowtie2 steps)
- runtime/speedup tables such as speedup_table.csv or runtime_alignment_comparison.csv
- automated consolidated analysis report

## How to apply

Initialize a fresh workflow project from systemPipeRdata, which ships sample FASTQ reads and a pre-configured targets file, then load the pre-configured RNA-Seq workflow template from systemPipeR's parameter files. Define each step (e.g., short-read alignment) in the step templates, optionally swapping the default Bowtie2/TopHat2 aligner for a faster one such as HISAT2 or Rsubread since alignment dominates compute time. Submit command-line steps as cluster jobs with clusterRun, parallelized at the sample level across CPU cores, because every workflow step can be parallelized per sample and runtime scales nearly linearly with sample count. Record wall-clock runtimes per condition (e.g., 1-core vs N-core) and compute speedup as single-core runtime divided by multi-core runtime to validate scaling. systemPipeR's built-in reporting feature generates the consolidated analysis report, improving reproducibility.

## Related tools

- **systemPipeR** (Bioconductor package that builds, runs, and reports end-to-end NGS workflows; provides clusterRun for parallel job submission and pre-configured RNA-Seq templates) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Companion data package supplying sample FASTQ files and pre-configured workflow templates for instantiation)
- **HISAT2** (Faster alternative aligner that can replace the default Bowtie2/TopHat2 alignment step to cut RNA-Seq workflow runtime below 3 hours for 100 FASTQ files)
- **Rsubread** (Alternative fast RNA-Seq aligner for the alignment step)
- **Bowtie2/TopHat2** (Default alignment configuration in the RNA-Seq template; serves as the 6-8 hour baseline for runtime comparison on 100 FASTQ files)

## Examples

```
library(systemPipeRdata); genWorkenvir(workflow="rnaseq"); runWF(args=args)
```

## Evaluation signals

- Runtime table contains entries for both 1-core and N-core conditions plus a computed speedup value that approaches the number of cores used (near-linear scaling)
- Alignment runtime comparison reproduces the reported pattern: HISAT2/Rsubread configuration under 3 hours vs 6-8 hours for Bowtie2/TopHat2 on 100 FASTQ files
- Workflow executes end-to-end on the systemPipeRdata sample FASTQ files without step failures
- Automated report is generated and reproduces the analysis results reproducibly

## Limitations

- Reported runtimes assume a computer cluster environment; single-machine runs may not reach the 10x/100x speedups claimed for 10/100 cores
- Version requirement constraints (R >= 3.2, Bioconductor >= 3.2) may limit reproducibility with newer environments
- Workflow templates for additional NGS applications were still under development and not yet available

## Evidence

- [abstract] uniform interface, cluster support: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [abstract] pre-configured RNA-Seq templates: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [full_text] near-linear sample-level scaling: [evidence span withheld — anchor status: not_found]
- [full_text] faster aligners cut runtime: [evidence span withheld — anchor status: not_found]
- [discussion] reporting improves reproducibility: "Its functionalities reduce the complexity and time required to translate NGS data into interpretable research results, while a built-in reporting feature improves reproducibility."
- [readme] project README overview: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems, while simultaneously generating publication-quality analysis"
