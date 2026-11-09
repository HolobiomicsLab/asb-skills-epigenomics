---
name: 4dn-portal-data-retrieval
description: Use when extending the DNALONGBENCH contact map prediction task beyond
  the five primary cell lines (HFF, H1hESC, GM12878, IMR90, HCT116) to new cell types
  whose Hi-C matrices are available on the 4DN portal.
when_to_use_negative:
- You only need the five primary cell lines (HFF, H1hESC, GM12878, IMR90, HCT116)
  already distributed in the benchmark's Contact Map Prediction deposit
- Your task is one of the other DNALONGBENCH tasks (enhancer-target gene, eQTL, regulatory
  sequence activity, transcription initiation) that do not use Hi-C contact maps
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3912
- http://edamontology.org/topic_0622
tools:
- name: conda
  role: Creates the dnalongbench Python 3.9 environment used to run the processing
    and serialization steps
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
- name: DNALONGBENCH
  role: Provides the load_data utility and the tfr reformulation convention (train*/valid*/test*.tfr)
    that the retrieved data must match
  repo: https://github.com/ma-compbio/DNALONGBENCH.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-65077-4
    title: 'DNALONGBENCH: a benchmark suite for long-range DNA prediction tasks'
  source_content_hash: sha256:4755854148ec75f860e8512547b76e788517727441144a68ee6d7dd629df71e3
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
  - DNALONGBENCH
  - conda
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3912
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# 4DN Portal Data Retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves raw Hi-C data for additional cell types (e.g., HAP1, HeLa, HepG2, K562) from the 4DN data portal and processes it into DNALONGBENCH-format 448x448 binned contact map targets using the Akita processing pipeline.

## When to use

Use when extending the DNALONGBENCH contact map prediction task beyond the five primary cell lines (HFF, H1hESC, GM12878, IMR90, HCT116) to new cell types whose Hi-C matrices are available on the 4DN portal.

## When NOT to use

- You only need the five primary cell lines (HFF, H1hESC, GM12878, IMR90, HCT116) already distributed in the benchmark's Contact Map Prediction deposit
- Your task is one of the other DNALONGBENCH tasks (enhancer-target gene, eQTL, regulatory sequence activity, transcription initiation) that do not use Hi-C contact maps

## Inputs

- Raw Hi-C data for additional cell lines (e.g., HAP1, HeLa, HepG2, K562) from the 4DN data portal
- Existing Contact Map Prediction deposit (https://doi.org/10.7910/DVN/AZM25S) as a format reference
- Akita Hi-C processing pipeline for binning raw matrices

## Outputs

- 448x448 binned contact map targets per cell line
- train*/valid*/test*.tfr TensorFlow record files per cell line
- Datasets loadable via dnalongbench.utils.load_data(task_name='contact_map_prediction')

## How to apply

First create the dnalongbench conda environment with Python 3.9 (conda create -n dnalongbench python=3.9 -y) and install the benchmark after cloning ma-compbio/DNALONGBENCH, so the output format can mirror the existing Contact Map Prediction deposit (https://doi.org/10.7910/DVN/AZM25S). Download the raw Hi-C data for the target cell lines from the 4DN data portal and process the raw Hi-C matrices with the Akita processing pipeline to produce 448x448 binned contact map targets per cell line. Split the processed contact maps into train/validation/test following the benchmark's well-split convention, then serialize each split as TensorFlow record files named train*/valid*/test*.tfr, mirroring how the benchmark reformulates its data (e.g., the HFF subset). Decision point: the benchmark's primary evaluation covers only five primary cell lines; additional cell types (as in Table S4) require this retrieval-and-reformulation path rather than the standard deposit download.

## Related tools

- **conda** (Creates the dnalongbench Python 3.9 environment used to run the processing and serialization steps) — https://github.com/ma-compbio/DNALONGBENCH.git
- **DNALONGBENCH** (Provides the load_data utility and the tfr reformulation convention (train*/valid*/test*.tfr) that the retrieved data must match) — https://github.com/ma-compbio/DNALONGBENCH.git

## Examples

```
train_loader, valid_loader, test_loader = load_data(root=root, task_name='contact_map_prediction', subset='HFF', batch_size=16)
```

## Evaluation signals

- Output .tfr files for all four target cell lines (train*/valid*/test*) exist on disk
- Files load successfully with dnalongbench.utils.load_data(task_name='contact_map_prediction') at batch_size=16
- Loaded targets match the 448x448 Akita contact-map dimensions
- The well-split convention for train/validation/test partitions is respected, mirroring the HFF subset structure
- Format matches the reference deposit at https://doi.org/10.7910/DVN/AZM25S

## Limitations

- Results for the additional cell types (HAP1, HeLa, HepG2, K562) are only reported separately in Table S4 of the benchmark; the main evaluation covers just five primary cell lines
- Raw Hi-C matrices must be reprocessed with the Akita pipeline, so binning choices are inherited from Akita rather than freely configurable

## Evidence

- [supplementary] Five primary cell lines evaluated; extras in S4: [evidence span withheld — anchor status: not_found]
- [methods] Reformulate into tfr files: "reformulate these data into train*/valid*/test*.tfr"
- [methods] load_data example for contact map prediction: "train_loader, valid_loader, test_loader = load_data(root=root, task_name = 'contact_map_prediction', subset='HFF', batch_size=16)"
- [methods] Conda environment creation for benchmark: "conda create -n dnalongbench python=3.9 -y"
