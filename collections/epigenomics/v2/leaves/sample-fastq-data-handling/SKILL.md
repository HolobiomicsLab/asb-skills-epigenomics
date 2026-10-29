---
name: sample-fastq-data-handling
description: Use when a workflow must process many sample FASTQ files (e.g., RNA-Seq)
  on a cluster and the goal is to parallelize per-sample steps and measure or achieve
  near-linear speedup with core count.
when_to_use_negative:
- The analysis goal is a single-sample or non-parallelizable step that cannot be split
  per sample file
- Input data is not a set of sample FASTQ files and there is no targets file defining
  sample-level units
- No computer cluster or multi-core scheduler is available, so clusterRun-based submission
  cannot be used
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0769
tools:
- name: systemPipeR
  role: Provides the workflow environment, targets-file interface, and clusterRun
    function for submitting per-sample command-line jobs (e.g., alignment) to a computer
    cluster with a chosen core count
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Supplies the sample RNA-Seq workflow project with sample FASTQ files and the
    pre-configured targets file used as input
- name: R
  role: Programming language in which the workflow is instantiated, the clusterRun
    submissions are made, and runtimes/speedups are computed
- name: Bioconductor
  role: Distribution channel from which systemPipeR is installed
  repo: http://bioconductor.org/packages/devel/systemPipeR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:6cadc87e7655ce3ab9a8f59f16844531a23190a93550ff9279f1007422295bf9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sample-fastq-data-handling

## Summary

Method for parallelizing NGS workflow steps (e.g., short-read alignment) at the sample level across a set of FASTQ files using systemPipeR's clusterRun, so that runtime scales nearly linearly with the number of samples/cores on a computer cluster.

## When to use

Use when a workflow must process many sample FASTQ files (e.g., RNA-Seq) on a cluster and the goal is to parallelize per-sample steps and measure or achieve near-linear speedup with core count.

## When NOT to use

- The analysis goal is a single-sample or non-parallelizable step that cannot be split per sample file
- Input data is not a set of sample FASTQ files and there is no targets file defining sample-level units
- No computer cluster or multi-core scheduler is available, so clusterRun-based submission cannot be used

## Inputs

- Sample FASTQ files from the systemPipeRdata RNA-Seq workflow project
- Pre-configured targets file listing the samples
- Workflow command-line step templates (e.g., for short-read alignment)
- clusterRun resource configuration (core count, batch/qsub parameters)

## Outputs

- Alignment outputs for each sample FASTQ file
- Runtime table for 1-core vs N-core conditions
- speedup_table.csv containing runtimes and computed speedup values

## How to apply

Load systemPipeR in R and instantiate a pre-configured workflow project from systemPipeRdata, which supplies the sample FASTQ files and the targets file listing them. Configure the per-sample step of interest (e.g., short-read alignment) with the workflow's command-line step templates, then submit the jobs with clusterRun first under a single-core condition and record elapsed runtime. Resubmit the identical jobs via clusterRun with N cores so that each sample (FASTQ) file is processed as an independent task on the cluster. Compute speedup as single-core runtime divided by N-core runtime and write the runtime/speedup table (e.g., speedup_table.csv); because every workflow step can be parallelized per sample, observed speedup should approach the number of cores used, matching the paper's claim of ~10x/100x speedup for 10/100 cores on 100 FASTQ files.

## Related tools

- **systemPipeR** (Provides the workflow environment, targets-file interface, and clusterRun function for submitting per-sample command-line jobs (e.g., alignment) to a computer cluster with a chosen core count) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Supplies the sample RNA-Seq workflow project with sample FASTQ files and the pre-configured targets file used as input)
- **R** (Programming language in which the workflow is instantiated, the clusterRun submissions are made, and runtimes/speedups are computed)
- **Bioconductor** (Distribution channel from which systemPipeR is installed) — http://bioconductor.org/packages/devel/systemPipeR

## Examples

```
Rscript -e 'library(systemPipeR); library(systemPipeRdata); genWorkenvir(workflow="rnaseq"); setwd("rnaseq"); runWF(...); clusterRun(args, conffile=".BatchJobs.R", resource=list(walltime="20:00", nodes=paste0("1:ppn=", ncores)))'
```

## Evaluation signals

- The runtime table contains elapsed-runtime entries for both the 1-core and N-core clusterRun conditions
- A computed speedup value (single-core runtime / multi-core runtime) is present and approaches the number of cores used for near-linear scaling
- Alignment jobs completed for every sample in the targets file in both conditions (same set of per-sample outputs)
- speedup_table.csv is written and includes the runtime and speedup columns
- Identical command-line step templates and inputs were used in both the single-core and multi-core submissions, so the comparison is controlled

## Limitations

- Scaling assumes every workflow step can be parallelized per sample; steps that pool data across samples will not benefit
- Version requirement constraints (R version ≥3.2, Bioconductor version ≥3.2) may limit reproducibility with newer environments
- Workflow templates for additional NGS applications were still under development and not yet available
- No changelog was found, complicating reproducibility tracking

## Evidence

- [full_text] systemPipeR workflows parallelized at the sample level scale nearly linearly in runtime with the number of sample (FASTQ) files on a computer cluster, since every workflow step can be parallelized per sample.: [evidence span withheld — anchor status: not_found]
- [abstract] the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [abstract] uniform workflow interface across NGS applications with cluster support: "support for running both R and command-line software on local computers"
- [readme] README introduction describing workflow execution on local machines and HPC systems: "designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC) systems"
- [discussion] version constraints limiting reproducibility: "Other requirements: R version ≥3.2, Bioconductor version ≥3.2"
