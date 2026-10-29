---
name: genomic-bed-data-loading
description: Use when reproducing a DNALONGBENCH long-range task (eQTL, RSAP, TISP,
  contact map, enhancer-target gene) and raw bed-file data downloaded from a Dataverse
  DOI must be converted into train/valid/test loaders for model training and inference.
when_to_use_negative:
- The analysis targets a short-range DNA benchmark (e.g. Genomic Benchmarks, NT Benchmark,
  GUE, BEND, LRB) rather than a DNALONGBENCH long-range task — the splits and tfr
  reformulation are specific to this benchmark.
- The model is a pretrained DNA foundation model used zero-shot for generation (e.g.
  GENERator sequence generation), which does not consume the supervised train/valid/test
  loaders.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dnalongbench.utils.load_data
  role: Core loading utility that converts reformulated task data into train/valid/test
    loaders given root, task_name, subset, and batch_size
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Creates the dnalongbench Python 3.9 environment in which the package and loaders
    are installed
- name: DNALONGBENCH
  role: Benchmark package providing the five long-range tasks, their Dataverse data
    deposits, and the load_data API
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:1cafd16d783d6f80a4cef8158f7325aa03902f9c3d99dda289d9dad38327b42f
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
  - CNN
  - Caduceus-PS
  - Caduceus-Ph
  - DNALONGBENCH
  - HyenaDNA
  - conda
  - dnalongbench.utils.load_data
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# genomic-bed-data-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load DNALONGBENCH long-range genomic prediction task data (bed-file-derived eQTL, TISP, RSAP sets distributed via Dataverse DOIs) into train/valid/test PyTorch loaders using dnalongbench.utils.load_data, so that CNN, HyenaDNA, and Caduceus models can be fine-tuned and evaluated on the correct held-out splits.

## When to use

Use when reproducing a DNALONGBENCH long-range task (eQTL, RSAP, TISP, contact map, enhancer-target gene) and raw bed-file data downloaded from a Dataverse DOI must be converted into train/valid/test loaders for model training and inference.

## When NOT to use

- The analysis targets a short-range DNA benchmark (e.g. Genomic Benchmarks, NT Benchmark, GUE, BEND, LRB) rather than a DNALONGBENCH long-range task — the splits and tfr reformulation are specific to this benchmark.
- The model is a pretrained DNA foundation model used zero-shot for generation (e.g. GENERator sequence generation), which does not consume the supervised train/valid/test loaders.

## Inputs

- Bed files for a DNALONGBENCH task downloaded from a Dataverse DOI (e.g. eQTL bed files at https://doi.org/10.7910/DVN/YUP2G5)
- Task data reformulated into train*/valid*/test*.tfr TensorFlow-record files
- Installed dnalongbench package from ma-compbio/DNALONGBENCH

## Outputs

- train_loader, valid_loader, and test_loader objects from dnalongbench.utils.load_data (batch_size=16)
- Batched long-range DNA sequences (up to 450,000 bp for eQTL) with per-task labels for model fine-tuning and held-out evaluation

## How to apply

First create the dnalongbench conda environment (Python 3.9), clone ma-compbio/DNALONGBENCH, and install it with 'pip install .'. Download the task-specific data from its Dataverse DOI (e.g. 10.7910/DVN/YUP2G5 for eQTL bed files, 10.7910/DVN/VXQKWO for TISP, 10.7910/DVN/MNUEZR for RSAP) and reformulate the bed-derived data into train*/valid*/test*.tfr TensorFlow-record files where required. Call the load_data utility specifying the task_name (e.g. 'eqtl'), subset where applicable, and batch_size=16 to obtain train_loader, valid_loader, and test_loader. Rationale: the benchmark's tasks span up to 1,000,000 bp inputs (e.g. 450,000 bp for eQTL), so loading must preserve the predefined chromosome/split structure that later evaluation on held-out test chromosomes (e.g. chr8, chr9 for TISP) depends on.

## Related tools

- **dnalongbench.utils.load_data** (Core loading utility that converts reformulated task data into train/valid/test loaders given root, task_name, subset, and batch_size) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Creates the dnalongbench Python 3.9 environment in which the package and loaders are installed)
- **DNALONGBENCH** (Benchmark package providing the five long-range tasks, their Dataverse data deposits, and the load_data API) — https://github.com/ma-compbio/DNALONGBENCH.git

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='eqtl', subset=None, batch_size=16)
```

## Evaluation signals

- load_data returns three non-empty loaders (train_loader, valid_loader, test_loader) for the requested task_name and batch_size (16).
- Reformulated data directory contains the expected train*/valid*/test*.tfr files before load_data is called.
- Sample counts match the benchmark table (e.g. 31,282 samples for eQTL with 450,000-bp input and binary output).
- Downstream evaluation on the held-out test split (e.g. chr8/chr9 chromosomes for TISP) is only meaningful if the test_loader was never mixed into training data.
- Per-model results CSVs (eqtl_auroc_results.csv, rsap_pcc_scores.csv, tisp_pcc_results.csv) can only be reproduced if the same loaders fed every model.

## Limitations

- Transformer-based DNA foundation models are infeasible for these tasks due to the quadratic cost of self-attention on the long input sequences.
- The paper's methods note that task data must be reformulated into .tfr files before loading; skipping this step for tasks distributed as raw bed files will break load_data.
- The DNALONGBENCH citation block and task table in the methods are incomplete (truncated rows), so some task metadata must be confirmed from the README or Dataverse deposits.

## Evidence
