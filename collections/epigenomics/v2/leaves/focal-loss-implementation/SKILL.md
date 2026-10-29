---
name: focal-loss-implementation
description: Use when training GILoop (or a similar dual-branch GCN + U-Net loop caller)
  on Hi-C cis-contact maps where loop pixels are vastly outnumbered by non-loop pixels,
  so plain cross-entropy over-weights easy negatives.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — the paper states GILoop
  'is still not capable enough to deal with' scHi-C
- Input is already a loop call set or feature table rather than raw contact-map patches
  needing per-pixel classification
- Sequencing depth differs from any pre-trained model in the repository and heavy
  downsampling pre-processing is infeasible — the paper notes weak usability in that
  scenario
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: GILoop
  role: The dual-branch neural network (GCN graph branch + U-Net image branch with
    bilinear pooling fusion) whose per-pixel loop/non-loop head is supervised by the
    focal cross-entropy loss
  repo: github.com/fzbio/GILoop
- name: SIP
  role: Alternative image-view loop caller used as a baseline comparison for GILoop's
    machine-learning-based loop detection
  repo: PouletAxel/SIP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:9917bbca151fdde79746158cbd20337b7b61c2a21cea1dd9d89a86024135f28d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - SIP
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# focal-loss-implementation

## Summary

Implements and trains GILoop with a focal cross-entropy loss that supervises per-pixel loop vs. non-loop classification on Hi-C contact maps, handling the extreme class imbalance between loop pixels and background pixels in CTCF-mediated loop calling.

## When to use

Use when training GILoop (or a similar dual-branch GCN + U-Net loop caller) on Hi-C cis-contact maps where loop pixels are vastly outnumbered by non-loop pixels, so plain cross-entropy over-weights easy negatives.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — the paper states GILoop 'is still not capable enough to deal with' scHi-C
- Input is already a loop call set or feature table rather than raw contact-map patches needing per-pixel classification
- Sequencing depth differs from any pre-trained model in the repository and heavy downsampling pre-processing is infeasible — the paper notes weak usability in that scenario

## Inputs

- KR-normalized O/E intra-chromosomal Hi-C contact matrix (from .cool file or Juicer dump text format)
- Per-pixel binary labels for loop vs. non-loop pixels on each contact-map patch
- GCN branch node features and edge features (contact-map pixel values) from the edge-encoder scheme
- U-Net per-pixel image features (16-dimensional vectors per pixel)
- Synthetic random Hi-C-like input tensors for smoke-testing forward-pass shapes

## Outputs

- Per-pixel loop/non-loop logits or predictions over each Hi-C contact-map patch
- Focal cross-entropy loss value per training step for monitoring convergence
- Trained dual-branch (GCN + U-Net + bilinear fusion) GILoop model checkpoint

## How to apply

Within the GILoop training pipeline, the model produces one logit per pixel of the Hi-C patch by fusing the 16-dimensional GCN and U-Net features via bilinear pooling; supervise these logits with a focal cross-entropy loss rather than plain cross-entropy so that easy, correctly classified background pixels contribute near-zero gradient and hard loop-pixel examples dominate. Trained branches are first pre-trained separately on their corresponding views of the data, then fused, so the loss is applied to the fused per-pixel classification head. Training data is a KR-normalized O/E Hi-C contact matrix (optionally downsampled to ~260 million cis-contacts, or trained in the 'lazy' non-downsampling mode on the input matrix directly). The loss sits alongside the GCN node-MLP/edge-MLP edge-encoder branch, the U-Net encoder-decoder image branch, and the bilinear fusion layer in the assembled model module (e.g. giloop_model.py). Validate by a smoke-test forward pass on random Hi-C-like tensors producing per-pixel logits of the expected shape, then confirm the loss decreases and loop precision/recall behave on held-out chromosomes.

## Related tools

- **GILoop** (The dual-branch neural network (GCN graph branch + U-Net image branch with bilinear pooling fusion) whose per-pixel loop/non-loop head is supervised by the focal cross-entropy loss) — github.com/fzbio/GILoop
- **SIP** (Alternative image-view loop caller used as a baseline comparison for GILoop's machine-learning-based loop detection) — PouletAxel/SIP

## Examples

```
python demo.py  # full GILoop pipeline: preprocessing (sequencing depth alignment, normalization, expected vector calculation), sampling, training with focal cross-entropy supervision
```

## Evaluation signals

- giloop_model.py executes without error and a smoke-test forward pass on random Hi-C-like inputs returns per-pixel predictions of the expected shape (one logit per pixel) through the fused 16-dimensional representation
- Focal loss on easy background pixels approaches zero, confirming down-weighting of well-classified negatives in the loop-imbalanced setting
- Training loss decreases monotonically on the KR-normalized O/E patches and converges without divergence under the 'lazy' non-downsampling mode
- GILoop loop calls outperform baseline machine-learning loop caller Peakachu on CTCF-associated loop recovery
- Trained model shows robust performance on downsampled low-quality Hi-C libraries (e.g. at the optimal density of ~260 million cis-contacts)

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- GILoop requires a relatively large amount of pre-processing time before training, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line has a sequencing depth different from any of the pre-trained models provided in the repository
- Focal-loss hyperparameters (gamma/alpha) are not specified in the README, so they must be taken from the reference implementation or paper

## Evidence

- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [abstract] the model outperforms the state-of-the-art loop calling framework and is also more robust against low-quality Hi-C libraries: "the model outperforms the state-of-the-art loop calling framework and is also more robust against low-quality Hi-C libraries"
- [discussion] users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training.: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
