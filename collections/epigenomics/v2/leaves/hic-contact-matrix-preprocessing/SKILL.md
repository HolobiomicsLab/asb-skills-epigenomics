---
name: hic-contact-matrix-preprocessing
description: Use when raw .hic files (e.g., GM12878 accession 4DNFI1UEG1HD from 4DN/GSE63525)
  must be dumped into NxN KR-normalized matrices and chunked into 400x400 submatrices
  before GAE-based TAD calling.
when_to_use_negative:
- Input is already an NxN tab-separated intra-chromosomal contact matrix — only the
  400×400 diagonal partitioning step is needed, not .hic dumping
- Data are not Hi-C contact matrices (e.g., the EPIANN enhancer-promoter CSV dataset
  referenced in the README) — this preprocessing produces graph adjacency matrices,
  not epigenomic interaction tables
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: Consumes the partitioned 400×400 submatrices as graph adjacency matrices for
    downstream GAE training and TAD calling
  repo: https://github.com/ghaiyan/TOAST
- name: Juicer
  role: Dumps KR-normalized NxN intra-chromosomal contact matrices from .hic files
- name: Cooltools
  role: Hi-C matrix handling library listed among TOAST's required packages
  repo: https://github.com/ghaiyan/TOAST
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_008
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:72363641d19ad7d29f113bad990784f7aa55bb3f316f96e1c9b856d61fe385b7
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
  - Cooltools
  - Juicer
  - NumPy
  - TOAST
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hic-contact-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess Hi-C data into tab-separated N×N intra-chromosomal contact matrices and partition them along the diagonal into non-overlapping 400×400 submatrices, the input format required by TOAST's graph auto-encoder TAD identification pipeline.

## When to use

Use when raw .hic files (e.g., GM12878 accession 4DNFI1UEG1HD from 4DN/GSE63525) must be dumped into NxN KR-normalized matrices and chunked into 400x400 submatrices before GAE-based TAD calling.

## When NOT to use

- Input is already an NxN tab-separated intra-chromosomal contact matrix — only the 400×400 diagonal partitioning step is needed, not .hic dumping
- Data are not Hi-C contact matrices (e.g., the EPIANN enhancer-promoter CSV dataset referenced in the README) — this preprocessing produces graph adjacency matrices, not epigenomic interaction tables

## Inputs

- .hic file (e.g., 4DNFI1UEG1HD from the 4D Nucleome Data Portal or GEO GSE63525)
- Simulated Hi-C contact matrices (e.g., Trussart et al. toy genomes or Forcato et al. datasets)
- Juicer tools for dumping .hic data

## Outputs

- Tab-separated N×N intra-chromosomal contact matrix per chromosome (NxN format, KR-normalized)
- m non-overlapping 400×400 submatrices per chromosome (adjacency matrices A, one per 400-bin graph)

## How to apply

Download the .hic file and use Juicer tools to dump the KR-normalized intra-chromosomal contact matrix as a tab-separated N×N matrix, where N is the number of equal-sized genomic regions of a chromosome (e.g., 50 Kb, 25 Kb, or 5 Kb resolution for GM12878 chromosomes 1–22). Treat each genomic bin as a graph node and the contact matrix as the graph's adjacency matrix. Partition the matrix along the diagonal into m non-overlapping n×n submatrices using a step of n=400, so each 400×400 submatrix serves as the adjacency matrix A of a 400-node graph; initialize the node feature matrix X as a diagonal matrix of ones. These submatrices are then the training inputs for the 2-layer GAE (hidden_dim=64, embedding dimension 16) in downstream TOAST steps.

## Related tools

- **TOAST** (Consumes the partitioned 400×400 submatrices as graph adjacency matrices for downstream GAE training and TAD calling) — https://github.com/ghaiyan/TOAST
- **Juicer** (Dumps KR-normalized NxN intra-chromosomal contact matrices from .hic files)
- **Cooltools** (Hi-C matrix handling library listed among TOAST's required packages) — https://github.com/ghaiyan/TOAST

## Examples

```
java -jar juicer_tools.jar dump observed KR 4DNFI1UEG1HD chr1 chr1 BP 50000 gm12878_chr1_50kb.txt
```

## Evaluation signals

- Dumped matrix is square N×N, tab-separated, and KR-normalized, with N equal to the number of equal-sized chromosome regions at the chosen resolution
- Partitioning yields m = floor(N/400) non-overlapping 400×400 submatrices along the diagonal with step n=400
- Each submatrix is symmetric and usable as an adjacency matrix for a 400-node graph with diagonal-ones feature matrix X
- Downstream GAE training on the submatrices shows decreasing MSE reconstruction loss over the 200 epochs, confirming the inputs are valid
- Identified TADs fall within the expected 200 Kb–5 Mb length interval and show a plausible TAD_quality (intra-TAD minus inter-TAD average contact frequency)

## Limitations

- The 400×400 partitioning step is specified for resolutions under 5 Kb; other resolutions may require different n
- Sequential preprocessing/processing of GM12878 matrices is slow (2, 4, and 11 hours at 50 Kb, 25 Kb, 5 Kb respectively), though parallelization across chromosomes reduces this to roughly 0.5 hours
- TOAST requires Python 3.6.12-era dependencies (PyTorch 1.7.1, NumPy 1.19.5, Cooltools 0.4.1, Fanc 0.9.24) per the README, which may conflict with newer environments

## Evidence
