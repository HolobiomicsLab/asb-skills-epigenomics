---
name: graph-autoencoder-embedding-training
description: Use when an NxN intra-chromosomal Hi-C contact matrix needs conversion
  into low-dimensional node features for unsupervised TAD identification, e.g. inputs
  dumped from .hic files at 50 Kb, 25 Kb, or 5 Kb resolution.
when_to_use_negative:
- Input is not a contact matrix but already a feature/embedding table or genomic annotation
  — clustering or downstream TAD-calling modules should be used directly
- Hi-C resolution exceeds 5 Kb such that the 400×400 submatrix partitioning (n=400)
  no longer applies as specified for resolutions under 5 Kb scenarios in the paper
- The goal is supervised TAD boundary prediction rather than unsupervised embedding
  generation — TOAST is a fully unsupervised pipeline
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TOAST
  role: Provides the GAE training pipeline (toast.py) that partitions contact matrices
    and trains the 2-layer graph auto-encoder
  repo: https://github.com/ghaiyan/TOAST
- name: PyTorch
  role: Framework for building the GCN encoder/decoder, Adam optimizer, and MSE loss
    (README pins PyTorch 1.7.1)
- name: NumPy
  role: Loads the NxN contact matrices and stores the m×400×16 embedding array (gae_embeddings.npy);
    README pins NumPy 1.19.5
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_001
  - task_004
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:da6fcfc0de0b6d372b272d370eb3eb896b0312e2642f0a980f506a6fb9f9a8cd
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
  - HiCSeg
  - Juicer
  - NumPy
  - PyTorch
  - TOAST
  - TopDom
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# graph-autoencoder-embedding-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a 2-layer graph auto-encoder (GAE) on non-overlapping 400×400 Hi-C submatrices in PyTorch to produce 16-dimensional node embeddings per genomic bin, following the TOAST optimal architecture (hidden_dim=64, output_dim=16, Adam lr=0.001, 200 epochs, MSE reconstruction loss).

## When to use

Use when an NxN intra-chromosomal Hi-C contact matrix needs conversion into low-dimensional node features for unsupervised TAD identification, e.g. inputs dumped from .hic files at 50 Kb, 25 Kb, or 5 Kb resolution.

## When NOT to use

- Input is not a contact matrix but already a feature/embedding table or genomic annotation — clustering or downstream TAD-calling modules should be used directly
- Hi-C resolution exceeds 5 Kb such that the 400×400 submatrix partitioning (n=400) no longer applies as specified for resolutions under 5 Kb scenarios in the paper
- The goal is supervised TAD boundary prediction rather than unsupervised embedding generation — TOAST is a fully unsupervised pipeline

## Inputs

- NxN tab-separated intra-chromosomal Hi-C contact matrix (KR-normalized, dumped from .hic via juicer_tools)
- Non-overlapping 400×400 submatrices partitioned along the matrix diagonal (n=400)
- Node feature matrix X initialized as a diagonal matrix of ones

## Outputs

- 16-dimensional node embeddings Z for every genomic bin in each submatrix (gae_embeddings.npy with shape m×400×16)
- Trained GCN encoder/decoder weights per submatrix
- Training loss curve showing MSE reconstruction loss decreasing over 200 epochs

## How to apply

Divide the NxN contact matrix along the diagonal into m non-overlapping 400×400 submatrices (n=400 for 50 Kb resolution), treating each bin as a graph node, the submatrix as adjacency matrix A, and initializing node features X as a diagonal matrix of ones. Build a 2-layer GCN encoder (hidden layer 64, ReLU activation, embedding dimension f=16) plus a GCN decoder in PyTorch that reconstructs the 400×400 adjacency matrix. Train with the Adam optimizer, learning rate lr=0.001, for 200 epochs under Mean Squared Error (MSE) reconstruction loss — this 2-layer hidden_dim=64/output_dim=16 configuration was found optimal and most robust across six GAE variants tested. Extract and save the final 16-dimensional embeddings Z for every submatrix. Rationale: the graph representation lets the auto-encoder learn structural contact features that HDBSCAN subsequently clusters into TADs.

## Related tools

- **TOAST** (Provides the GAE training pipeline (toast.py) that partitions contact matrices and trains the 2-layer graph auto-encoder) — https://github.com/ghaiyan/TOAST
- **PyTorch** (Framework for building the GCN encoder/decoder, Adam optimizer, and MSE loss (README pins PyTorch 1.7.1))
- **NumPy** (Loads the NxN contact matrices and stores the m×400×16 embedding array (gae_embeddings.npy); README pins NumPy 1.19.5)

## Examples

```
python toast.py  # edit input/output folder paths in the script per README
```

## Evaluation signals

- gae_embeddings.npy exists and has shape m×400×16, confirming embedding dimension 16 per node as specified
- MSE reconstruction loss decreases monotonically/consistently over the 200 training epochs
- Architecture sweep confirms hidden_dim=64 / output_dim=16 (2 layers) achieves the highest FMS/similarity scores, matching the paper's reported optimum
- Downstream FMS against ground-truth TADs on Wang et al. simulated data (noise levels 4-20) stays around 0.98
- TAD_quality score on real GM12878 50 Kb data reaches ~1654.03 with an average TAD number of ~4736

## Limitations

- The 400×400 submatrix size (n=400) is calibrated for resolutions under 5 Kb; other resolutions require re-checking the partitioning step
- Downstream TADs are only retained from contiguous same-label (HDBSCAN) regions longer than 200 Kb, so short embeddings-derived segments are discarded
- Sequential GAE training across all chromosomes is slow (2/4/11 hours for GM12878 50/25/5 Kb); parallelism across chromosomes is needed to finish in ~0.5 hours

## Evidence

- [other] Training hyperparameters: [evidence span withheld — anchor status: not_found]
