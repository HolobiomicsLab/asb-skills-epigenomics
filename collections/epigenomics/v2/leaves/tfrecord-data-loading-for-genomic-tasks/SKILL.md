---
name: tfrecord-data-loading-for-genomic-tasks
description: Use when you need to feed DNALONGBENCH benchmark data (e.g., contact
  map prediction .tfr files for a specific cell line like HFF) into a model training
  or inference loop as batched train/valid/test loaders.
when_to_use_negative:
- Your data is not in the DNALONGBENCH .tfr format and has not been reformulated into
  train*/valid*/test*.tfr files — load_data expects this structure.
- You are working with a different benchmark (e.g., BEND, LRB, Genomic Benchmarks)
  whose data layout and loaders differ from DNALONGBENCH.
- You need a single merged dataset across all cell lines rather than a per-subset
  loader (load_data is called per subset, e.g., subset='HFF').
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: DNALONGBENCH
  role: Provides the load_data utility that reads .tfr files and returns train/valid/test
    loaders for each task and subset
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Creates the dnalongbench Python 3.9 environment in which the package and load_data
    utility are installed
- name: HyenaDNA
  role: Downstream consumer of the loaded DataLoaders when fine-tuning on tasks such
    as contact map prediction
  repo: https://github.com/HazyResearch/hyena-dna.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:09425f89ff9753bf3ce2d57c4ea81fd3967f478e9a10863806f1e6fb5ff243d9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ma-compbio/DNALONGBENCH
  repo_url: https://github.com/ma-compbio/DNALONGBENCH.git
  tools:
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# tfrecord-data-loading-for-genomic-tasks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loads DNALONGBENCH task data stored as train/valid/test TFRecord (.tfr) files into PyTorch train, validation, and test DataLoaders via the dnalongbench load_data utility, enabling consistent batching for tasks like contact map prediction across cell line subsets.

## When to use

Use when you need to feed DNALONGBENCH benchmark data (e.g., contact map prediction .tfr files for a specific cell line like HFF) into a model training or inference loop as batched train/valid/test loaders.

## When NOT to use

- Your data is not in the DNALONGBENCH .tfr format and has not been reformulated into train*/valid*/test*.tfr files — load_data expects this structure.
- You are working with a different benchmark (e.g., BEND, LRB, Genomic Benchmarks) whose data layout and loaders differ from DNALONGBENCH.
- You need a single merged dataset across all cell lines rather than a per-subset loader (load_data is called per subset, e.g., subset='HFF').

## Inputs

- DNALONGBENCH task data reformulated into train*/valid*/test*.tfr TFRecord files
- Task name string (e.g., 'contact_map_prediction')
- Cell line subset name (e.g., 'HFF')
- Batch size (e.g., 16)

## Outputs

- train_loader (PyTorch DataLoader of training batches)
- valid_loader (PyTorch DataLoader of validation batches)
- test_loader (PyTorch DataLoader of test batches)

## How to apply

After installing the ma-compbio/DNALONGBENCH package in the dnalongbench conda environment (Python 3.9), ensure the task data has been downloaded (e.g., Contact Map Prediction from Harvard Dataverse DOI 10.7910/DVN/AZM25S) and reformulated into train*/valid*/test*.tfr files. Call the dnalongbench load_data utility with the dataset root, the task name (e.g., task_name='contact_map_prediction'), the target subset (e.g., subset='HFF' for one of the nine cell lines), and a batch size (the paper uses batch_size=16). The function returns three loaders — train_loader, valid_loader, and test_loader — which are then used to train the CNN baseline or fine-tune HyenaDNA and Caduceus models, or to run inference with the Akita expert model. Verify by checking that each loader yields batches with the expected input length (e.g., up to 1,000,000 bp sequences) and output shape for the chosen task, and that the split assignment matches the well-split files provided in the deposit.

## Related tools

- **DNALONGBENCH** (Provides the load_data utility that reads .tfr files and returns train/valid/test loaders for each task and subset) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Creates the dnalongbench Python 3.9 environment in which the package and load_data utility are installed)
- **HyenaDNA** (Downstream consumer of the loaded DataLoaders when fine-tuning on tasks such as contact map prediction) — https://github.com/HazyResearch/hyena-dna.git

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- load_data returns exactly three objects (train_loader, valid_loader, test_loader) without error for the requested task_name and subset.
- Batches from the loaders match the task's expected shapes, e.g., contact map prediction input length of 450,000+ bp with 2D contact map outputs at batch_size=16.
- The loaded splits correspond to the well-split train/valid/test files from the Harvard Dataverse deposit (DOI 10.7910/DVN/AZM25S), with no leakage between splits.
- Per-cell-line subsets (e.g., HFF) load independently and counts match the # Samples documented in the benchmark's task table.
- Downstream training or inference runs (CNN baseline, HyenaDNA, Caduceus, Akita expert) consume the loaders without shape or dtype errors.

## Limitations

- Data must first be downloaded via the provided task-specific links and reformulated into train*/valid*/test*.tfr files; raw deposits may not be directly loadable.
- The paper's task table is truncated in some sources, so verifying expected sample counts per subset may require consulting the README or Dataverse deposit directly.

## Evidence

- [other] Load each cell line subset (e.g. HFF) with the dnalongbench load_data utility (task_name='contact_map_prediction', batch_size=16) to obtain train, validation, and test loaders.: [evidence span withheld — anchor status: not_found]
- [other] Download the Contact Map Prediction dataset from the Harvard Dataverse deposit (DOI 10.7910/DVN/AZM25S), which provides well-split train/valid/test files.: [evidence span withheld — anchor status: not_found]
