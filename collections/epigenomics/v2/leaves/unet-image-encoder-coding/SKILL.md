---
name: unet-image-encoder-coding
description: Use when reconstructing or modifying the GILoop dual-branch architecture
  and you need to code the U-Net encoder-decoder that produces per-pixel image features
  from Hi-C contact-map patches for bilinear fusion.
when_to_use_negative:
- The input is extremely sparse single-cell Hi-C data, which GILoop is not capable
  of dealing with
- Only the graph view is needed and no image branch or fusion is being built
- The tool used treats matrices purely as images with classical filters (e.g., SIP/Chromosight),
  not a learned dual-branch deep model
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: GILoop
  role: Dual-branch network (U-Net image branch + GCN branch + bilinear fusion) being
    implemented; reference code from its repository fixes layer sizes and interfaces
  repo: github.com/fzbio/GILoop
- name: SIP
  role: 'Contrast case: treats Hi-C matrices as images with classical image processing
    rather than a U-Net encoder'
  repo: PouletAxel/SIP
- name: Chromosight
  role: 'Contrast case: treats matrices as images without learned per-pixel encoders'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:722a0d692710455687dd42db97dbdad9f1bd8369fc270ccbe6bb6b1ef2fa2a0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chromosight
  - GILoop
  - SIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# unet-image-encoder-coding

## Summary

Implements the U-Net image-encoder branch of GILoop, the dual-branch CTCF loop-calling network, which extracts per-pixel image features from Hi-C cis-contact maps that are later fused with GCN branch features via bilinear pooling.

## When to use

Use when reconstructing or modifying the GILoop dual-branch architecture and you need to code the U-Net encoder-decoder that produces per-pixel image features from Hi-C contact-map patches for bilinear fusion.

## When NOT to use

- The input is extremely sparse single-cell Hi-C data, which GILoop is not capable of dealing with
- Only the graph view is needed and no image branch or fusion is being built
- The tool used treats matrices purely as images with classical filters (e.g., SIP/Chromosight), not a learned dual-branch deep model

## Inputs

- KR-normalized O/E Hi-C contact-map patches (Juicer dump text format)
- .cool files (Cooler format, now supported)
- Synthetic random Hi-C-like tensors for smoke-testing the forward pass

## Outputs

- Per-pixel 16-dimensional image-feature vectors from the U-Net encoder-decoder
- Per-pixel logits for loop vs. non-loop classification after bilinear fusion with the GCN branch

## How to apply

Model the normalized Hi-C cis-contact map (e.g., KR-normalized O/E matrices from Juicer dump or .cool inputs) as an image view, and implement a U-Net with encoder-decoder blocks that outputs a 16-dimensional feature vector per pixel. Keep interfaces consistent with the GCN branch implemented from the reference repository (github.com/fzbio/GILoop) so the bilinear pooling fusion layer can combine the two 16-dim vectors into a joint representation. Train the branches separately first, each on its corresponding view, before fusing; supervise loop vs. non-loop pixel classification with focal cross-entropy loss. Optionally enable the non-downsampling 'lazy' mode, which directly trains the branch on the input matrix, or downsample to the optimal density of 260 million cis-contacts. Assemble the branch in giloop_model.py and validate with a forward pass on synthetic random Hi-C-like inputs to confirm per-pixel predictions of the expected shape.

## Related tools

- **GILoop** (Dual-branch network (U-Net image branch + GCN branch + bilinear fusion) being implemented; reference code from its repository fixes layer sizes and interfaces) — github.com/fzbio/GILoop
- **SIP** (Contrast case: treats Hi-C matrices as images with classical image processing rather than a U-Net encoder) — PouletAxel/SIP
- **Chromosight** (Contrast case: treats matrices as images without learned per-pixel encoders)

## Examples

```
conda create -n GIL python=3.8; conda activate GIL; pip install -r requirements.txt; python demo.py
```

## Evaluation signals

- giloop_model.py exists and executes without error
- A smoke-test forward pass on random Hi-C-like inputs returns per-pixel predictions of the expected shape (one logit per pixel)
- Each pixel yields a 16-dimensional fused representation from the bilinear pooling of the GCN and U-Net branch features
- Focal cross-entropy loss decreases when trained on labeled loop vs. non-loop pixels

## Limitations

- GILoop only focuses on bulk Hi-C and is not capable of dealing with extremely sparse single-cell Hi-C (scHi-C) data
- Downsampling-style preprocessing incurs relatively large pre-processing time (matrix transformations and hard disk reading/writing) before training
- Weak usability if the targeted cell line's sequencing depth differs from any pre-trained models provided in the repository

## Evidence

- [abstract] GILoop dual-branch architecture: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [discussion] lazy mode training: "users can also choose to use the non-downsampling mode (the “lazy” mode) to train the model, which directly trains the GCN branch on the input matrix"
- [discussion] downsampling to optimal density: "which automatically downsamples the Hi-C contact map to the optimal density that we experimentally obtained in this study (260 million cis-contacts)"
- [readme] input formats: "You can either use `.cool` files as input, or use the output format of Juicer dump as input."
- [readme] preprocessing overhead: "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.)"
