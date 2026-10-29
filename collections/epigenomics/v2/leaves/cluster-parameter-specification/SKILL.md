---
name: cluster-parameter-specification
description: Use when configuring a new NEAT project (Targets.txt) that will execute
  its unzip/QC/chiprx/map/filter/peakcalling blocks on an institutional qsub or bsub
  cluster and the cluster-specific header fields must be set.
when_to_use_negative:
- The pipeline will run on a local machine or a cloud service without the institution's
  qsub/bsub scheduler — the parameters are explicitly cluster-specific and meaningless
  elsewhere.
- You are already at step 4 of the NEAT workflow (transferring the project locally
  and creating human-readable pdf/xls outputs), where no scheduler submission occurs.
- The Targets.txt 'SAMPLES INFO' section and 'Steps_to_execute_pipe' line are what
  need editing — those belong to sample-configuration and pipeline-block assembly,
  not the cluster header.
tools:
- name: NEAT
  role: Reads the cluster-parameter header of Targets.txt to schedule and throttle
    pipeline module submissions on the institutional cluster
  repo: https://github.com/pschorderet/NEAT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-016-0902-3
    title: 'NEAT: a framework for building fully automated NGS pipelines and analyses'
  source_content_hash: sha256:f89267a2198bf0f4f629c7e00e68ba4e0557bc1d9a51e77b104a23fece1ed79e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pschorderet/NEAT
  tools:
  - NEAT
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cluster-parameter-specification

## Summary

Specifies the cluster-scheduler block of NEAT's centralized Targets.txt project file — storage space, maximum number of job submissions, wall time, queuing priority, node allocation, CPUs and virtual paths to reference genomes — so the ChIPseq/RNAseq pipeline can run on the institution's own cluster.

## When to use

Use when configuring a new NEAT project (Targets.txt) that will execute its unzip/QC/chiprx/map/filter/peakcalling blocks on an institutional qsub or bsub cluster and the cluster-specific header fields must be set.

## When NOT to use

- The pipeline will run on a local machine or a cloud service without the institution's qsub/bsub scheduler — the parameters are explicitly cluster-specific and meaningless elsewhere.
- You are already at step 4 of the NEAT workflow (transferring the project locally and creating human-readable pdf/xls outputs), where no scheduler submission occurs.
- The Targets.txt 'SAMPLES INFO' section and 'Steps_to_execute_pipe' line are what need editing — those belong to sample-configuration and pipeline-block assembly, not the cluster header.

## Inputs

- Targets.txt template from the NEAT package
- Institution cluster scheduler type (qsub or bsub)
- Cluster quotas: storage space, maximum job submissions, wall time, queuing priority, node allocation, number of CPUs
- Virtual paths to reference genomes on the cluster

## Outputs

- Completed cluster-parameter header of a project-specific Targets.txt
- Executable NEAT pipeline configured to submit jobs (unzip, QC, chiprx, map, filter, peakcalling, bigwig, wig, GRanges) to the institutional cluster

## How to apply

Start from the original Targets.txt template shipped with the NEAT package and copy it into the newly created project directory. Fill the upper portion of Targets.txt with cluster-specific parameters — queuing priority, node allocation, number of CPUs, wall time, storage space and maximum number of job submissions — plus virtual paths to reference genomes; these values stay fixed for a given institution's cluster because NEAT always runs on that institution's own scheduler (qsub or bsub). Leave the 'SAMPLES INFO' section and the 'Steps_to_execute_pipe' line (exact plain-word blocks such as 'unzip', 'QC', 'chiprx', 'map', 'filter', 'peakcalling', 'cleanfiles', 'granges') to be handled by the sample-configuration and pipeline-assembly steps. Rationale: centralizing cluster settings in one file lets non-expert users run reproducible NGS pipelines without terminal access or manual scheduler tuning, and keeps NEAT efficient (<24 h completion).

## Related tools

- **NEAT** (Reads the cluster-parameter header of Targets.txt to schedule and throttle pipeline module submissions on the institutional cluster) — https://github.com/pschorderet/NEAT

## Evaluation signals

- Targets.txt upper portion contains all cluster fields: storage space, maximum number of job submissions, wall time, queuing priority, node allocation, number of CPUs and virtual paths to reference genomes.
- Scheduler keywords in Targets.txt match the institution's actual scheduler (qsub or bsub).
- Virtual paths to reference genomes resolve to existing genome files on the cluster so the 'map' module (bwa or bowtie) can dereference them.
- Values in the cluster header are constant across samples and pipeline blocks — they should not vary per sample, unlike the SAMPLES INFO section.
- Validation pass confirms Targets.txt exists with all required fields (sample names, relationships, antibody, alignment/filtering/peak-calling parameters, cluster paths).

## Limitations

- NEAT runs on the institution's own cluster; parameters are not portable to other institutions' schedulers without re-tuning.
- Operating system support is limited to Mac OSx; no Linux/Windows support is stated.
- NEAT is still under development — bugs should be reported, and no changelog or version number is provided in the discussion section.

## Evidence

- [abstract] Cluster parameters are customizable: "Storage space, maximum number of job submissions, wall time and cluster-specific parameters can be customized as NEAT is run on the institution's cluster"
- [abstract] Centralized file encodes all settings: "NEAT projects are built around a unique and centralized file containing sample names, replicates, conditions, antibodies, alignment-, filtering- and peak calling parameters as well as"
- [readme] Cluster execution modes: "NEAT can be run either on a cluster (qsub or bsub) via the command line or directly through the respective applications."
- [readme] Pipeline modules to be scheduled: "[ 2 ]       Build a pipeline

    (i)     Unzip and rename

    (ii)    QC
    
    (iii)	ChIP-rx normalization"
