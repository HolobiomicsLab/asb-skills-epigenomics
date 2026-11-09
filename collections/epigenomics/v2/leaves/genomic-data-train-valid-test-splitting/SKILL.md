---
name: genomic-data-train-valid-test-splitting
description: Use when raw task data downloaded for a DNALONGBENCH task must be partitioned
  and serialized into train*/valid*/test*.tfr files so downstream loaders and model
  training consume consistent splits.
when_to_use_negative:
- Input data has already been reformulated into train*/valid*/test*.tfr files — re-splitting
  would break reproducibility of the benchmark's well-split convention
- The task is not one of the DNALONGBENCH long-range tasks (e.g., short-range DNA
  benchmarks like Genomic Benchmarks, NT Benchmark, or GUE) since their data formats
  and split conventions differ
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: conda
  role: Creates the dnalongbench Python 3.9 environment used to run the reformulation
    and loading scripts
- name: DNALONGBENCH
  role: Benchmark package providing the data reformulation utilities and the load_data
    loader for the .tfr splits
  repo: https://github.com/ma-compbio/DNALONGBENCH
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:6d68959964458a330dddfb4a629d14039706890e9864f90519717a6c330cc332
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/ma-compbio/DNALONGBENCH
  tools:
  - DNALONGBENCH
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genomic-data-train-valid-test-splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reformulates downloaded benchmark task data into train/valid/test TensorFlow record (.tfr) files so that DNALONGBENCH tasks (e.g., contact_map_prediction) can be loaded with a fixed, reproducible split convention via load_data.

## When to use

Use when raw task data downloaded for a DNALONGBENCH task must be partitioned and serialized into train*/valid*/test*.tfr files so downstream loaders and model training consume consistent splits.

## When NOT to use

- Input data has already been reformulated into train*/valid*/test*.tfr files — re-splitting would break reproducibility of the benchmark's well-split convention
- The task is not one of the DNALONGBENCH long-range tasks (e.g., short-range DNA benchmarks like Genomic Benchmarks, NT Benchmark, or GUE) since their data formats and split conventions differ

## Inputs

- Downloaded DNALONGBENCH task data (e.g., Contact Map Prediction deposit https://doi.org/10.7910/DVN/AZM25S, or task data via the provided links)
- Raw Hi-C matrices processed with the Akita pipeline into 448x448 binned contact map targets (for contact map task extensions)
- Task name and subset identifiers (e.g., task_name='contact_map_prediction', subset='HFF')

## Outputs

- TensorFlow record files named train*/valid*/test*.tfr per task subset
- Loadable split loaders: (train_loader, valid_loader, test_loader) returned by load_data

## How to apply

Create the dnalongbench conda environment with Python 3.9 and install the ma-compbio/DNALONGBENCH package after cloning it. Download the task-specific deposit (e.g., https://doi.org/10.7910/DVN/AZM25S for Contact Map Prediction) and apply the benchmark's reformulation step to split the data into train, validation, and test subsets, serializing each as TensorFlow record files named train*/valid*/test*.tfr (as done, e.g., for the HFF subset). Follow the benchmark's well-split convention when assigning samples to splits so subsets remain comparable to published results. After writing the .tfr files, verify them by loading with dnalongbench's load_data(root=root, task_name=..., subset=..., batch_size=16) and checking the tensor shapes match the task's output specification (e.g., 448x448 binned contact maps for contact map prediction).

## Related tools

- **conda** (Creates the dnalongbench Python 3.9 environment used to run the reformulation and loading scripts)
- **DNALONGBENCH** (Benchmark package providing the data reformulation utilities and the load_data loader for the .tfr splits) — https://github.com/ma-compbio/DNALONGBENCH

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- The output directory contains train*/valid*/test*.tfr files for every processed cell line or subset (e.g., HFF and any additional cell lines such as HAP1, HeLa, HepG2, K562)
- load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16) returns three loaders without error
- Targets loaded from the .tfr files match the expected task output shape (448x448 Akita contact-map dimensions for the contact map task)
- Split membership follows the benchmark's well-split convention, so metrics remain comparable to results reported in the DNALONGBENCH paper (including Table S4 for additional cell types)

## Limitations

- Task data must be downloaded via the provided links (or Box mirror) first; the splitting step cannot run without the deposit
- Model performance varies substantially across tasks, so split-level results are not comparable across different DNALONGBENCH tasks
- The paper's task table is truncated in the extracted text, so per-task sample counts (e.g., 33,521 | PCC rows) may be incomplete when validating split sizes

## Evidence
