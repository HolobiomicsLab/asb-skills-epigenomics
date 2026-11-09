---
name: runtime-speedup-benchmarking
description: Use when a paper claims near-linear or 10x/100x speedup for parallel
  execution of sample-level NGS steps (e.g., short-read alignment over 100 FASTQ files)
  and the claim needs empirical reproduction.
when_to_use_negative:
- The workflow step cannot be parallelized per sample (e.g., steps operating jointly
  across all samples), since the claimed near-linear scaling relies on sample-level
  parallelization
- The goal is method-accuracy validation (alignment quality, variant calls) rather
  than runtime/speedup measurement
- No cluster or multi-core execution environment is available, so a 1-core vs N-core
  comparison cannot be run
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: systemPipeR
  role: Provides clusterRun to submit workflow steps (e.g., short-read alignment on
    sample FASTQ files) to a computer cluster with a configurable number of cores
    for the timing runs
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: R
  role: Language in which the workflow is loaded, timings are recorded, and speedup
    ratios are computed
- name: Bioconductor
  role: Distribution channel from which systemPipeR is installed for the benchmark
  repo: http://bioconductor.org/packages/devel/systemPipeR
- name: systemPipeRdata
  role: Supplies the sample RNA-Seq workflow project including sample FASTQ files
    and pre-configured targets file used as benchmark inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-016-1241-0
    title: 'systemPipeR: NGS workflow and report generation environment'
  source_content_hash: sha256:04259089b1ad8ec53dbf46c6dbabffbd84ed56898a8a315d050989cb8f2ced1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - systemPipeR
  - systemPipeRdata
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# runtime-speedup-benchmarking

## Summary

Benchmarks the parallel scaling of a workflow's runtime by comparing elapsed times of sample-parallel jobs submitted at 1 core versus N cores (e.g., via systemPipeR's clusterRun), computing observed speedup against the core count.

## When to use

Use when a paper claims near-linear or 10x/100x speedup for parallel execution of sample-level NGS steps (e.g., short-read alignment over 100 FASTQ files) and the claim needs empirical reproduction.

## When NOT to use

- The workflow step cannot be parallelized per sample (e.g., steps operating jointly across all samples), since the claimed near-linear scaling relies on sample-level parallelization
- The goal is method-accuracy validation (alignment quality, variant calls) rather than runtime/speedup measurement
- No cluster or multi-core execution environment is available, so a 1-core vs N-core comparison cannot be run

## Inputs

- sample RNA-Seq workflow project from systemPipeRdata (sample FASTQ files and pre-configured targets file)
- command-line alignment step templates for the workflow
- cluster scheduler access for submitting 1-core and N-core job batches

## Outputs

- speedup_table.csv with runtime entries for the 1-core and N-core conditions and a computed speedup value
- observed speedup ratio per core count, comparable to the paper's claimed 10x/100x figures

## How to apply

Instantiate the workflow project (e.g., the systemPipeR RNA-Seq workflow from systemPipeRdata, which supplies sample FASTQ files and a pre-configured targets file) and configure one compute-bound step such as short-read alignment using the package's command-line step templates. Submit the identical set of jobs twice on a computer cluster with clusterRun: once with 1 core and once with N cores, parallelizing at the sample (FASTQ file) level, recording elapsed runtime for each condition. Compute speedup as single-core runtime divided by N-core runtime and write the runtime/speedup table to CSV (e.g., speedup_table.csv). Because systemPipeR workflows parallelize every step per sample, observed speedup should approach N (e.g., near 10x on 10 cores, 100x on 100 cores over 100 FASTQ files) if the near-linear scaling claim holds; large deviations indicate scheduling overhead or insufficient samples to saturate the cores.

## Related tools

- **systemPipeR** (Provides clusterRun to submit workflow steps (e.g., short-read alignment on sample FASTQ files) to a computer cluster with a configurable number of cores for the timing runs) — http://bioconductor.org/packages/devel/systemPipeR
- **R** (Language in which the workflow is loaded, timings are recorded, and speedup ratios are computed)
- **Bioconductor** (Distribution channel from which systemPipeR is installed for the benchmark) — http://bioconductor.org/packages/devel/systemPipeR
- **systemPipeRdata** (Supplies the sample RNA-Seq workflow project including sample FASTQ files and pre-configured targets file used as benchmark inputs)

## Examples

```
clusterRun(args, FUN = runCommandline, Njobs = 10, convergence = "sample")  # time alignment of sample FASTQ files at N cores vs 1 core
```

## Evaluation signals

- speedup_table.csv contains runtime entries for both the 1-core and N-core conditions
- speedup_table.csv contains a computed speedup value (single-core runtime / multi-core runtime)
- Computed speedup approaches the number of cores used, validating the paper's near-linear scaling claim (≈10x on 10 cores, ≈100x on 100 cores for 100 FASTQ files)
- Both timing runs execute the identical set of alignment jobs (same FASTQ files and step templates), differing only in core count
- Elapsed runtimes are positive and multi-core runtime is strictly less than single-core runtime

## Limitations

- The paper's claim applies to workflows parallelized at the sample level; steps that cannot be parallelized per sample will not show near-linear scaling
- Older version requirements (R ≥3.2, Bioconductor ≥3.2) may limit reproducibility with newer environments
- Templates for additional NGS applications were still under development, so benchmark coverage is limited to pre-configured workflows (RNA-Seq, ChIP-Seq, VAR-Seq, Ribo-Seq)

## Evidence

- [abstract] support for running both R and command-line software on local computers and clusters: [evidence span withheld — anchor status: not_found]
- [abstract] the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq: "the package provides pre-configured workflows and reporting templates for RNA-Seq, ChIP-Seq, VAR-Seq and Ribo-Seq"
- [discussion] systemPipeR unites R/Bioconductor resources with external command-line software: "The systemPipeR package unites R/Bioconductor resources with external command-line software"
- [readme] README project description of executing workflows on HPC systems: "designing, building, and executing end-to-end data analysis workflows on local
machines and high-performance computing (HPC) systems"
