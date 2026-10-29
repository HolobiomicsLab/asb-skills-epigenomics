---
name: neural-network-training-with-mse-reconstruction-loss
description: Use when a Hi-C N×N contact matrix must be converted into low-dimensional
  node embeddings (dim 16) via a graph auto-encoder before unsupervised clustering
  of genomic bins into TADs.
when_to_use_negative:
- The goal is direct TAD calling from a full Hi-C matrix without embedding generation
  — dynamic-programming or boundary-detection methods (Armatus, Arrowhead, TopDom)
  are the alternatives TOAST compares against
- The embedding dimension or architecture differs from the validated configuration
  (2 layers, hidden 64, output 16) without re-tuning, since TOAST validated robustness
  only for hidden_dim=64/output_dim=16 or a single-layer output_dim=128
- Input is a raw .hic file — it must first be converted to an NxN matrix with juicer_tools
  per the README
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0102
tools:
- name: TOAST
  role: Pipeline whose GAE module implements the auto-encoder training on 400×400
    Hi-C submatrices; toast.py in /src runs TAD identification
  repo: https://github.com/ghaiyan/TOAST
- name: PyTorch
  role: Framework for constructing and training the 2-layer GCN encoder/decoder with
    MSE loss and Adam (lr=0.001)
- name: NumPy
  role: Stores the resulting m×400×16 embedding array as gae_embeddings.npy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:e6042c16aa2dabcd71be9353b5f74b27799657230c1833727b2edc4de447e75c
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Neural-network-training-with-MSE-reconstruction-loss

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a 2-layer graph auto-encoder (GCN encoder/decoder) on non-overlapping 400×400 Hi-C contact submatrices using MSE reconstruction loss (Adam, lr=0.001, 200 epochs) to produce 16-dimensional embeddings per genomic bin for downstream HDBSCAN-based TAD calling in TOAST.

## When to use

Use when a Hi-C N×N contact matrix must be converted into low-dimensional node embeddings (dim 16) via a graph auto-encoder before unsupervised clustering of genomic bins into TADs.

## When NOT to use

- The goal is direct TAD calling from a full Hi-C matrix without embedding generation — dynamic-programming or boundary-detection methods (Armatus, Arrowhead, TopDom) are the alternatives TOAST compares against
- The embedding dimension or architecture differs from the validated configuration (2 layers, hidden 64, output 16) without re-tuning, since TOAST validated robustness only for hidden_dim=64/output_dim=16 or a single-layer output_dim=128
- Input is a raw .hic file — it must first be converted to an NxN matrix with juicer_tools per the README

## Inputs

- N×N tab-separated intra-chromosomal Hi-C contact matrix (NxN format, e.g. dumped from .hic via juicer_tools)
- Partitioned non-overlapping 400×400 submatrices serving as graph adjacency matrices
- Diagonal ones matrix as initial node feature matrix X

## Outputs

- m×400×16 NumPy array of 16-dimensional node embeddings (gae_embeddings.npy)
- Decreasing MSE reconstruction-loss curve over 200 epochs
- Embedding features passed to HDBSCAN clustering module

## How to apply

Load the N×N intra-chromosomal contact matrix (tab-separated N×N format per the TOAST README) and partition it along the diagonal into m non-overlapping 400×400 submatrices (step n=400, as specified for resolutions under 5 Kb). Treat each submatrix as the adjacency matrix A of a 400-node graph and initialize node features X as a diagonal matrix of ones. Build a 2-layer GCN encoder in PyTorch with hidden dimension 64 and embedding dimension 16 (ReLU between layers), paired with a GCN decoder reconstructing the 400×400 adjacency; train with MSE reconstruction loss, Adam optimizer, lr=0.001, for 200 epochs. The rationale is that TOAST found optimal TAD identification when the network has 2 layers, embedding dimension 16, and hidden dimension 64. Save all m×400×16 embeddings (e.g. gae_embeddings.npy) and confirm the MSE loss decreases across the 200 epochs.

## Related tools

- **TOAST** (Pipeline whose GAE module implements the auto-encoder training on 400×400 Hi-C submatrices; toast.py in /src runs TAD identification) — https://github.com/ghaiyan/TOAST
- **PyTorch** (Framework for constructing and training the 2-layer GCN encoder/decoder with MSE loss and Adam (lr=0.001))
- **NumPy** (Stores the resulting m×400×16 embedding array as gae_embeddings.npy)

## Examples

```
conda activate pytorch1.7.1; cd /src; python toast.py  # edit input/output folders; pipeline trains the GAE (hidden 64, dim 16, MSE, Adam lr=0.001, 200 epochs) and calls TADs
```

## Evaluation signals

- Saved embedding file gae_embeddings.npy exists and has shape m×400×16 (16 features per node)
- MSE reconstruction loss decreases monotonically/en trend over the 200 training epochs
- Submatrix partitioning is non-overlapping: m = floor(N/400) submatrices of 400×400 each
- Trained configuration matches the validated optimum: 2 GCN layers, hidden_dim=64, output_dim=16
- Downstream HDBSCAN on the embeddings yields contiguous same-label regions exceeding 200 Kb classified as TADs

## Limitations

- The 400×400 partitioning with step n=400 is specified for resolutions under 5 Kb; other resolutions may require a different submatrix size
- TOAST was validated on GM12878 Hi-C at 50 Kb, 25 Kb, and 5 Kb and simulated matrices at five noise levels; performance on other cell lines or resolutions is untested here
- Robustness claim holds for hidden_dim=64/output_dim=16 (or one-layer output_dim=128); other hyperparameter settings may degrade TAD identification

## Evidence
