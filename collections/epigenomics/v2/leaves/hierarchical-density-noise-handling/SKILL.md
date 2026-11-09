---
name: hierarchical-density-noise-handling
description: Use when you have a dense node-embedding matrix (e.g. 16-dim per bin
  from a GAE encoder) and need per-point cluster labels without pre-specifying the
  number of clusters, including density-varying structure and noise points.
when_to_use_negative:
- The number of clusters is known in advance and you need centroid-based assignment
  rather than density-based hierarchical extraction
- Input is the raw Hi-C contact matrix rather than a trained GAE embedding — TOAST
  applies HDBSCAN only to the embeddings, not the 400x400 submatrices directly
- Every point must be assigned a meaningful cluster (HDBSCAN will flag low-density
  points as noise, which here translates to unclustered genomic bins)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3678
tools:
- name: HDBSCAN
  role: Unsupervised density-based clustering algorithm applied to the GAE embeddings;
    assigns a label to each genomic bin and marks low-density bins as noise
  repo: https://github.com/ghaiyan/TOAST
- name: hdbscan
  role: Python package providing HDBSCAN; installed via 'pip install hdbscan' per
    the TOAST README
  repo: https://github.com/ghaiyan/TOAST
- name: numpy
  role: Loads and manipulates the 16-dimensional embedding matrix Z prior to clustering
  repo: https://github.com/ghaiyan/TOAST
- name: pandas
  role: Writes the per-bin cluster labels to bin_labels.csv, one row per bin
  repo: https://github.com/ghaiyan/TOAST
- name: TOAST
  role: 'Pipeline in which this clustering step sits: GAE embeddings from 400x400
    submatrices are clustered and contiguous same-label regions >200 Kb become TADs'
  repo: https://github.com/ghaiyan/TOAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:dc9d7fd040eeaec338cb0369856645a0530e16e82293bd0adf6c0af16156f5dc
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
  - TOAST
  - hdbscan
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hierarchical-density-noise-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns a cluster label to every genomic bin by running HDBSCAN in unsupervised mode on 16-dimensional GAE embeddings derived from 400x400 Hi-C submatrices, letting the hierarchical density-based algorithm automatically discover clusters of varying densities and flag low-density bins as noise. This is the clustering step of TOAST that turns continuous embeddings into discrete labels from which TADs are delineated.

## When to use

Use when you have a dense node-embedding matrix (e.g. 16-dim per bin from a GAE encoder) and need per-point cluster labels without pre-specifying the number of clusters, including density-varying structure and noise points.

## When NOT to use

- The number of clusters is known in advance and you need centroid-based assignment rather than density-based hierarchical extraction
- Input is the raw Hi-C contact matrix rather than a trained GAE embedding — TOAST applies HDBSCAN only to the embeddings, not the 400x400 submatrices directly
- Every point must be assigned a meaningful cluster (HDBSCAN will flag low-density points as noise, which here translates to unclustered genomic bins)

## Inputs

- 16-dimensional GAE node embedding matrix Z per 400x400 Hi-C submatrix (hidden layer dimension 64, 2-layer network)
- Genomic bin index / chromosome or submatrix identifier table

## Outputs

- bin_labels.csv with one row per genomic bin (bin index, chromosome/submatrix identifier, cluster label), including noise labels
- Per-bin cluster labels used to delineate contiguous same-label regions as candidate TADs

## How to apply

Load the 16-dimensional node embedding matrix Z output by the trained GAE encoder for each non-overlapping 400x400 Hi-C submatrix using pandas and numpy. Run the hdbscan package's HDBSCAN algorithm in unsupervised mode on Z, which builds a hierarchical tree of clusters of varying densities and extracts clusters by pruning the condensed hierarchy; do not pre-set the cluster count. Allow HDBSCAN to label low-density points as noise per its built-in procedure, then collect one label per genomic bin (bin index, chromosome/submatrix identifier, cluster label) and write bin_labels.csv with one row per bin. Downstream, contiguous genomic regions sharing the same label are treated as candidate TADs, and in TOAST a contiguous region is confirmed as a TAD only if its length exceeds 200 Kb. The rationale is that HDBSCAN requires no metric parameter configuration, unlike prior methods (Armatus, Arrowhead, TopDom, TADtree, PSYCHIC, TADbit) that the article criticizes for complex parameter settings.

## Related tools

- **HDBSCAN** (Unsupervised density-based clustering algorithm applied to the GAE embeddings; assigns a label to each genomic bin and marks low-density bins as noise) — https://github.com/ghaiyan/TOAST
- **hdbscan** (Python package providing HDBSCAN; installed via 'pip install hdbscan' per the TOAST README) — https://github.com/ghaiyan/TOAST
- **numpy** (Loads and manipulates the 16-dimensional embedding matrix Z prior to clustering) — https://github.com/ghaiyan/TOAST
- **pandas** (Writes the per-bin cluster labels to bin_labels.csv, one row per bin) — https://github.com/ghaiyan/TOAST
- **TOAST** (Pipeline in which this clustering step sits: GAE embeddings from 400x400 submatrices are clustered and contiguous same-label regions >200 Kb become TADs) — https://github.com/ghaiyan/TOAST

## Examples

```
import numpy, pandas, hdbscan; Z = numpy.load('embeddings.npy'); labels = hdbscan.HDBSCAN().fit_predict(Z); pandas.DataFrame({'bin': range(len(labels)), 'label': labels}).to_csv('bin_labels.csv', index=False)
```

## Evaluation signals

- bin_labels.csv exists, is parseable with pandas, and contains exactly one label per genomic bin of each 400x400 submatrix
- Every bin receives either a cluster label or the noise label (HDBSCAN's built-in noise flagging), with no missing values
- Contiguous regions sharing the same label form candidate TADs; only regions longer than 200 Kb should be kept as TADs
- Downstream TAD calls reproduce the paper's validation behavior (e.g. FMS around 0.98 on Wang et al. simulated data, TAD number difference 1-8 vs ground truth)
- Runs reproduce with the pinned environment from the README (Python 3.6.12, Numpy 1.19.5, Pandas 1.1.5, hdbscan installed via pip)

## Limitations

- HDBSCAN flags low-density bins as noise, so some genomic bins receive no cluster and cannot contribute to TAD delineation
- Cluster structure depends on embedding quality: the optimal result required a 2-layer GAE with embedding dimension 16 and hidden layer dimension 64; other configurations (e.g. single-layer output_dim=128) performed differently
- TOAST applies HDBSCAN to non-overlapping 400x400 submatrices partitioned along the diagonal, so bin labels are only valid within each submatrix's genomic span

## Evidence
