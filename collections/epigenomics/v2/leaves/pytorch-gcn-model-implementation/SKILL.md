---
name: pytorch-gcn-model-implementation
description: Use when you need to convert an N×N Hi-C contact matrix into low-dimensional
  node embeddings (e.g., 16-D per genomic bin) as features for downstream unsupervised
  clustering of TADs.
when_to_use_negative:
- The input is already a feature/embedding table — this skill consumes raw N×N contact
  matrices as graph adjacency matrices
- The task is TAD calling by dynamic programming or statistical segmentation (e.g.,
  Armatus, TopDom, HiCSeg, TADtree) rather than GAE-based embedding plus clustering
- Hi-C resolution or matrix structure makes non-overlapping 400×400 diagonal partitioning
  inappropriate (e.g., very low-resolution matrices far above 5 Kb, where the article's
  n=400 specification was made)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: Pipeline whose GAE module performs the 2-layer GCN auto-encoder training on
    400×400 Hi-C submatrices; run via toast.py from the repository /src folder
  repo: https://github.com/ghaiyan/TOAST
- name: PyTorch
  role: Framework used to implement the 2-layer GCN encoder/decoder and train with
    Adam (lr=0.001) and MSE reconstruction loss; README specifies PyTorch 1.7.1
  repo: https://github.com/ghaiyan/TOAST
- name: NumPy
  role: Array handling for the N×N contact matrix input and for saving the m×400×16
    embedding array to gae_embeddings.npy; README specifies NumPy 1.19.5
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Downstream consumer of the 16-D GAE embeddings — assigns cluster labels to
    each genomic bin after embedding is complete
  repo: https://github.com/ghaiyan/TOAST
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:745d35d9e5494859848f2b84db9f57dd88dbcf3c87e21b39c7b79758fa9c4400
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
  - NumPy
  - PyTorch
  - TOAST
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pytorch-gcn-model-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement and train a 2-layer graph auto-encoder (GAE) in PyTorch whose GCN encoder embeds genomic bins of 400x400 Hi-C submatrices into 16-dimensional node features, with a GCN decoder reconstructing the adjacency matrix via MSE loss. This is the feature-generation core of TOAST's TAD identification pipeline.

## When to use

Use when you need to convert an N×N Hi-C contact matrix into low-dimensional node embeddings (e.g., 16-D per genomic bin) as features for downstream unsupervised clustering of TADs.

## When NOT to use

- The input is already a feature/embedding table — this skill consumes raw N×N contact matrices as graph adjacency matrices
- The task is TAD calling by dynamic programming or statistical segmentation (e.g., Armatus, TopDom, HiCSeg, TADtree) rather than GAE-based embedding plus clustering
- Hi-C resolution or matrix structure makes non-overlapping 400×400 diagonal partitioning inappropriate (e.g., very low-resolution matrices far above 5 Kb, where the article's n=400 specification was made)

## Inputs

- N×N intra-chromosomal Hi-C contact matrix in tab-separated NxN format (e.g., dumped from .hic files with juicer_tools.js)
- Submatrix size parameter n (400 as specified for resolutions under 5 Kb)

## Outputs

- m×400×16 array of 16-dimensional node embeddings (one 16-D vector per genomic bin per 400×400 submatrix), e.g., saved to gae_embeddings.npy
- Per-epoch MSE reconstruction loss trace over 200 training epochs

## How to apply

Partition the N×N Hi-C contact matrix along the diagonal into m non-overlapping 400×400 submatrices (step n=400, as specified for resolutions under 5 Kb), treating each as the adjacency matrix A of a 400-node graph with node feature matrix X initialized as a diagonal matrix of ones. Build a 2-layer GCN encoder in PyTorch with hidden layer dimension 64 and embedding dimension f=16, using ReLU activation between layers. Train the full auto-encoder (GCN encoder plus GCN decoder reconstructing the 400×400 adjacency) with Mean Squared Error reconstruction loss, the Adam optimizer at lr=0.001, for 200 epochs. The article's ablations found the optimal TAD identification with 2 layers, hidden_dim=64, output_dim=16 (or a single-layer net with output_dim=128), and 64/16 was selected as most robust. Extract the final 16-dimensional embeddings Z for every submatrix and save them (e.g., gae_embeddings.npy) for input to the HDBSCAN clustering module.

## Related tools

- **TOAST** (Pipeline whose GAE module performs the 2-layer GCN auto-encoder training on 400×400 Hi-C submatrices; run via toast.py from the repository /src folder) — https://github.com/ghaiyan/TOAST
- **PyTorch** (Framework used to implement the 2-layer GCN encoder/decoder and train with Adam (lr=0.001) and MSE reconstruction loss; README specifies PyTorch 1.7.1) — https://github.com/ghaiyan/TOAST
- **NumPy** (Array handling for the N×N contact matrix input and for saving the m×400×16 embedding array to gae_embeddings.npy; README specifies NumPy 1.19.5) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Downstream consumer of the 16-D GAE embeddings — assigns cluster labels to each genomic bin after embedding is complete) — https://github.com/ghaiyan/TOAST

## Examples

```
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu; conda activate pytorch1.7.1; python toast.py  # run from /src with input and output folders changed to point at your N×N contact matrix and TAD output
```

## Evaluation signals

- Output embedding file gae_embeddings.npy exists and has shape m×400×16 (16 dimensions per genomic bin node, as specified)
- MSE reconstruction loss decreases monotonically (or near-monotonically) over the 200 training epochs
- Encoder architecture matches the tuned optimum: 2 GCN layers, hidden dimension 64, embedding dimension 16
- Submatrix count m matches ceil(N/400) with non-overlapping 400×400 blocks along the matrix diagonal
- Downstream HDBSCAN labels produce contiguous same-label regions exceeding 200 Kb that score as valid TADs in evaluate_TAD.py

## Limitations

- The n=400 submatrix partitioning was specified for Hi-C resolutions under 5 Kb; other resolutions may need retuning of block size and network dimensions
- Embedding quality depends on reconstruction (MSE) fit to noisy contact matrices; simulated-data validation showed FMS decreasing with noise level (though it remained around 0.98 on Wang et al. data)
- GAE training is reconstructive and unsupervised — it does not directly optimize TAD boundary accuracy; boundary quality is only assessed after HDBSCAN clustering and TAD generation

## Evidence
