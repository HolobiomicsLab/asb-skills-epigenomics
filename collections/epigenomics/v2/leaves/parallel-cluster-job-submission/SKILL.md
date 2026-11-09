---
name: parallel-cluster-job-submission
description: Use when an R/Bioconductor NGS workflow must process many sample files
  (e.g., ~100 FASTQ files) faster by parallelizing per-sample steps across multiple
  cluster cores and quantifying the speedup.
when_to_use_negative:
- Workflow steps that cannot be parallelized at the sample level (e.g., steps operating
  jointly across all samples) — near-linear scaling depends on per-sample parallelization
- Runs on a local single-machine environment with no cluster/scheduler access, where
  clusterRun-based job submission is not applicable
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: systemPipeR
  role: Provides clusterRun for submitting workflow steps (R and command-line software)
    as parallel jobs to clusters and a uniform workflow interface across NGS applications
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Supplies the sample RNA-Seq workflow project with sample FASTQ files and pre-configured
    targets file used as the benchmark input
- name: R
  role: Programming environment in which the workflow is loaded, clusterRun is invoked,
    and elapsed runtimes/speedups are computed
- name: Bioconductor
  role: Distribution channel from which systemPipeR is installed for all common operating
    systems
  repo: http://bioconductor.org/packages/devel/systemPipeR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:cc1d30ae896a3efeae8c286a41ea3c421204fe729469476103734b7a8d223370
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# parallel-cluster-job-submission

## Summary

A method for submitting workflow steps (e.g., short-read alignment on multiple FASTQ files) as parallel jobs to an HPC cluster via systemPipeR's clusterRun, to achieve near-linear runtime scaling with the number of CPU cores/sample files.

## When to use

Use when an R/Bioconductor NGS workflow must process many sample files (e.g., ~100 FASTQ files) faster by parallelizing per-sample steps across multiple cluster cores and quantifying the speedup.

## When NOT to use

- Workflow steps that cannot be parallelized at the sample level (e.g., steps operating jointly across all samples) — near-linear scaling depends on per-sample parallelization
- Runs on a local single-machine environment with no cluster/scheduler access, where clusterRun-based job submission is not applicable

## Inputs

- Sample RNA-Seq workflow project from systemPipeRdata (sample FASTQ files + pre-configured targets file)
- Command-line step templates for the alignment stage (e.g., short-read alignment on FASTQ files)
- Cluster scheduler resources/configuration for clusterRun (number of cores N)

## Outputs

- Elapsed runtimes for the 1-core and N-core conditions
- speedup_table.csv containing runtime entries for 1-core and N-core conditions and a computed speedup value (single-core runtime / multi-core runtime)

## How to apply

Instantiate a pre-configured workflow project from systemPipeRdata (e.g., the sample RNA-Seq workflow with its FASTQ files and targets file), and configure the alignment-stage command-line step templates. First run the step on a single core via clusterRun and record elapsed runtime as the baseline. Resubmit the identical jobs with N cores so that per-sample (per-FASTQ-file) tasks are distributed across cluster nodes. Compute speedup as single-core runtime divided by N-core runtime; because every workflow step can be parallelized at the sample level, speedup should approach N (the paper claims 10x/100x for 10/100 cores).

## Related tools

- **systemPipeR** (Provides clusterRun for submitting workflow steps (R and command-line software) as parallel jobs to clusters and a uniform workflow interface across NGS applications) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Supplies the sample RNA-Seq workflow project with sample FASTQ files and pre-configured targets file used as the benchmark input)
- **R** (Programming environment in which the workflow is loaded, clusterRun is invoked, and elapsed runtimes/speedups are computed)
- **Bioconductor** (Distribution channel from which systemPipeR is installed for all common operating systems) — http://bioconductor.org/packages/devel/systemPipeR

## Examples

```
library(systemPipeR); library(systemPipeRdata); genWorkenvir(workflow="rnaseq"); runWF(...); clusterRun(args, conffile=".BatchJobs.R", Njobs=N, blocks=1)
```

## Evaluation signals

- speedup_table.csv contains runtime entries for both the 1-core and N-core conditions
- A computed speedup value (single-core runtime / multi-core runtime) is present in the table
- The speedup approaches the number of cores N used (e.g., ~10x for 10 cores, ~100x for 100 cores on 100 FASTQ files), consistent with near-linear sample-level scaling
- The same alignment jobs/templates are used for both conditions, so the only variable is core count
- Jobs actually execute on the cluster via clusterRun rather than locally (elapsed runtimes reflect scheduler-submitted execution)

## Limitations

- The paper's 10x/100x speedup claims were made for ~100 sample FASTQ files; scaling assumes every workflow step can be parallelized per sample, which may not hold for non-per-sample steps
- Reproducibility with newer environments may be constrained by version requirements (R ≥3.2, Bioconductor ≥3.2)
- Templates for additional NGS applications were still under development and not yet available at publication time

## Evidence

- [abstract] support for running both R and command-line software on local computers and clusters: "Its unique features include a uniform workflow interface across different NGS applications, automated report generation, and support for running both R and command-line software on local computers"
- [other] systemPipeR workflows parallelized at the sample level scale nearly linearly in runtime: [evidence span withheld — anchor status: not_found]
- [other] validation of the speedup table contents: [evidence span withheld — anchor status: not_found]
- [readme] systemPipeR executes on HPC systems: "The systemPipeR project provides a suite of R/Bioconductor packages for designing, building, and executing end-to-end data analysis workflows on local machines and high-performance computing (HPC)"
