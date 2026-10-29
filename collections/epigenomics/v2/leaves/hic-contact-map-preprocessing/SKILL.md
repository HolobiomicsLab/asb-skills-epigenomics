---
name: hic-contact-map-preprocessing
description: Use when raw Hi-C matrices from new cell types must be reformatted into
  the DNALONGBENCH/Akita 448x448 contact map .tfr format so that load_data(task_name='contact_map_prediction')
  can consume them.
when_to_use_negative:
- The cell line already has .tfr files shipped with DNALONGBENCH (e.g., the HFF subset)
  — no reprocessing is needed, just load directly.
- The prediction task is not contact map prediction (e.g., enhancer-target gene or
  eQTL tasks), which use different input lengths (450,000 bp) and output shapes than
  the 448x448 contact maps.
tools:
- name: DNALONGBENCH
  role: Benchmark package providing the load_data utility and the .tfr data schema
    that the reconstructed contact maps must match
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: conda
  role: Creates the dnalongbench Python 3.9 environment in which the processing and
    validation scripts run
- name: Akita processing pipeline
  role: Converts raw Hi-C matrices into 448x448 binned contact map targets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:dd37b0c83d361a134c396db3d57ae76c50f2db1e5773843f82076fbbae7b760a
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
  - Akita processing pipeline
  - DNALONGBENCH
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hic-contact-map-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extends the DNALONGBENCH contact map prediction task beyond its five primary cell lines (HFF, H1hESC, GM12878, IMR90, HCT116) by downloading raw 4DN Hi-C data for additional cell lines (e.g., HAP1, HeLa, HepG2, K562) and converting it into Akita-style 448x448 binned contact map targets serialized as TensorFlow record (.tfr) files.

## When to use

Use when raw Hi-C matrices from new cell types must be reformatted into the DNALONGBENCH/Akita 448x448 contact map .tfr format so that load_data(task_name='contact_map_prediction') can consume them.

## When NOT to use

- The cell line already has .tfr files shipped with DNALONGBENCH (e.g., the HFF subset) — no reprocessing is needed, just load directly.
- The prediction task is not contact map prediction (e.g., enhancer-target gene or eQTL tasks), which use different input lengths (450,000 bp) and output shapes than the 448x448 contact maps.

## Inputs

- Raw Hi-C matrices for additional cell lines (e.g., HAP1, HeLa, HepG2, K562) from the 4DN data portal
- Existing Contact Map Prediction deposit (https://doi.org/10.7910/DVN/AZM25S) as a format reference

## Outputs

- 448x448 binned contact map targets per cell line, split into train/valid/test
- TensorFlow record files named train*/valid*/test*.tfr loadable by dnalongbench.utils.load_data(task_name='contact_map_prediction')

## How to apply

Create the dnalongbench conda environment with Python 3.9 (conda create -n dnalongbench python=3.9 -y) and install the package from ma-compbio/DNALONGBENCH. Download raw Hi-C data for the additional cell lines from the 4DN data portal and use the existing Contact Map Prediction deposit (https://doi.org/10.7910/DVN/AZM25S) as the format reference. Process the raw Hi-C matrices with the Akita processing pipeline to produce 448x448 binned contact map targets per cell line, then split them into train/valid/test sets following the benchmark's well-split convention. Serialize each split as TensorFlow record files named train*/valid*/test*.tfr, mirroring how DNALONGBENCH reformulates its per-cell-line subsets (e.g., HFF). The rationale is that the benchmark's loaders expect .tfr-shipped, 448x448 Akita-dimension targets, so any new cell type must be homogenized to that schema before model evaluation.

## Related tools

- **DNALONGBENCH** (Benchmark package providing the load_data utility and the .tfr data schema that the reconstructed contact maps must match) — https://github.com/ma-compbio/DNALONGBENCH.git
- **conda** (Creates the dnalongbench Python 3.9 environment in which the processing and validation scripts run)
- **Akita processing pipeline** (Converts raw Hi-C matrices into 448x448 binned contact map targets)

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- Output .tfr files named train*/valid*/test*.tfr exist for all four additional cell lines.
- Files load successfully via dnalongbench.utils.load_data(task_name='contact_map_prediction') at batch_size=16.
- Loaded targets match the 448x448 Akita contact-map output dimensions.
- Split partitions follow the benchmark's well-split convention and mirror the structure of the existing HFF subset.

## Limitations

- Contact map prediction is evaluated on five primary cell lines (HFF, H1hESC, GM12878, IMR90, HCT116); additional cell types (HAP1, HeLa, HepG2, K562) appear only as results reported separately in Table S4, so extended-cell-line results are supplementary rather than part of the core benchmark.
- The Contact Map Prediction data must be downloaded via the provided deposit link (https://doi.org/10.7910/DVN/AZM25S) or the Box mirror (https://cmu.box.com/s/cyn3tqfej3v4tg4xwv1god3jemq7916y); no local reprocessing path replaces these downloads for the reference format.

## Evidence
