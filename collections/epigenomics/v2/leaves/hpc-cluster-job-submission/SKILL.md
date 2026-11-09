---
name: hpc-cluster-job-submission
description: Use when a ChIA-PIPE (or similar) pipeline run is too large for an interactive
  shell — e.g., 200–400 million read-pair libraries from ENCODE4/4DN — and a PBS-scheduler
  cluster is available.
when_to_use_negative:
- Small test datasets that complete comfortably in an interactive shell via 0.chia_pipe_shell.sh
  — PBS submission adds scheduler overhead without benefit
- Clusters that do not use the PBS scheduler (qsub -F flags and .pbs scripts assume
  PBS)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0607
tools:
- name: ChIA-PIPE
  role: Pipeline being submitted to the cluster; 0.chia_pipe_hpc.pbs is its PBS entry
    point
  repo: https://github.com/TheJacksonLaboratory/ChIA-PIPE
- name: CPU (ChIA-PET Utilities)
  role: Modularized executables integrated by ChIA-PIPE that run within the submitted
    job
  repo: https://github.com/cheehongsg/CPU
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_006
  source_papers:
  - doi: 10.1126/sciadv.aay2078
    title: 'ChIA-PIPE: A fully automated pipeline for comprehensive ChIA-PET data
      analysis and visualization'
  source_content_hash: sha256:7ddae0195b39ad9a4d5e9b18e6951d7ca065f5a29afd29c3fc8e753e332de078
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TheJacksonLaboratory/ChIA-PIPE
  tools:
  - BASIC Browser
  - CPU (ChIA-PET Utilities)
  - ChIA-PIPE
  - HiGlass
  - Juicebox
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# hpc-cluster-job-submission

## Summary

Run large-scale ChIA-PET pipelines (e.g., ChIA-PIPE) on HPC clusters by submitting a PBS job script with a configuration file, so libraries of hundreds of millions of PET read pairs can be processed beyond typical workstation capacity.

## When to use

Use when a ChIA-PIPE (or similar) pipeline run is too large for an interactive shell — e.g., 200–400 million read-pair libraries from ENCODE4/4DN — and a PBS-scheduler cluster is available.

## When NOT to use

- Small test datasets that complete comfortably in an interactive shell via 0.chia_pipe_shell.sh — PBS submission adds scheduler overhead without benefit
- Clusters that do not use the PBS scheduler (qsub -F flags and .pbs scripts assume PBS)

## Inputs

- ChIA-PET FASTQ read-pair files (e.g., LDK0004-ds_*.fastq.gz in a fastq/ directory)
- ChIA-PIPE configuration file (e.g., example_config_file.sh)
- ChIA-PIPE PBS job script (0.chia_pipe_hpc.pbs)

## Outputs

- Called peaks and chromatin interaction loops
- Automated QC assessment outputs
- .hic and .cool contact map files for Juicebox and HiGlass

## How to apply

Clone ChIA-PIPE and install dependencies locally with local_install_chia_pipe_dependencies.sh -i ${dep_dir}, then place LDK0004-ds_*.fastq.gz test reads into a fastq/ subdirectory as required by the configuration. Instead of launching 0.chia_pipe_shell.sh directly, submit the PBS wrapper via qsub with the config file passed through the -F/--conf flag so the scheduler manages resource allocation. ChIA-PIPE demonstrably completed ~300 million read pairs of an HFFc6 CTCF HiSeq library in 10 hours wall-clock time using 20 threads and 60 Gb RAM, so provision at least that scale of threads and memory per job. The PBS route executes the same linker filtering, read mapping (hybrid BWA-ALN/BWA-MEM), peak calling, and loop calling stages as the shell route, so outputs are equivalent.

## Related tools

- **ChIA-PIPE** (Pipeline being submitted to the cluster; 0.chia_pipe_hpc.pbs is its PBS entry point) — https://github.com/TheJacksonLaboratory/ChIA-PIPE
- **CPU (ChIA-PET Utilities)** (Modularized executables integrated by ChIA-PIPE that run within the submitted job) — https://github.com/cheehongsg/CPU

## Examples

```
qsub -F "--conf chia_pipe-master/example_config_file.sh" chia_pipe-master/0.chia_pipe_hpc.pbs
```

## Evaluation signals

- PBS job completes (qsub job exits successfully) within an expected wall-clock range — reference: ~10 hours for ~300M read pairs at 20 threads and 60 Gb RAM
- Peak and loop call outputs are produced alongside the automated QC assessment
- Valid .hic and .cool files are generated and render in Juicebox and HiGlass
- Running the same config through 0.chia_pipe_shell.sh yields comparable outputs, confirming the HPC wrapper did not alter pipeline stages

## Limitations

- The PBS script and qsub -F flag syntax are PBS-specific; other schedulers (SLURM, SGE) would require adapting the job script
- Wall-clock and memory needs scale with library size; the 10 h / 20-thread / 60 Gb benchmark was for one HFFc6 CTCF HiSeq library (~300M read pairs)

## Evidence

- [other] ChIA-PIPE processed an HFFc6 CTCF HiSeq library of ~300 million read pairs, detecting peaks and chromatin interactions, in a wall run time of 10 hours using 20 threads and 60 Gb RAM.: [evidence span withheld — anchor status: not_found]
- [other] Alternatively, for HPC execution, submit the PBS script via 'qsub -F "--conf chia_pipe-master/example_config_file.sh" chia_pipe-master/0.chia_pipe_hpc.pbs'.: [evidence span withheld — anchor status: not_found]
