---
name: hdbscan-density-clustering
description: Use when you have low-dimensional node embeddings (e.g. 16-dim from a
  2-layer GAE with hidden_dim=64) per genomic bin and need unsupervised, density-adaptive
  labels without specifying k a priori.
when_to_use_negative:
- Input is the raw NxN intra-chromosomal Hi-C contact matrix — cluster the GAE embeddings
  instead
- The downstream task requires TADs directly; this skill only assigns per-bin labels
  — contiguous same-label regions still need the 200 Kb minimum-length filter
- The number of clusters is known a priori and must be fixed (HDBSCAN discovers clusters
  of varying densities automatically and cannot enforce k)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0128
tools:
- name: HDBSCAN
  role: Unsupervised density-based clustering algorithm applied to GAE embeddings
    to label each genomic bin
- name: hdbscan
  role: Python package implementing HDBSCAN used by TOAST's clustering module
- name: TOAST
  role: Pipeline whose clustering module coordinates the generated embeddings and
    labels each bin
  repo: https://github.com/ghaiyan/TOAST
- name: numpy
  role: Load and manipulate the embedding matrix before clustering
- name: pandas
  role: Write bin-label records to bin_labels.csv
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:4f50f513ad653df62ef2b19ec7d6b2d7e4175e9b4c14f7fc693303afdef33e2a
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
  - HDBSCAN
  - Juicer
  - NumPy
  - TOAST
  - hdbscan
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# hdbscan-density-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Unsupervised density-based clustering of 16-dimensional GAE node embeddings with HDBSCAN to assign a cluster label to every genomic bin, the step in TOAST that precedes TAD delineation from contiguous same-label regions.

## When to use

Use when you have low-dimensional node embeddings (e.g. 16-dim from a 2-layer GAE with hidden_dim=64) per genomic bin and need unsupervised, density-adaptive labels without specifying k a priori.

## When NOT to use

- Input is the raw NxN intra-chromosomal Hi-C contact matrix — cluster the GAE embeddings instead
- The downstream task requires TADs directly; this skill only assigns per-bin labels — contiguous same-label regions still need the 200 Kb minimum-length filter
- The number of clusters is known a priori and must be fixed (HDBSCAN discovers clusters of varying densities automatically and cannot enforce k)

## Inputs

- 16-dimensional node embedding matrix Z per 400x400 Hi-C submatrix (output of the 2-layer GAE, hidden_dim=64, lr=0.001, 200 epochs, MSE loss)
- Submatrix/bin metadata mapping each embedding row to a genomic bin and chromosome

## Outputs

- bin_labels.csv with exactly one HDBSCAN cluster label per genomic bin (columns: bin index, chromosome/submatrix identifier, cluster label)
- Noise flags (label -1) for low-density bins, per HDBSCAN's built-in extraction

## How to apply

Load the 16-dimensional embedding matrix Z produced by the trained GAE encoder for each 400x400 Hi-C submatrix using numpy/pandas. Run the hdbscan package's HDBSCAN in unsupervised mode on Z so the algorithm builds a density hierarchy automatically, pruning the condensed hierarchy and flagging low-density bins as noise. Collect one label per bin (bin index, chromosome/submatrix identifier, label) into bin_labels.csv, one row per bin. Rationale: TOAST's sweep across six GAE variants showed clustering 16-dim embeddings (hidden_dim=64) yields the most robust TAD identification, so embeddings — not raw contact matrices — are the correct clustering input.

## Related tools

- **HDBSCAN** (Unsupervised density-based clustering algorithm applied to GAE embeddings to label each genomic bin)
- **hdbscan** (Python package implementing HDBSCAN used by TOAST's clustering module)
- **TOAST** (Pipeline whose clustering module coordinates the generated embeddings and labels each bin) — https://github.com/ghaiyan/TOAST
- **numpy** (Load and manipulate the embedding matrix before clustering)
- **pandas** (Write bin-label records to bin_labels.csv)

## Examples

```
import hdbscan, numpy as np; Z = np.loadtxt('embeddings.csv', delimiter='\t'); labels = hdbscan.HDBSCAN().fit_predict(Z)
```

## Evaluation signals

- bin_labels.csv exists, is parseable, and contains exactly one label per genomic bin of each 400x400 submatrix
- Every embedding row received a label or an explicit noise flag (-1), matching the requirement that HDBSCAN assigns a label to every bin
- Clustering is run on 16-dimensional embeddings from the hidden_dim=64/output_dim=16 GAE, not on the contact matrix or higher-dim variants
- Downstream contiguous same-label regions (kept if > 200 Kb, i.e. ≥4 bins at 50 Kb resolution) yield a plausible TAD count (GM12878 50 Kb average 4736) and TAD_quality ~1654.03
- On simulated Trussart data, this configuration attains the highest Fowlkes-Mallows score / Rand Index versus ground-truth TADs (FMS ≈ 0.98 even at high noise)

## Limitations

- HDBSCAN flags low-density embedding points as noise; those bins receive no cluster and are excluded from TAD formation
- Label quality depends on the upstream GAE architecture: hidden_dim=64/output_dim=16 was selected as optimal/robust; other configurations underperform
- Noise levels in simulated Hi-C matrices affect FMS (it decreases with noise level), so heavily degraded contact data may degrade cluster labels

## Evidence
