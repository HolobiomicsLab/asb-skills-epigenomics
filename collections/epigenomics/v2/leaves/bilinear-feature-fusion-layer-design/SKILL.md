---
name: bilinear-feature-fusion-layer-design
description: 'Use when reconstructing or adapting GILoop''s dual-branch architecture:
  two single-view encoders (GCN and U-Net) must be combined per pixel into one fused
  representation that feeds loop vs. non-loop pixel classification.'
when_to_use_negative:
- 'Data is extremely sparse single-cell Hi-C (scHi-C): the article states GILoop ''is
  still not capable enough to deal with'' scHi-C and only focuses on bulk Hi-C.'
- 'Only one view is needed: prior image-only tools (SIP, Chromosight) or graph-only
  approaches apply when graph-image fusion is not the goal.'
- The targeted cell line's sequencing depth differs from any pre-trained model in
  the repository and no retraining/downsampling pipeline is available — the article
  flags this as 'weak usability'.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3474
tools:
- name: GILoop
  role: Dual-branch neural network whose GCN and U-Net branches are fused via bilinear
    pooling; reference implementation for layer sizes and interfaces
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Contrast image-view-only loop caller that regards Hi-C matrices as images;
    context for why a dual-branch fused model is used instead
  repo: PouletAxel/SIP
- name: Peakachu
  role: Machine-learning-based loop calling baseline that GILoop (with fused features)
    significantly outperforms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:3db2744f75ba6f30fca0c8165135b370421dc368b7fdb3eaadf7020d5c409622
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  - SIP
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bilinear-feature-fusion-layer-design

## Summary

Design and validate the bilinear pooling fusion layer that combines GILoop's separately pre-trained GCN (graph-view) and U-Net (image-view) branch outputs into a joint 16-dimensional per-pixel representation for CTCF-mediated chromatin loop calling on Hi-C contact maps.

## When to use

Use when reconstructing or adapting GILoop's dual-branch architecture: two single-view encoders (GCN and U-Net) must be combined per pixel into one fused representation that feeds loop vs. non-loop pixel classification.

## When NOT to use

- Data is extremely sparse single-cell Hi-C (scHi-C): the article states GILoop 'is still not capable enough to deal with' scHi-C and only focuses on bulk Hi-C.
- Only one view is needed: prior image-only tools (SIP, Chromosight) or graph-only approaches apply when graph-image fusion is not the goal.
- The targeted cell line's sequencing depth differs from any pre-trained model in the repository and no retraining/downsampling pipeline is available — the article flags this as 'weak usability'.

## Inputs

- Hi-C cis-contact matrix patches represented as graphs (node features + edge features from contact-map pixel values)
- Hi-C cis-contact matrix patches represented as images
- Loop vs. non-loop pixel labels for focal cross-entropy supervision

## Outputs

- Fused 16-dimensional per-pixel feature representation from bilinear pooling of GCN and U-Net branch outputs
- Per-pixel loop/non-loop classification logits (one logit per pixel)

## How to apply

First pre-train the two branches separately, each on its corresponding view of the Hi-C cis-contact matrix, as the article describes for GILoop. Implement the GCN branch with node MLP operations transforming node features and edge MLP operations encoding contact-map pixel values as edge features, and the U-Net branch with encoder-decoder blocks extracting per-pixel image features. Add a bilinear pooling layer that combines the two 16-dimensional feature vectors produced per pixel by the GCN and U-Net branches into a joint representation. Supervise the fused output with a focal cross-entropy loss for loop vs. non-loop pixel classification, and verify the whole model (e.g. in giloop_model.py, following github.com/fzbio/GILoop) with a smoke-test forward pass on random Hi-C-like inputs that returns one logit per pixel.

## Related tools

- **GILoop** (Dual-branch neural network whose GCN and U-Net branches are fused via bilinear pooling; reference implementation for layer sizes and interfaces) — github.com/fzbio/GILoop
- **SIP** (Contrast image-view-only loop caller that regards Hi-C matrices as images; context for why a dual-branch fused model is used instead) — PouletAxel/SIP
- **Peakachu** (Machine-learning-based loop calling baseline that GILoop (with fused features) significantly outperforms)

## Examples

```
python -c "from giloop_model import GILoopModel; import torch; m=GILoopModel(); x=torch.rand(1,1,32,32); out=m(x); assert out.shape[-2:]==(32,32)"
```

## Evaluation signals

- Module giloop_model.py exists and executes without error
- Smoke-test forward pass on random Hi-C-like input tensors returns per-pixel predictions of the expected shape (one logit per pixel)
- Intermediate fused representation is 16-dimensional per pixel, combining the 16-dim GCN-branch and 16-dim U-Net-branch vectors
- Fused model outperforms each single-view branch (graph-only or image-only) on CTCF-mediated loop calling, corroborating the complementary relationship between the two views
- Fused GILoop model significantly outperforms Peakachu, the most recent machine learning-based chromatin loop detector

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data.
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing).
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository.
- Graph-based and image-based models show distinct preferences for matrix density, so branch behavior depends on downsampling to the optimal density of 260 million cis-contacts (or using the 'lazy' non-downsampling mode).

## Evidence

- [full_text] GILoop's two branches are first pre-trained separately, each on its corresponding view of the data, and the separately trained branches are then fused via a bilinear pooling operation.: [evidence span withheld — anchor status: not_found]
- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training.: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
