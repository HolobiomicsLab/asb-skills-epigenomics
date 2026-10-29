---
name: graph-neural-network-architecture-implementation
description: Use when reimplementing or retraining the GILoop dual-branch GCN + U-Net
  loop-calling architecture on Hi-C cis-contact maps, or porting the fused model to
  new cell lines.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) — the paper states the tool
  is not capable of dealing with scHi-C and only focuses on bulk Hi-C.
- The task is running loop calling with an existing pre-trained model on .cool/Juicer-dump
  input, not reconstructing or retraining the network architecture.
- The targeted cell line's sequencing depth differs from all pre-trained models in
  the repository and the downsampling pre-processing cannot be performed.
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: GILoop
  role: Reference dual-branch GCN + U-Net implementation providing layer sizes, branch
    interfaces, and pre-trained models for the reconstruction
  repo: github.com/fzbio/GILoop
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:4b33fa5aad7ae31bdf1156a93d82577d9d58f2a778472e9a3963ce63963d8dc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# graph-neural-network-architecture-implementation

## Summary

Reconstruct GILoop's dual-branch neural network — a GCN branch over the graph view and a U-Net branch over the image view of Hi-C cis-contact maps — with bilinear pooling fusing the two 16-dimensional per-pixel feature vectors, supervised by focal cross-entropy for CTCF-mediated loop calling.

## When to use

Use when reimplementing or retraining the GILoop dual-branch GCN + U-Net loop-calling architecture on Hi-C cis-contact maps, or porting the fused model to new cell lines.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) — the paper states the tool is not capable of dealing with scHi-C and only focuses on bulk Hi-C.
- The task is running loop calling with an existing pre-trained model on .cool/Juicer-dump input, not reconstructing or retraining the network architecture.
- The targeted cell line's sequencing depth differs from all pre-trained models in the repository and the downsampling pre-processing cannot be performed.

## Inputs

- Hi-C cis-contact map in two views: a graph (nodes with features; edges encoding contact-map pixel values) and a 2D image (contact-map patch)
- KR-normalized O/E matrices from .cool files or Juicer-dump text files (per README)
- Synthetic random Hi-C-like input tensors for smoke-testing the assembled model

## Outputs

- giloop_model.py module defining the fused dual-branch model
- Per-pixel loop vs. non-loop logits (one logit per pixel) from a forward pass
- Fused 16-dimensional per-pixel representation produced by bilinear pooling

## How to apply

Reconstruct the model as a single Python module (e.g., giloop_model.py), fixing layer sizes and interfaces against the reference implementation at github.com/fzbio/GILoop. Implement the GCN branch with node MLP operations transforming node features and edge MLP operations encoding contact-map pixel values as edge features, and the U-Net branch with encoder-decoder blocks extracting per-pixel image features; per the paper, the branches are first pre-trained separately, each on its corresponding view, before fusion. Add the bilinear pooling layer that combines the two 16-dimensional per-pixel vectors into a joint representation, plus the focal cross-entropy loss supervising loop vs. non-loop pixel classification. Assemble into one model class and validate with a smoke-test forward pass on synthetic random tensors, confirming one logit per pixel through the fused 16-dim representation. For training data, use the downsampling mode targeting the experimentally optimal 260-million-cis-contact density, or the non-downsampling ('lazy') mode that trains the GCN branch directly on the input matrix.

## Related tools

- **GILoop** (Reference dual-branch GCN + U-Net implementation providing layer sizes, branch interfaces, and pre-trained models for the reconstruction) — github.com/fzbio/GILoop

## Examples

```
conda activate GIL; python demo.py
```

## Evaluation signals

- giloop_model.py exists and executes without error
- Smoke-test forward pass on random Hi-C-like tensors returns one logit per pixel of the expected shape
- GCN and U-Net branches each emit a 16-dimensional feature vector per pixel before bilinear pooling fusion
- Focal cross-entropy loss computes on loop vs. non-loop pixel labels without shape errors
- Layer sizes and branch interfaces match the reference implementation at github.com/fzbio/GILoop

## Limitations

- GILoop only handles bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data.
- Pre-processing (sequencing depth alignment, normalization, expected vector calculation, downsampling) is heavy and can lead to large running time.
- Usability weakens if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository.

## Evidence

- [abstract] dual-branch neural network for loop calling: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] lazy (non-downsampling) GCN training mode: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [discussion] optimal downsampling density: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [discussion] bulk Hi-C scope limitation: "The tool we developed in this study, however, only focuses on bulk Hi-C."
- [readme] model purpose from repository README: "GILoop is a deep learning model for detecting CTCF-mediated loops on Hi-C contact maps."
