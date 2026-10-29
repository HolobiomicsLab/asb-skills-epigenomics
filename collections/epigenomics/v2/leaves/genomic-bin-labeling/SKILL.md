---
name: genomic-bin-labeling
description: Use when you already have per-bin node embeddings from a GAE trained
  on 400×400 Hi-C submatrices and need an unsupervised cluster label per genomic bin
  to delineate TAD boundaries.
when_to_use_negative:
- Input is raw Hi-C contact counts rather than trained GAE embeddings — clustering
  raw matrices is not what this skill does
- The goal is supervised bin classification with known labels; HDBSCAN here runs in
  unsupervised mode and cannot use ground-truth labels
- The analysis targets features other than genomic bins (e.g. EPI pair prediction
  in EPIANN), where per-bin TAD labeling is irrelevant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: pipeline whose GAE module produces the 16-dimensional embeddings that this
    clustering step consumes and whose TAD generation module consumes the labels
  repo: https://github.com/ghaiyan/TOAST
- name: hdbscan
  role: Python package providing the HDBSCAN unsupervised clustering algorithm that
    assigns a label to each genomic bin embedding
  repo: https://github.com/ghaiyan/TOAST
- name: numpy
  role: loads and arrays the 16-dimensional embedding matrix Z passed to HDBSCAN
- name: pandas
  role: writes one-row-per-bin cluster labels to bin_labels.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:b2d162b1bfbbca26ea0ce30c9ac28d567bb248e9a144af7b5adfb5b34fbba8bf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - TOAST
  - hdbscan
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genomic-bin-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Labels each genomic bin of a Hi-C contact matrix by running unsupervised HDBSCAN on the 16-dimensional graph auto-encoder (GAE) embeddings that TOAST derives from non-overlapping 400×400 submatrices; contiguous regions sharing a label (and longer than 200 Kb) become TADs.

## When to use

Use when you already have per-bin node embeddings from a GAE trained on 400×400 Hi-C submatrices and need an unsupervised cluster label per genomic bin to delineate TAD boundaries.

## When NOT to use

- Input is raw Hi-C contact counts rather than trained GAE embeddings — clustering raw matrices is not what this skill does
- The goal is supervised bin classification with known labels; HDBSCAN here runs in unsupervised mode and cannot use ground-truth labels
- The analysis targets features other than genomic bins (e.g. EPI pair prediction in EPIANN), where per-bin TAD labeling is irrelevant

## Inputs

- 16-dimensional GAE node embedding matrix Z (one row per genomic bin) from 400×400 Hi-C submatrices (hidden_dim=64, output_dim=16)
- NxN intra-chromosomal Hi-C contact matrix in tab-separated format (e.g. dumped from .hic via juicer_tools) from which bins derive

## Outputs

- bin_labels.csv with one cluster label per genomic bin (bin index, chromosome/submatrix identifier, cluster label), noise-flagged bins included
- Cluster labels feeding the TAD generation module (contiguous same-label regions > 200 Kb reported as TADs)

## How to apply

Load the 16-dimensional node embedding matrix Z produced by the trained GAE encoder (trained on non-overlapping 400×400 Hi-C submatrices with a 2-layer network, hidden layer dimension 64). Run the hdbscan package's HDBSCAN algorithm in unsupervised mode on Z, letting it discover clusters of varying density as a hierarchical tree and flag low-density bins as noise. Collect one label per genomic bin (bin index, chromosome/submatrix identifier, cluster label) with pandas. In downstream TAD generation, contiguous regions with the same label are identified as TADs only if their length exceeds 200 Kb. The unsupervised setup is deliberate: no cluster-count parameter is needed, which supports TOAST's goal of avoiding complex parameter settings.

## Related tools

- **TOAST** (pipeline whose GAE module produces the 16-dimensional embeddings that this clustering step consumes and whose TAD generation module consumes the labels) — https://github.com/ghaiyan/TOAST
- **hdbscan** (Python package providing the HDBSCAN unsupervised clustering algorithm that assigns a label to each genomic bin embedding) — https://github.com/ghaiyan/TOAST
- **numpy** (loads and arrays the 16-dimensional embedding matrix Z passed to HDBSCAN)
- **pandas** (writes one-row-per-bin cluster labels to bin_labels.csv)

## Examples

```
import pandas as pd, numpy as np, hdbscan; Z = np.load('embeddings.npy'); labels = hdbscan.HDBSCAN().fit_predict(Z); pd.DataFrame({'bin': range(len(labels)), 'label': labels}).to_csv('bin_labels.csv', index=False)
```

## Evaluation signals

- bin_labels.csv exists, is parseable with pandas, and contains exactly one label per genomic bin of each 400×400 submatrix
- HDBSCAN run in unsupervised mode (no cluster count supplied) and low-density points are flagged as noise per the algorithm's built-in condensed-hierarchy pruning
- Downstream contiguous same-label regions are filtered to those exceeding 200 Kb before being called TADs
- On Wang et al. simulated Hi-C data, the full TOAST pipeline (including this labeling step) achieves FMS around 0.98 across noise levels, so labeled TADs should closely match ground truth
- Robust default embedding geometry: 2-layer GAE with hidden layer dimension 64 and embedding dimension 16, per the paper's hyperparameter search

## Limitations

- HDBSCAN may flag low-density bins as noise, leaving some bins unlabeled; the pipeline relies on the algorithm's built-in condensed-hierarchy pruning rather than forcing assignments
- Optimal TAD identification was tied to a specific architecture (2-layer GAE, embedding dimension 16, hidden layer 64); other configurations (e.g. single-layer with output_dim=128) can perform comparably but were not chosen as most robust
- Contiguous same-label regions shorter than 200 Kb are not accepted as TADs, so short domains are dropped by design

## Evidence
