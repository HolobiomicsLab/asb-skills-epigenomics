---
name: genomic-graph-representation-construction
description: Use when input is a tab-separated N×N intra-chromosomal Hi-C contact
  matrix (e.g. dumped from .hic data via juicer_tools) and the goal is to represent
  genomic bins as graph nodes for GAE-based feature learning or clustering.
when_to_use_negative:
- Input is already an embedding/feature table or a labelled TAD coordinate file —
  the graph construction step is only for raw NxN contact matrices.
- Data are not intra-chromosomal contact matrices (e.g. inter-chromosomal or 1D genomic
  tracks), since the adjacency-matrix interpretation requires a square bin-by-bin
  contact matrix.
- The goal is direct TAD calling without representation learning (e.g. using Armatus,
  Arrowhead, TopDom, TADbit, PSYCHIC, or HiCSeg), which use dynamic programming, insulation,
  or statistical segmentation instead.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3474
tools:
- name: TOAST
  role: Full pipeline that treats each genomic bin as a graph node with the Hi-C contact
    matrix as the adjacency matrix, and trains the graph auto-encoder on 400×400 submatrices
    to derive 16-dimensional embeddings
  repo: https://github.com/ghaiyan/TOAST
- name: PyTorch
  role: Implements the 2-layer GCN encoder/decoder auto-encoder trained with MSE loss
    and Adam (lr=0.001, 200 epochs)
- name: NumPy
  role: Loads and partitions the N×N contact matrix and stores the resulting node
    embeddings (gae_embeddings.npy)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:7e1d7a2a37391f3927e2301811cb0f7faca66640d1fbbc073919325de299113b
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
  - NumPy
  - PyTorch
  - TOAST
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# genomic-graph-representation-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a graph representation of a chromosome from an N×N Hi-C contact matrix by treating each genomic bin as a graph node and the contact matrix as the graph's adjacency matrix, enabling downstream graph auto-encoder (GAE) embedding of genomic bins.

## When to use

Use when input is a tab-separated N×N intra-chromosomal Hi-C contact matrix (e.g. dumped from .hic data via juicer_tools) and the goal is to represent genomic bins as graph nodes for GAE-based feature learning or clustering.

## When NOT to use

- Input is already an embedding/feature table or a labelled TAD coordinate file — the graph construction step is only for raw NxN contact matrices.
- Data are not intra-chromosomal contact matrices (e.g. inter-chromosomal or 1D genomic tracks), since the adjacency-matrix interpretation requires a square bin-by-bin contact matrix.
- The goal is direct TAD calling without representation learning (e.g. using Armatus, Arrowhead, TopDom, TADbit, PSYCHIC, or HiCSeg), which use dynamic programming, insulation, or statistical segmentation instead.

## Inputs

- Tab-separated N×N intra-chromosomal Hi-C contact matrix (N = number of equal-sized genomic regions of a chromosome), e.g. dumped from .hic format using juicer_tools
- Genomic bin size / Hi-C resolution (e.g. 50 Kb, 25 Kb, 5 Kb for GM12878 datasets)
- Submatrix size n=400 (for resolutions under 5 Kb) used to partition the contact matrix along the diagonal

## Outputs

- m non-overlapping 400×400 adjacency matrices (subgraphs of genomic bins)
- Node feature matrix X initialized as a diagonal matrix of ones per submatrix
- 16-dimensional node embeddings Z per submatrix, of shape m×400×16 (e.g. saved to gae_embeddings.npy)

## How to apply

Load the N×N intra-chromosomal Hi-C contact matrix (N equal-sized genomic regions of a chromosome) and treat it directly as the adjacency matrix A of a graph whose nodes are genomic bins, following TOAST's formulation. Partition the matrix along the diagonal into m non-overlapping n×n submatrices using a step of n=400 for resolutions under 5 Kb, treating each 400×400 submatrix as a 400-node graph's adjacency matrix. Initialize the node feature matrix X as a diagonal matrix of ones since no external node attributes exist. Feed A and X into a 2-layer GCN encoder (hidden dimension 64, embedding dimension 16, ReLU between layers) trained with a GCN decoder that reconstructs the adjacency matrix under MSE loss, Adam optimizer, lr=0.001, for 200 epochs. The rationale is that the learned 16-dimensional embeddings capture local 3D contact topology, which HDBSCAN later clusters to delineate TADs.

## Related tools

- **TOAST** (Full pipeline that treats each genomic bin as a graph node with the Hi-C contact matrix as the adjacency matrix, and trains the graph auto-encoder on 400×400 submatrices to derive 16-dimensional embeddings) — https://github.com/ghaiyan/TOAST
- **PyTorch** (Implements the 2-layer GCN encoder/decoder auto-encoder trained with MSE loss and Adam (lr=0.001, 200 epochs))
- **NumPy** (Loads and partitions the N×N contact matrix and stores the resulting node embeddings (gae_embeddings.npy))

## Examples

```
python toast.py  # run from /src of ghaiyan/TOAST with input/output folders changed; input is the N×N tab-separated contact matrix, output is the TAD file at the merge file
```

## Evaluation signals

- Output embedding file gae_embeddings.npy exists and has shape m×400×16 (16-dimensional embedding per genomic bin node, as specified).
- MSE reconstruction loss decreases over the 200 training epochs, indicating the GCN decoder successfully reconstructs the 400×400 adjacency matrix.
- Number of submatrices m matches ceil(N/n) non-overlapping partitions of the diagonal with step n=400.
- Downstream HDBSCAN clustering on the embeddings yields contiguous same-label regions exceeding 200 Kb that can be identified as TADs.
- TAD calls reproduce paper-level quality: FMS around 0.98 on Wang et al. simulated data across noise levels, and boundary enrichment for CTCF (average anchoring ratio 0.66) on GM12878 Hi-C at 50 Kb, 25 Kb, and 5 Kb resolutions.

## Limitations

- The n=400 submatrix partitioning is specified for Hi-C resolutions under 5 Kb; other resolutions may need a different submatrix step.
- Node features are initialized as a diagonal matrix of ones because no external node attributes are used, so embedding quality depends entirely on adjacency structure of the contact matrix.
- TOAST was validated on GM12878 Hi-C (50 Kb, 25 Kb, 5 Kb) and simulated matrices at limited noise levels; performance on noisier or very-low-coverage data is not established here.
- The TOAST README requires a specific environment (PyTorch 1.7.1, Python 3.6.12, NumPy 1.19.5, Cooltools 0.4.1, Fanc 0.9.24) that may not match newer stacks.

## Evidence
